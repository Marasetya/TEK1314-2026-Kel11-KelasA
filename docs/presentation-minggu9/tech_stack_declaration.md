# Deklarasi Tech Stack & Prinsip Replikasi Zero-Risk, SOC BiliCheck

## 1. Identitas Kelompok & Informasi Dokumen

| Atribut Dokumen | Keterangan Lengkap |
| :--- | :--- |
| **Mata Kuliah** | TEK1314, Keamanan Siber (D4 Teknologi Rekayasa Komputer, Sekolah Vokasi IPB) |
| **Kelompok & Kelas** | **Kelompok 11, Kelas A** |
| **Nama Proyek IoT** | **BiliCheck** (*Bilirubin Clinical Health Evaluator*, Tim AGRION / Kelompok 9 Proyek Sistem IoT) |
| **Repositori GitHub** | `TEK1314-2026-Kel11-KelasA` (`https://github.com/Marasetya/TEK1314-2026-Kel11-KelasA`) |
| **Nama Berkas & Path** | `tech_stack_declaration.md` (Lokasi di repositori: `docs/presentation-minggu9/tech_stack_declaration.md`) |
| **Jenis Dokumen (Apa Ini?)** | **Dokumen Deklarasi Pemetaan Teknologi Proyek IoT Asli ke Target Node Replika & Pernyataan Kepatuhan Simulasi Zero-Risk** |
| **Fungsi & Tujuan Dokumen** | Memetakan secara komprehensif setiap komponen teknologi backend pada proyek IoT BiliCheck asli ke lingkungan replika di laboratorium VirtualBox (sesuai Bagian III Panduan PBL Pertemuan 9), menjelaskan alur sensor ke backend, serta menjamin isolasi penuh dari sistem produksi Mitra. |
| **Tahap Pelaporan** | Checkpoint Minggu ke-9 (Transisi Fase 1 Hardening & Baseline Menuju Fase 2 Vulnerability Assessment) |
| **Tanggal Pemutakhiran** | 23 Oktober 2026 |

### Daftar Anggota Kelompok 11 (Kelas A) & Pembagian Peran SOC

| No. | Kelas | Nama Mahasiswa | NIM | Role (Peran Tim) | Tanggung Jawab Utama pada Proyek SOC BiliCheck |
| :--- | :--- | :--- | :--- | :--- | :--- |
| 1 | A | **Moh Rifky Marasetya** | `J0404231049` | **Lead** | Koordinasi arsitektur simulasi, penyelarasan *Tech Stack Declaration*, pengelolaan repositori GitHub, dan integrasi laporan NIST SP 800-61r2 |
| 2 | A | **Nayla Januarisa Prihadi** | `J0404241149` | **Blue Team** *(Defender / SIEM)* | Konfigurasi *Monitoring Node* Security Onion (`192.168.11.200`), pemulihan Elastic Stack, aktivasi sensor NIDS Snort/Sguil, dan analisis *baseline traffic* |
| 3 | A | **Muhammad Adam Ilyasa** | `J0404241048` | **Blue Team** *(Defender / Host)* | Penanggung jawab *Laptop Utama*, instalasi & eksekusi 9 kontrol *hardening* pada *Target Node* Ubuntu Server 24.04.5 (`192.168.11.5`), serta dokumentasi bukti *before/after* |
| 4 | A | **Fauzan Nafis Jiyad Akbar** | `J0404241152` | **Red Team** *(Attacker)* | Penanggung jawab *Laptop Backup*, konfigurasi *Attacker Node* Kali Linux (`192.168.11.100`), pemetaan permukaan layanan (*reconnaissance*), dan perancangan 3 skenario *Attack Plan* Fase 2 |

## 2. Konteks Proyek IoT BiliCheck, Cara Kerja Sensor, & Urgensi Pengamanan SOC

