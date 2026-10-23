# Red Team Attack Plan & Kerangka Purple Team Loop, SOC BiliCheck

## 1. Identitas Kelompok & Informasi Dokumen

| Atribut Dokumen | Keterangan Lengkap |
| :--- | :--- |
| **Mata Kuliah** | TEK1314, Keamanan Siber (D4 Teknologi Rekayasa Komputer, Sekolah Vokasi IPB) |
| **Kelompok & Kelas** | **Kelompok 11, Kelas A** |
| **Nama Proyek IoT** | **BiliCheck** (*Bilirubin Clinical Health Evaluator*, Tim AGRION / Kelompok 9 Proyek Sistem IoT) |
| **Repositori GitHub** | `TEK1314-2026-Kel11-KelasA` (`https://github.com/Marasetya/TEK1314-2026-Kel11-KelasA`) |
| **Nama Berkas & Path** | `attack_plan.md` (Lokasi di repositori: `docs/redteam/attack_plan.md`) |
| **Jenis Dokumen (Apa Ini?)** | **Dokumen Rencana Skenario Serangan Red Team Fase 2 (Bagian IX), Kesiapan Attacker Node, & Siklus Purple Team Loop** |
| **Fungsi & Tujuan Dokumen** | Merancang minimal 3 skenario serangan siber terkontrol dari 3 kategori berbeda yang menyasar celah spesifik pada arsitektur replika BiliCheck (`192.168.11.5`), mencantumkan tools eksplisit yang telah disiapkan di Kali Linux (`192.168.11.100`), serta menetapkan indikator deteksi dan mitigasi bagi Blue Team. |
| **Tahap Pelaporan** | Checkpoint Minggu ke-9 (Transisi Fase 1 Hardening & Baseline Menuju Fase 2 Vulnerability Assessment) |
| **Tanggal Pemutakhiran** | 23 Oktober 2026 |

### Daftar Anggota Kelompok 11 (Kelas A) & Pembagian Peran SOC

| No. | Kelas | Nama Mahasiswa | NIM | Role (Peran Tim) | Tanggung Jawab Utama pada Proyek SOC BiliCheck |
| :--- | :--- | :--- | :--- | :--- | :--- |
| 1 | A | **Moh Rifky Marasetya** | `J0404231049` | **Lead** | Koordinasi arsitektur simulasi, penyelarasan *Tech Stack Declaration*, pengelolaan repositori GitHub, dan integrasi laporan NIST SP 800-61r2 |
| 2 | A | **Nayla Januarisa Prihadi** | `J0404241149` | **Blue Team** *(Defender / SIEM)* | Konfigurasi *Monitoring Node* Security Onion (`192.168.11.200`), pemulihan Elastic Stack, aktivasi sensor NIDS Snort/Sguil, dan analisis *baseline traffic* |
| 3 | A | **Muhammad Adam Ilyasa** | `J0404241048` | **Blue Team** *(Defender / Host)* | Penanggung jawab *Laptop Utama*, instalasi & eksekusi 9 kontrol *hardening* pada *Target Node* Ubuntu Server 24.04.5 (`192.168.11.5`), serta dokumentasi bukti *before/after* |
| 4 | A | **Fauzan Nafis Jiyad Akbar** | `J0404241152` | **Red Team** *(Attacker)* | Penanggung jawab *Laptop Backup*, konfigurasi *Attacker Node* Kali Linux (`192.168.11.100`), pemetaan permukaan layanan (*reconnaissance*), dan perancangan 3 skenario *Attack Plan* Fase 2 |

## 2. Ruang Lingkup, Batasan Keselamatan, & Kesiapan Attacker Node (`192.168.11.100`)

Seluruh skenario pengujian keamanan dalam dokumen ini dirancang khusus untuk menguji ketahanan **Target Node Replika (`192.168.11.5`)** di dalam jaringan internal laboratorium `bilicheck-lab`. Red Team mematuhi larangan mutlak untuk tidak melakukan pemindaian maupun pengujian apa pun terhadap VPS Domcloud produksi, project Firebase asli, maupun perangkat IoT milik Mitra.

Sebelum menyusun rencana skenario Fase 2, Red Team telah menyelesaikan konfigurasi dan verifikasi kesiapan **Attacker Node (`attacker-kali`)** dengan bukti nyata yang tersimpan di `docs/redteam/evidence/`:

