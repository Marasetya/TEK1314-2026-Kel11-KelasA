# Deklarasi Tech Stack & Prinsip Replikasi Zero-Risk — SOC BiliCheck

## 1. Identitas Kelompok & Informasi Dokumen

| Atribut Dokumen | Keterangan Lengkap |
| :--- | :--- |
| **Mata Kuliah** | TEK1314 — Keamanan Siber (D4 Teknologi Rekayasa Komputer, Sekolah Vokasi IPB) |
| **Kelompok & Kelas** | **Kelompok 11 — Kelas A** |
| **Nama Proyek IoT** | **BiliCheck** (*Bilirubin Clinical Health Evaluator* — Tim AGRION / Kelompok 9 Proyek Sistem IoT) |
| **Repositori GitHub** | `TEK1314-2026-Kel11-KelasA` (`https://github.com/Marasetya/TEK1314-2026-Kel11-KelasA`) |
| **Nama Berkas & Path** | `tech_stack_declaration.md` (Lokasi di repositori: `docs/presentation-minggu9/tech_stack_declaration.md`) |
| **Jenis Dokumen (Apa Ini?)** | **Dokumen Deklarasi Pemetaan Teknologi Proyek IoT Asli ke Target Node Replika & Pernyataan Kepatuhan Simulasi Zero-Risk** |
| **Fungsi & Tujuan Dokumen** | Memetakan secara komprehensif setiap komponen teknologi backend pada proyek IoT BiliCheck asli ke lingkungan replika di laboratorium VirtualBox (sesuai Bagian III Panduan PBL Pertemuan 9), serta menjamin isolasi penuh dari sistem produksi Mitra. |
| **Tahap Pelaporan** | Checkpoint Minggu ke-9 (Transisi Fase 1 Hardening & Baseline Menuju Fase 2 Vulnerability Assessment) |
| **Tanggal Pemutakhiran** | 22 Oktober 2026 |

### Daftar Anggota Kelompok 11 (Kelas A) & Pembagian Peran SOC

| No. | Kelas | Nama Mahasiswa | NIM | Role (Peran Tim) | Tanggung Jawab Utama pada Proyek SOC BiliCheck |
| :--- | :--- | :--- | :--- | :--- | :--- |
| 1 | A | **Moh Rifky Marasetya** | `J0404231049` | **Lead** | Koordinasi arsitektur simulasi, penyelarasan *Tech Stack Declaration*, pengelolaan repositori GitHub, dan integrasi laporan NIST SP 800-61r2 |
| 2 | A | **Nayla Januarisa Prihadi** | `J0404241149` | **Blue Team** *(Defender / SIEM)* | Konfigurasi *Monitoring Node* Security Onion (`192.168.11.200`), pemulihan Elastic Stack, aktivasi sensor NIDS Snort/Sguil, dan analisis *baseline traffic* |
| 3 | A | **Muhammad Adam Ilyasa** | `J0404241048` | **Blue Team** *(Defender / Host)* | Penanggung jawab *Laptop Utama*, instalasi & eksekusi 9 kontrol *hardening* pada *Target Node* Ubuntu Server 24.04.5 (`192.168.11.5`), serta dokumentasi bukti *before/after* |
| 4 | A | **Fauzan Nafis Jiyad Akbar** | `J0404241152` | **Red Team** *(Attacker)* | Penanggung jawab *Laptop Backup*, konfigurasi *Attacker Node* Kali Linux (`192.168.11.100`), pemetaan permukaan layanan (*reconnaissance*), dan perancangan 3 skenario *Attack Plan* Fase 2 |

## 2. Konteks Proyek IoT BiliCheck & Fokus Pengamanan SOC

**BiliCheck** (*Bilirubin Clinical Health Evaluator*) adalah sistem perangkat medis IoT non-invasif yang dikembangkan oleh **Tim AGRION (Kelompok 9 Proyek Sistem IoT)** bekerja sama dengan Mitra (**Bapak Aep Setiawan, S.Si., M.Si.**) untuk membantu tenaga kesehatan puskesmas dalam mendeteksi dini ikterus neonatorum (*hiperbilirubinemia* / bayi kuning) pada bayi baru lahir.

Secara teknis, perangkat fisik BiliCheck menggunakan sumber cahaya **LED Biru (panjang gelombang 460 nm)** dan **Sensor Fotodioda BST** (beresistor seri 30 kOhm) yang dibaca oleh mikrokontroler **ESP32 WROOM-32**. Nilai *raw ADC* 12-bit (`0–4095`) dan intensitas cahaya (*Lux*) dikirimkan ke infrastruktur server backend untuk dikonversi menjadi estimasi kadar bilirubin serum (`mg/dL`) menggunakan persamaan regresi kurva kalibrasi berbasis **Hukum Beer-Lambert**, lalu disinkronkan ke basis data untuk ditampilkan pada dashboard petugas puskesmas dan aplikasi mobile orang tua pasien.