**BiliCheck** (*Bilirubin Clinical Health Evaluator*) adalah sistem perangkat medis IoT portabel non-invasif yang dikembangkan oleh **Tim AGRION (Kelompok 9 Proyek Sistem IoT)** bekerja sama dengan Mitra (**Bapak Aep Setiawan, S.Si., M.Si.**) untuk membantu tenaga kesehatan di Puskesmas dan Posyandu dalam mendeteksi dini *ikterus neonatorum* (*hiperbilirubinemia* / bayi kuning) pada bayi baru lahir secara cepat dan *real-time*.

### Alur Kerja End-to-End (Dari Sensor Optik Menuju Backend & Aplikasi)
1. **Akuisisi Optik (`LED Biru 460 nm + Fotodioda BST`)**: Alat memancarkan cahaya dari **LED Biru berpanjang gelombang 460 nm** (titik serapan puncak molekul bilirubin pada jaringan subkutan). Cahaya yang tidak terserap oleh bilirubin diteruskan dan ditangkap oleh **Sensor Fotodioda BST** (beresistor seri $30\text{ k}\Omega$) untuk diubah menjadi sinyal tegangan analog.
2. **Konversi Digital 12-Bit (`ESP32 WROOM-32`)**: Sinyal tegangan analog dibaca oleh konverter ADC 12-bit pada mikrokontroler **ESP32 WROOM-32**, menghasilkan nilai digital mentah (*raw ADC*) dengan rentang fisik pasti antara **`0` hingga `4095`** beserta nilai intensitas cahaya (*Lux*).
3. **Transmisi Telemetri (`MQTT` & `HTTPS REST API`)**: ESP32 mengemas data (`deviceId`, `pasienId`, `rawAdc`, `nilaiLux`, `timestamp`) ke dalam format JSON dan mengirimkannya melalui protokol **MQTT over TCP/IP (Port 1883)** menggunakan pustaka `PubSubClient` menuju *server backend*.
4. **Komputasi Hukum Beer-Lambert & Visualisasi (`Server Backend` $\rightarrow$ `mg/dL` $\rightarrow$ `Web Dashboard & Aplikasi Mobile`)**: *Server backend* mengolah nilai `rawAdc` dan `nilaiLux` menggunakan **persamaan regresi kurva kalibrasi berbasis Hukum Beer-Lambert** (terkalibrasi larutan standar *ruthenium*/Ru) menjadi estimasi kadar bilirubin serum (`mg/dL`). Hasilnya disimpan ke database **MySQL** serta disinkronkan ke **Cloud Firestore** untuk ditampilkan secara *real-time* pada **Web Dashboard Puskesmas** dan **Aplikasi Mobile Flutter** orang tua pasien.

### Urgensi Pengamanan Infrastruktur Backend oleh Tim SOC
Dalam simulasi **TEK1314 Keamanan Siber**, **Kelompok 11 (Kelas A)** berperan sebagai tim **Security Operations Center (SOC)** yang berfokus mengamankan **infrastruktur backend** (server OS, broker MQTT, API kalkulasi kalibrasi, database rekam medis, dan dashboard monitoring), tanpa menyentuh perangkat sensor fisiknya. Pengamanan lapisan backend ini sangat krusial karena:
* **Keselamatan Klinis Pasien (*Integrity & Patient Safety*)**: Manipulasi nilai `rawAdc` pada jalur MQTT/API (misal injeksi nilai di luar rentang `0–4095`) dapat menyebabkan kesalahan estimasi kadar bilirubin (`mg/dL`), baik berupa *false negative* (bayi kuning berat terbaca normal sehingga terlambat mendapat fototerapi dan berisiko *kernikterus*) maupun *false positive*.
* **Privasi Rekam Medis Bayi (*Confidentiality*)**: Database menyimpan data sensitif pasien bayi baru lahir dan orang tua yang wajib dilindungi dari pencurian data (*data breach*).
* **Ketersediaan Layanan Skrining (*Availability*)**: Gangguan pada server atau broker MQTT akan menghentikan proses deteksi dini di Puskesmas/Posyandu.

## 3. Tabel Wajib Deklarasi Tech Stack (Pemetaan Sistem Asli ke Target Node Replika)