| No. | Aspek Kesiapan Attacker Node | Hasil Verifikasi Aktual di Kali Linux (`192.168.11.100`) | Berkas Bukti di `docs/redteam/evidence/` | Status Minggu 9 |
| :--- | :--- | :--- | :--- | :--- |
| 1 | **Identitas Host & Pengalamatan IP Statis** | Hostname dikonfigurasi menjadi `attacker-kali`; antarmuka `eth0` (`Wired connection 1`) aktif dengan IP statis **`192.168.11.100/24`** dan rute `192.168.11.0/24 dev eth0`. | `01-konfigurasi-ip-dan-routing-kali.png` | **Selesai (Terverifikasi)** |
| 2 | **Konektivitas Layer 2 & Layer 3 ke Target Node** | Uji `ping -c 4 192.168.11.5` berhasil (`4 received, 0% packet loss`, RTT avg `1.592 ms`) dan tabel ARP (`ip neigh`) mengonfirmasi alamat MAC Target Node `08:00:27:47:9f:01 REACHABLE`. | `02-uji-ping-dan-cek-arp-ke-ubuntu.png` | **Selesai (Terverifikasi)** |
| 3 | **Verifikasi Kesiapan Tools Eksplisit Red Team** | Pemeriksaan versi perangkat lunak pengujian mengonfirmasi ketersediaan: **`Nmap 7.99`**, **`curl 8.20.0`** (`OpenSSL 3.6.2`), **`tcpdump 4.99.6`**, dan **`Python 3.13.12`**. | `03-verifikasi-versi-tools-pengujian.png` | **Selesai (Terverifikasi)** |

## 3. Matriks Tiga Skenario Serangan Fase 2 (Standar Bagian IX Panduan PBL)

Sesuai ketentuan **Bagian IX Panduan PBL Pertemuan 9** (minimal 3 skenario serangan dari minimal 2 kategori berbeda dan menyebutkan *tools* secara eksplisit), berikut adalah tiga skenario serangan yang dirancang oleh Red Team Kelompok 11:

| ID & Kategori (Standar Bagian IX) | Nama Skenario Serangan | Tools Eksplisit di Kali Linux (`.100`) | Target Komponen & Port di Replika (`.5`) | Keterkaitan dengan Arsitektur & Celah Riset BiliCheck | Hasil yang Diharapkan & Indikator Deteksi Blue Team | Status Minggu 9 |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **Skenario 1 (S1)**<br>Kategori:<br>**Reconnaissance** | **Pemetaan Permukaan Layanan (Port Scanning), OS Fingerprinting, & Enumerasi Banner Backend** | `nmap` (`v7.99`),<br>`Wireshark` / `tcpdump` | **`192.168.11.5`**<br>Port TCP: `22` (SSH), `80/5000` (Web API), `1883` (MQTT), `3306` (MySQL), `8080/9099` (Firebase Emulator) | Mengidentifikasi layanan backend yang terbuka secara berlebihan sebelum/sesudah penerapan firewall UFW serta memetakan versi daemon sebagai pijakan eksploitasi. | **Hasil Uji:** Mengungkap perbedaan port terbuka sebelum vs sesudah UFW/binding localhost.<br>**Deteksi Blue Team:** Alert NIDS Snort pada Sguil (`ET SCAN Potential SSH Scan` & `ET SCAN Suspicious inbound`). | **Baseline Uji Awal Selesai & Terdeteksi di Sguil** |
| **Skenario 2 (S2)**<br>Kategori:<br>**Web / API / Protokol IoT** | **Injeksi Telemetri Palsu (*MQTT Topic Spoofing*) & Evaluasi Validasi Nilai Sensor ADC (`0–4095`)** | `mosquitto_pub`,<br>`mosquitto_sub`,<br>`MQTT Explorer`,<br>`curl` (`v8.20.0`) / `Burp Suite` | **`192.168.11.5`**<br>Port `1883/tcp` (Mosquitto MQTT) & Port `80/5000/8080` (Web API / Firestore Emulator) | Pada sistem BiliCheck, ESP32 mengirim *raw ADC* 12-bit via MQTT untuk dihitung dengan rumus regresi Hukum Beer-Lambert menjadi kadar bilirubin bayi (`mg/dL`). Jika broker mengizinkan akses anonim atau tanpa validasi input, penyerang dapat menyuntikkan nilai ekstrem (mis. `rawAdc: 99999`) yang memalsukan diagnosis klinis. | **Hasil Uji:** Sebelum mitigasi, payload anonim/tidak valid diterima; sesudah mitigasi (`allow_anonymous false` & filter ADC), koneksi ditolak (`Connection Refused: not authorized`).<br>**Deteksi Blue Team:** Log `/var/log/mosquitto/mosquitto.log` & custom rule Snort topik `bilicheck/`. | **Parsial / Dalam Pengerjaan** (validasi `rawAdc` pada API replika sedang diuji; MQTT, autentikasi broker, dan deteksi keamanan belum diuji dan dijadwalkan Minggu 10-11) |
| **Skenario 3 (S3)**<br>Kategori:<br>**Exploitation / Akses** | **Audit Ketahanan Autentikasi (*Dictionary Attack* Terkontrol pada Layanan SSH / Login Dashboard)** | `Hydra`,<br>`ssh` client,<br>`Nmap NSE` (`ssh-auth-methods`) | **`192.168.11.5`**<br>Port `22/tcp` (OpenSSH, akun uji lab `soc_test`) atau Port `80/5000` (Endpoint Login Petugas Puskesmas) | Menguji ketahanan mekanisme autentikasi administrasi server dan portal petugas puskesmas terhadap percobaan tebakan kata sandi terotomasi secara berulang. | **Hasil Uji:** Autentikasi password ditolak oleh konfigurasi `PasswordAuthentication no` dan IP penyerang (`.100`) diblokir otomatis oleh jail `sshd` pada `Fail2Ban`.<br>**Deteksi Blue Team:** Lonjakan event gagal di `/var/log/auth.log`, status ban di `fail2ban-client status sshd`, & alert brute-force di Sguil. | **Direncanakan (Eksekusi Fase 2 Minggu 10–11)** |