Dalam simulasi **TEK1314 Keamanan Siber**, **Kelompok 11 (Kelas A)** berperan sebagai tim **Security Operations Center (SOC)** yang bertugas mengamankan **infrastruktur backend** dari sistem BiliCheck tersebut (meliputi sistem operasi server, broker pesan telemetri IoT, layanan komputasi kalibrasi/API, basis data rekam medis, dan dashboard monitoring), tanpa menyentuh perangkat keras sensor fisik maupun server produksi milik Mitra.

## 3. Tabel Wajib Deklarasi Tech Stack (Pemetaan Sistem Asli ke Target Node Replika)

Sesuai ketentuan **Bagian III Panduan PBL Pertemuan 9**, tabel berikut memetakan empat komponen wajib beserta lapisan pendukung dari arsitektur *hybrid* BiliCheck asli ke dalam **Target Node Replika (`192.168.11.5`)** di laboratorium:

| Komponen Wajib (Standar Bagian III) | Teknologi pada Proyek IoT Asli (BiliCheck — Tim AGRION) | Direplikasi sebagai (Target Node Lab: `192.168.11.5`) | Fungsi dalam Simulasi & Status Kesiapan Minggu ke-9 |
| :--- | :--- | :--- | :--- |
| **1. Sistem Operasi Backend** | Linux Cloud Server (**Domcloud VPS**) & Infrastruktur *Serverless* **Google Firebase** | **Ubuntu Server 24.04.5 LTS** (`target-bilicheck`, Kernel `6.8.0-147-generic`) pada VM VirtualBox di jaringan internal `bilicheck-lab` | Menjadi fondasi host target pengujian yang diperkuat (*hardened*) dengan OpenSSH Key-Auth, UFW, Fail2Ban, Sysctl, AppArmor, Auditd, dan AIDE.<br>**Status Minggu 9:** *Selesai & Terverifikasi 100%*. |
| **2. Database / Penyimpanan Data** | **MySQL** (di Domcloud VPS untuk log telemetri), **Cloud Firestore** (6 koleksi: `puskesmas`, `users`, `device`, `pasien`, `kalibrasi`, `pemeriksaan`), & **Backup CSV** | **MySQL Server 8.0 Lokal** (`bind-address = 127.0.0.1:3306`), **Firebase Local Emulator Suite** (Firestore Emulator Port `8080`), & **Direktori Cron Backup CSV** | Menyimpan data rekam medis bayi, koefisien kalibrasi larutan *ruthenium* (Ru), dan log pembacaan sensor menggunakan **100% data sintetis/dummy**.<br>**Status Minggu 9:** *Arsitektur ditetapkan; instalasi daemon pada Fase 2*. |
| **3. Protokol Komunikasi Sensor** | **MQTT over TCP/IP** (ESP32 WROOM-32 mengirim *raw ADC* 12-bit via pustaka `PubSubClient`) & **HTTPS REST API** | **Mosquitto MQTT Broker** (Port `1883`) & **Endpoint REST API Lokal** yang menerima trafik dari skrip simulator Python (`mock_esp32_bilicheck.py`) | Menyalurkan payload JSON telemetri (`deviceId`, `pasienId`, `rawAdc`, `nilaiLux`, `timestamp`) pada topik `bilicheck/puskesmas01/dev01/telemetry`.<br>**Status Minggu 9:** *Arsitektur ditetapkan; eksekusi pengujian pada Fase 2*. |
| **4. Dashboard / Monitoring & Layanan Aplikasi** | **Web Dashboard Admin Laboratorium**, **Aplikasi Mobile BiliCheck** (Flutter + FCM), **Firebase Authentication** (Role *Petugas* vs *Orang Tua*), & Layanan Kurva Kalibrasi | **Nginx Web Server / Layanan Backend Python** (Port `80`/`5000`), **Firebase Auth Emulator** (Port `9099`), serta Pemantauan SIEM oleh **Security Onion (`192.168.11.200`)** | Menyediakan antarmuka monitoring hasil pemeriksaan bilirubin *dummy* sekaligus memantau seluruh paket jaringan melalui sensor NIDS **Snort (`snort-1`)** & **Sguil**.<br>**Status Minggu 9:** *Security Onion & Sguil Aktif/Terverifikasi; Web/Auth Emulator pada Fase 2*. |

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
| 7 | **Sensor NIDS & Packet Capture** | Snort NIDS (`snort-1`), `barnyard2-1`, & `pcap_agent` (`tcpdump`) | Monitoring Node (`192.168.11.200` — `eth0`) | Menyadap seluruh frame Ethernet pada segmen `bilicheck-lab` dan mencocokkannya dengan aturan deteksi intrusi (*IDS ruleset*). |
| 8 | **Konsol Investigasi SOC & SIEM** | Sguil (`SGUIL-0.9.0`) & Elastic Stack (Elasticsearch, Logstash, Kibana) | Monitoring Node (`192.168.11.200`) | Menampilkan *real-time alert* dari aktivitas pemindaian maupun eksploitasi serta menyimpan log jaringan terpusat. |
| 9 | **Platform Reconnaissance & Audit** | Kali Linux 2026.2 (`nmap 7.99`, `curl 8.20.0`, `tcpdump 4.99.6`, `Hydra`) | Attacker Node (`192.168.11.100`) | Menjalankan pemetaan port/layanan, injeksi payload uji MQTT/API, dan audit ketahanan autentikasi sesuai `attack_plan.md`. |
| 10 | **Repositori Dokumentasi & Bukti** | Git & GitHub (`TEK1314-2026-Kel11-KelasA`) | Cloud / Lokal | Menyimpan seluruh dokumen desain, deklarasi teknologi, *checklist hardening*, *attack plan*, serta artefak bukti *before/after*. |