Sesuai ketentuan **Bagian III Panduan PBL Pertemuan 9**, tabel berikut memetakan empat komponen wajib dari arsitektur *hybrid* BiliCheck asli ke dalam **Target Node Replika (`192.168.11.5`)** di laboratorium VirtualBox:

| Komponen Wajib (Standar Bagian III) | Teknologi pada Proyek IoT Asli (BiliCheck, Tim AGRION) | Direplikasi sebagai (Target Node Replika: `192.168.11.5`) | Fungsi dalam Simulasi & Status Kesiapan Minggu ke-9 |
| :--- | :--- | :--- | :--- |
| **1. Sistem Operasi Backend** | Linux VPS (**Domcloud**) + Infrastruktur *Serverless* **Google Firebase** | **Ubuntu Server 24.04.5 LTS** (`target-bilicheck`, Kernel `6.8.0-147-generic`) pada VM VirtualBox di jaringan internal `bilicheck-lab` | Menjadi fondasi host target pengujian yang diperkuat (*hardened*) dengan OpenSSH Key-Auth, UFW, Fail2Ban, Sysctl, AppArmor, Auditd, dan AIDE.<br>**Status Minggu 9:** *Selesai & Terverifikasi 100%* (`01-sebelum-identitas-server-ubuntu.png` s.d. `25-sesudah-daftar-port-terbuka.png`). |
| **2. Database / Penyimpanan Data** | **MySQL** (di Domcloud VPS untuk log telemetri) + **Cloud Firestore** (6 koleksi: `puskesmas`, `users`, `device`, `pasien`, `kalibrasi`, `pemeriksaan`) + **Backup CSV** | **MySQL 8.0 Lokal** (`bind-address = 127.0.0.1:3306`) + **Firebase Local Emulator Suite** (Firestore Emulator Port `8080`) + **Backup CSV** | Menyimpan data rekam medis bayi, koefisien kalibrasi larutan *ruthenium* (Ru), dan log pembacaan sensor menggunakan **100% data sintetis/dummy**.<br>**Status Minggu 9:** *Arsitektur ditetapkan; instalasi daemon pada Fase 2*. |
| **3. Protokol Komunikasi Sensor** | **MQTT** (`ESP32`, `PubSubClient`) + **HTTPS REST API** | **Mosquitto Broker (`1883`)** + **REST API Lokal** + **Simulator Python** (`mock_esp32_bilicheck.py`) | Menyalurkan payload JSON telemetri (`deviceId`, `pasienId`, `rawAdc`, `nilaiLux`, `timestamp`) pada topik `bilicheck/puskesmas01/dev01/telemetry`.<br>**Status:** *REST API Lokal (validasi `rawAdc`): Dalam Implementasi, ditandai terimplementasi setelah layanan dan pengujiannya berhasil. Mosquitto Broker dan Simulator Python: Rencana (belum dibuat), eksekusi pada Fase 2*. |
| **4. Dashboard / Monitoring** | **Web Dashboard** Puskesmas, **Aplikasi Mobile Flutter**, + **Firebase Auth** | **Nginx / Backend Python** (`80`/`5000`) + **Firebase Auth Emulator** (`9099`) + **Security Onion** (`192.168.11.200`) | Menyediakan antarmuka monitoring hasil pemeriksaan bilirubin *dummy* sekaligus memantau seluruh paket jaringan melalui sensor NIDS **Snort (`snort-1`)** & **Sguil**.<br>**Status Minggu 9:** *Security Onion & Sguil Aktif/Terverifikasi* (`01-konfigurasi-ip-dan-routing-seconion.png` s.d. `06-deteksi-alert-scan-dan-ping-di-sguil.png`). |

## 4. Pemetaan Lengkap Ekosistem Simulasi SOC (10 Komponen End-to-End)

Selain keempat komponen inti pada Target Node di atas, ekosistem simulasi Kelompok 11 mengintegrasikan sepuluh komponen teknis yang saling terhubung:

| No. | Komponen Ekosistem | Teknologi / Perangkat Lunak | Lokasi Node | Peran Spesifik dalam Simulasi Keamanan |
| :--- | :--- | :--- | :--- | :--- |
| 1 | **Simulator Perangkat IoT** | Skrip Python `paho-mqtt` (`mock_esp32_bilicheck.py`) & `ping` | Target Node (`.5`) / Attacker Node (`.100`) | Menggantikan perangkat fisik ESP32 dan sensor optik BST untuk membangkitkan trafik *baseline* normal secara berkala. |
| 2 | **Broker Pesan Telemetri** | Eclipse Mosquitto (`mosquitto`, `mosquitto-clients`) | Target Node (`192.168.11.5:1883`) | Objek evaluasi autentikasi broker (`allow_anonymous false`) dan pencegahan *MQTT topic spoofing* (Skenario 2). |
| 3 | **Layanan Kalkulasi & Web API** | Layanan Backend Python/Node.js & Nginx Web Server | Target Node (`192.168.11.5:80/5000`) | Objek evaluasi validasi batas nilai *raw ADC* 12-bit (`0 <= rawAdc <= 4095`) agar terhindar dari manipulasi status klinis pasien. |
| 4 | **Basis Data Relasional & NoSQL** | MySQL Server (`3306`) & Firestore Local Emulator (`8080`) | Target Node (`192.168.11.5`) | Objek penerapan *least-privilege database user* (`bili_app@localhost`), isolasi *localhost binding*, dan *Role-Based Access Control* (RBAC). |
| 5 | **Kontrol Akses & Firewall Host** | OpenSSH (`22`), UFW Firewall, & Fail2Ban (`sshd` jail) | Target Node (`192.168.11.5`) | Membatasi permukaan serangan jaringan (`default deny incoming`) dan memblokir percobaan *brute-force* kredensial (Skenario 1 & 3). |
| 6 | **Audit & Integritas Sistem** | Linux `auditd` (11 rules), `AppArmor` (23 enforce), `AIDE`, & `journald` | Target Node (`192.168.11.5`) | Mencatat setiap perubahan pada berkas konfigurasi krusial (`/etc/passwd`, `/etc/shadow`, `/etc/ssh`, `/etc/ufw`) dan memverifikasi *hash* filesystem. |
| 7 | **Sensor NIDS & Packet Capture** | Snort NIDS (`snort-1`), `barnyard2-1`, & `pcap_agent` (`tcpdump`) | Monitoring Node (`192.168.11.200`, `eth0`) | Menyadap seluruh frame Ethernet pada segmen `bilicheck-lab` dan mencocokkannya dengan aturan deteksi intrusi (*IDS ruleset*). |
| 8 | **Konsol Investigasi SOC & SIEM** | Sguil (`SGUIL-0.9.0`) & Elastic Stack (Elasticsearch, Logstash, Kibana) | Monitoring Node (`192.168.11.200`) | Menampilkan *real-time alert* dari aktivitas pemindaian maupun eksploitasi serta menyimpan log jaringan terpusat. |
| 9 | **Platform Reconnaissance & Audit** | Kali Linux 2026.2 (`nmap 7.99`, `curl 8.20.0`, `tcpdump 4.99.6`, `Hydra`) | Attacker Node (`192.168.11.100`) | Menjalankan pemetaan port/layanan, injeksi payload uji MQTT/API, dan audit ketahanan autentikasi sesuai `attack_plan.md`. |
| 10 | **Repositori Dokumentasi & Bukti** | Git & GitHub (`TEK1314-2026-Kel11-KelasA`) | Cloud / Lokal | Menyimpan seluruh dokumen desain, deklarasi teknologi, *checklist hardening*, *attack plan*, serta artefak bukti *before/after*. |

## 5. Pernyataan Prinsip Keamanan Simulasi (*Zero-Risk Guarantee*)

Sesuai instruksi wajib pada **Bagian III Panduan PBL Pertemuan 9**, seluruh anggota Kelompok 11 memahami dan menjamin tiga prinsip keselamatan berikut:

1. **Isolasi Penuh dari Sistem Produksi Mitra**: *Target Node* (`192.168.11.5`) beroperasi di dalam jaringan internal VirtualBox (`bilicheck-lab`) yang terisolasi dan **sama sekali tidak terhubung** ke VPS Domcloud produksi, project Google Firebase asli, maupun perangkat keras prototipe BiliCheck yang digunakan oleh Tim AGRION dan Mitra (**Bapak Aep Setiawan, S.Si., M.Si.**).
2. **Penggunaan 100% Data Sintetis (*Dummy Data*)**: Seluruh identitas puskesmas, akun petugas medis, akun orang tua, data rekam medis bayi (*pasienId* seperti `DUMMY-BY-001`), serta nilai pembacaan sensor ADC/Lux yang digunakan selama simulasi merupakan data sintetis buatan laboratorium dan tidak mengandung data medis pasien nyata.
3. **Relevansi Temuan untuk Rekomendasi Pasca-Praktikum**: Karena arsitektur dan tumpukan teknologi yang direplikasi meniru desain sistem BiliCheck asli, seluruh temuan celah keamanan, aturan deteksi NIDS, dan konfigurasi *hardening* yang terbukti efektif di laboratorium dapat direkomendasikan secara aman kepada tim pengembang IoT setelah rangkaian praktikum selesai.

## 6. Peta Letak Kerentanan (*Vulnerability Mapping*) & Fokus Pertahanan

| Komponen Tech Stack | Letak Kerentanan (*Vulnerability*) & Risiko Klinis BiliCheck | Skenario Pengujian Red Team (`attack_plan.md`) | Kontrol Hardening & Deteksi Blue Team (`hardening_checklist.md`) |
| :--- | :--- | :--- | :--- |
| **Layanan Jaringan & Port OS (`192.168.11.5`)** | Eksposur port administrasi atau basis data MySQL (`3306`) ke jaringan eksternal tanpa penyaringan firewall. | **Skenario 1 (Reconnaissance)**: Pemindaian TCP/layanan menggunakan `nmap` dari `.100` ke `.5`. | Kebijakan UFW `default deny incoming` (**B-01**), hardening kernel `sysctl` (**B-03**), isolasi MySQL `127.0.0.1` (**B-04**), & deteksi `ET SCAN` di Sguil (`06-deteksi-alert-scan-dan-ping-di-sguil.png`). |
| **Broker MQTT (`1883`) & Endpoint Kalkulasi Beer-Lambert** | Koneksi MQTT anonim (`allow_anonymous true`) & ketiadaan validasi batas nilai *raw ADC* 12-bit (`0–4095`), sehingga injeksi `rawAdc: 99999` dapat memalsukan hasil diagnosis `mg/dL`. | **Skenario 2 (Web/API/Protokol IoT)**: Publikasi pesan MQTT tanpa kredensial (`mosquitto_pub`) & pengiriman nilai ADC di luar batas 12-bit (`curl`/`Burp Suite`). | Penonaktifan akses anonim (`allow_anonymous false`) & `password_file` (**A-04**), validasi ketat rentang integer `0–4095` & proteksi `.env` (**C-04**), serta *custom rule* Snort topik `bilicheck/`. |
| **Layanan OpenSSH (`22`) & Login Portal Puskesmas** | Pengambilalihan akses server melalui tebakan kata sandi berulang (*dictionary / brute-force attack*) pada akun berhak akses tinggi. | **Skenario 3 (Exploitation / Akses)**: Audit ketahanan autentikasi menggunakan `Hydra` & `Nmap NSE (ssh-auth-methods)` pada akun uji terbatas. | Kewajiban *SSH Key-Based Auth* & `PasswordAuthentication no` (**A-01**), pemblokiran otomatis oleh `Fail2Ban` (**A-02**), penguncian `root` & PAM (**A-03**), serta audit integritas file dengan `auditd` (**C-01**) & `AIDE` (**C-03**). |