*(Catatan Kepatuhan Bagian IX: Ketiga skenario di atas berasal dari **3 kategori berbeda** dan **tidak menggunakan serangan Availability/DoS** sebagai satu-satunya skenario.)*

## 4. Rincian Teknis Prosedur Pengujian per Skenario

### 4.1 Skenario 1, Pemetaan Permukaan Layanan & Enumerasi Versi (*Reconnaissance*)

* **Tujuan Teknis**: Membandingkan keterjangkauan port TCP pada Target Node (`192.168.11.5`) serta memverifikasi apakah pemindaian jaringan dari `192.168.11.100` menghasilkan alert yang akurat pada sensor Snort/Sguil di Security Onion (`192.168.11.200`).
* **Perintah Eksekusi di Kali Linux**:
  ```bash
  # Pemindaian terbatas pada port arsitektur BiliCheck
  nmap -n -Pn -sT -T3 -vv -p 22,80,1883,3306,5000,8080,9099 --reason --max-retries 1 192.168.11.5

  # Pemindaian enumerasi service version & script default untuk memicu deteksi NIDS
  sudo nmap -sS -sV -T3 192.168.11.5 -oN docs/redteam/evidence/S1-nmap-scan.txt
  ```
* **Hasil Baseline Aktual yang Telah Diperoleh (Minggu ke-9)**:
  1. Pada pemindaian port terbatas pasca-aktivasi UFW (`25-sesudah-daftar-port-terbuka.png`), port **`22/tcp` berstatus `open` (`syn-ack`)**, sedangkan port **`80/tcp`, `1883/tcp`, dan `8080/tcp` berstatus `filtered` (`no-response`)** karena kebijakan `Default: deny (incoming)` pada UFW telah aktif (`15-sesudah-firewall-ufw-aktif-izinkan-ssh.png`).
  2. Pada pengujian deteksi sensor Security Onion (`docs/blueteam/evidence/monitoring-seconion/06-deteksi-alert-scan-dan-ping-di-sguil.png`), pemindaian dari `192.168.11.100` ke `192.168.11.5` **telah terbukti memicu alert *real-time* di Sguil** (`ET SCAN Potential SSH Scan` pada port 22 dan `ET SCAN Suspicious inbound` pada port database 3306, 1521, 1433, 5432).

### 4.2 Skenario 2, Injeksi Telemetri Palsu MQTT & Evaluasi Otorisasi Endpoint (*Web/API/Protokol IoT*)

* **Tujuan Teknis**: Menguji apakah pihak eksternal di segmen jaringan yang sama dapat menyuntikkan data sensor palsu ke topik MQTT BiliCheck tanpa kredensial atau mengirimkan nilai *raw ADC* di luar rentang fisik sensor 12-bit (`0–4095`).
* **Perintah Eksekusi di Kali Linux (Menggunakan Data 100% Dummy)**:
  ```bash
  # Tahap 1 (Sebelum Mitigasi): Mencoba publikasi anonim dengan nilai ADC tidak valid (99999)
  mosquitto_pub -h 192.168.11.5 -p 1883 \
    -t "bilicheck/puskesmas01/dev-test/telemetry" \
    -m '{"deviceId":"BC-TEST-001","pasienId":"DUMMY-001","rawAdc":99999,"nilaiLux":130.0,"timestamp":1761120000}'

  # Tahap 5 (Verifikasi Sesudah Mitigasi A-04 & C-04):
  # 1) Uji ulang tanpa kredensial -> Harus ditolak (Connection Refused: not authorized)
  # 2) Uji dengan kredensial lab namun nilai rawAdc = -50 atau 99999 -> Harus ditolak oleh validator backend
  ```