## 5. Pernyataan Prinsip Keamanan Simulasi (*Zero-Risk Guarantee*)

Sesuai instruksi wajib pada **Bagian III Panduan PBL Pertemuan 9**, seluruh anggota Kelompok 11 memahami dan menjamin tiga prinsip keselamatan berikut:

1. **Isolasi Penuh dari Sistem Produksi Mitra**: *Target Node* (`192.168.11.5`) beroperasi di dalam jaringan internal VirtualBox (`bilicheck-lab`) yang terisolasi dan **sama sekali tidak terhubung** ke VPS Domcloud produksi, project Google Firebase asli, maupun perangkat keras prototipe BiliCheck yang digunakan oleh Tim AGRION dan Mitra (**Bapak Aep Setiawan, S.Si., M.Si.**).
2. **Penggunaan 100% Data Sintetis (*Dummy Data*)**: Seluruh identitas puskesmas, akun petugas medis, akun orang tua, data rekam medis bayi (*pasienId* seperti `DUMMY-BY-001`), serta nilai pembacaan sensor ADC/Lux yang digunakan selama simulasi merupakan data sintetis buatan laboratorium dan tidak mengandung data medis pasien nyata.
3. **Relevansi Temuan untuk Rekomendasi Pasca-Praktikum**: Karena arsitektur dan tumpukan teknologi yang direplikasi meniru desain sistem BiliCheck asli, seluruh temuan celah keamanan, aturan deteksi NIDS, dan konfigurasi *hardening* yang terbukti efektif di laboratorium dapat direkomendasikan secara aman kepada tim pengembang IoT setelah rangkaian praktikum selesai.

## 6. Keterkaitan Tech Stack dengan Permukaan Risiko & Fokus Pertahanan

| Komponen Tech Stack | Risiko Keamanan Utama pada Konteks Medis BiliCheck | Skenario Pengujian Red Team (`attack_plan.md`) | Kontrol Hardening & Deteksi Blue Team (`hardening_checklist.md`) |
| :--- | :--- | :--- | :--- |
| **Layanan Jaringan & Port OS** | Eksposur port administrasi atau basis data yang tidak diperlukan ke jaringan luar. | **Skenario 1 (Reconnaissance)**: Pemindaian TCP/layanan menggunakan `nmap` dari `.100` ke `.5`. | Kebijakan UFW `default deny incoming` (hanya membuka port allowlist) & deteksi `ET SCAN` pada Sguil Security Onion. |
| **Broker MQTT & Endpoint Telemetri** | Injeksi nilai *raw ADC* palsu (mis. `rawAdc: 99999`) oleh pihak tak berwenang yang dapat memicu kesalahan diagnosis kadar bilirubin bayi. | **Skenario 2 (Web/API/Protokol IoT)**: Publikasi pesan MQTT tanpa kredensial (`mosquitto_pub`) & pengiriman nilai ADC di luar batas 12-bit. | Penonaktifan akses anonim (`allow_anonymous false`), autentikasi `password_file`, dan validasi ketat rentang integer `0–4095` di aplikasi. |
| **Layanan OpenSSH & Login Portal** | Pengambilalihan akses server melalui tebakan kata sandi berulang (*dictionary / brute-force attack*). | **Skenario 3 (Exploitation / Akses)**: Audit ketahanan autentikasi menggunakan `Hydra` / klien SSH pada akun uji terbatas. | Kewajiban *SSH Key-Based Auth*, `PasswordAuthentication no`, `PermitRootLogin no`, pemblokiran otomatis oleh `Fail2Ban`, dan alert SSH Scan di Sguil. |