* **Indikator Deteksi & Mitigasi Blue Team**:
  * Log penolakan koneksi pada `/var/log/mosquitto/mosquitto.log` (`sudo journalctl -u mosquitto --no-pager`).
  * Aturan deteksi kustom Snort/Suricata pada Security Onion:
    ```text
    alert tcp any any -> 192.168.11.5 1883 (msg:"BILICHECK-SOC Suspicious MQTT Telemetry Topic Access"; content:"bilicheck/"; nocase; sid:9000002; rev:1;)
    ```

### 4.3 Skenario 3, Audit Ketahanan Autentikasi Layanan (*Exploitation / Akses*)

* **Tujuan Teknis**: Menguji efektivitas penonaktifan *Password Authentication* pada SSH serta respons pemblokiran otomatis oleh `Fail2Ban` dan deteksi NIDS saat terjadi percobaan login gagal berulang dari `192.168.11.100`.
* **Perintah Eksekusi Terkontrol di Kali Linux**:
  ```bash
  # Pemeriksaan metode autentikasi SSH yang diizinkan oleh server
  nmap -p 22 --script ssh-auth-methods --script-args="ssh.user=soc_test" 192.168.11.5

  # Simulasi percobaan autentikasi gagal berulang secara terbatas (maks. 6 percobaan) dengan Hydra
  hydra -l soc_test -P ~/wordlist_dummy_lab.txt -t 1 -V ssh://192.168.11.5:22
  ```
* **Indikator Deteksi & Mitigasi Blue Team**:
  * Verifikasi bahwa login berbasis password langsung ditolak (`Permission denied (publickey)`).
  * Verifikasi pemblokiran IP `192.168.11.100` pada output `sudo fail2ban-client status sshd` (`16-sesudah-fail2ban-ssh-aktif.png`) dan pencatatan pada `/var/log/auth.log` (`20-sesudah-log-journald-persisten-dan-log-ssh.png`).
  * Munculnya alert `ET SCAN Potential SSH Scan` pada Sguil Security Onion (sebagaimana telah terbukti pada `06-deteksi-alert-scan-dan-ping-di-sguil.png`).

## 5. Kerangka Pelaksanaan 5 Tahap Purple Team Loop (Persiapan Minggu 10–11)

Sesuai ketentuan **Bagian X Panduan PBL Pertemuan 9**, setiap skenario serangan di atas akan dijalankan melalui siklus tertutup lima tahap (*Purple Team Loop*) pada Minggu ke-10 dan ke-11 untuk menghasilkan *Vulnerability Assessment Report* (dicatat pula secara bertahap pada berkas `docs/purple-team-log.md`):

| Tahap Purple Team Loop | Aktivitas Red Team (`192.168.11.100`) | Aktivitas Blue Team (`192.168.11.5` & `.200`) | Bukti Wajib yang Dicatat pada `docs/purple-team-log.md` |
| :--- | :--- | :--- | :--- |
| **1. Serang (*Baseline*)** | Mengeksekusi skenario pengujian (S1, S2, S3) pada kondisi konfigurasi awal sebelum mitigasi spesifik diterapkan. | Memantau konsol Sguil/Kibana di Security Onion dan log lokal di Ubuntu untuk mengamati dampak awal. | Screenshot eksekusi perintah di terminal Kali Linux + status awal layanan di Target Node. |
| **2. Deteksi** | Mencatat *timestamp* (UTC & WIB), parameter paket, serta port sasaran yang diuji. | Mengidentifikasi alert yang muncul di Sguil (`seconion-import`) atau mencatat sebagai *blind spot* bila belum memicu alert. | Screenshot tab *RealTime Events* Sguil (`06-deteksi-alert-scan-dan-ping-di-sguil.png`) dan hasil pencarian `grep`/`journalctl` pada log server. |
| **3. Analisis (*Root Cause*)** | Menjelaskan teknik serangan dan celah konfigurasi yang dieksploitasi pada masing-masing skenario. | Menganalisis akar masalah (*root cause*) mengapa serangan berhasil/gagal atau mengapa deteksi belum optimal. | Catatan analisis akar masalah sebagai bahan draf *Incident Report* berstandar NIST SP 800-61r2. |
| **4. Respons (*Mitigasi*)** | Menunggu penerapan kontrol pertahanan tambahan oleh Blue Team sebelum melakukan pengujian ulang. | Menerapkan kontrol penutup celah (aturan UFW, `allow_anonymous false` di Mosquitto, validasi ADC, `Fail2Ban`, serta *custom rule* Snort). | Bukti perbandingan konfigurasi (*diff config*) sebelum vs sesudah mitigasi dan potongan *rule* baru. |
| **5. Verifikasi** | Mengulangi skenario serangan yang **sama persis** setelah kontrol mitigasi diaktifkan. | Mengonfirmasi bahwa percobaan serangan ulang kini **gagal/ditolak** oleh Target Node sekaligus memicu alert deteksi di Security Onion. | Screenshot percobaan ulang yang gagal/diblokir di Kali Linux + screenshot alert aktif di Sguil. |
