# Hardening Checklist & Verifikasi Baseline Monitoring, SOC BiliCheck

> **Keterangan file:** Checklist penerapan dan verifikasi kontrol hardening pada Target Node dan layanan replika BiliCheck.
> **Isi utama:** Kontrol kategori A, B, dan C, bukti before/after, status API, MySQL, MQTT, dan monitoring.
> **Penanggung jawab:** Muhammad Adam Ilyasa sebagai satu-satunya anggota Blue Team.
> **Status dokumen:** Dokumen aktif. Status kontrol harus mengikuti bukti aktual.

## 1. Identitas Kelompok & Informasi Dokumen

| Atribut Dokumen | Keterangan Lengkap |
| :--- | :--- |
| **Mata Kuliah** | TEK1314, Keamanan Siber (D4 Teknologi Rekayasa Komputer, Sekolah Vokasi IPB) |
| **Kelompok & Kelas** | **Kelompok 11, Kelas A** |
| **Nama Proyek IoT** | **BiliCheck** (*Bilirubin Clinical Health Evaluator*, Tim AGRION / Kelompok 9 Proyek Sistem IoT) |
| **Repositori GitHub** | `TEK1314-2026-Kel11-KelasA` (`https://github.com/Marasetya/TEK1314-2026-Kel11-KelasA`) |
| **Nama Berkas & Path** | `hardening_checklist.md` (Lokasi di repositori: `docs/blueteam/hardening_checklist.md`) |
| **Jenis Dokumen (Apa Ini?)** | **Dokumen Daftar Periksa Kontrol Hardening Target Node (Bagian VIII), Bukti Before/After, & Status Monitoring Security Onion** |
| **Fungsi & Tujuan Dokumen** | Mendokumentasikan penerapan minimal 8 kontrol keamanan wajib yang tersebar pada Kategori A (Access & Authentication), Kategori B (Service & Attack Surface), dan Kategori C (Data, Logging, & System Integrity) beserta tautan berkas bukti nyata sebelum dan sesudah konfigurasi. |
| **Tahap Pelaporan** | Checkpoint Minggu ke-9 (Transisi Fase 1 Hardening & Baseline Menuju Fase 2 Vulnerability Assessment) |
| **Tanggal Pemutakhiran** | 5 November 2026 (revisi pembagian peran) |

### Daftar Anggota Kelompok 11 (Kelas A) dan Pembagian Peran SOC

| No. | Kelas | Nama Mahasiswa | NIM | Role (Peran Tim) | Tanggung Jawab Utama pada Proyek SOC BiliCheck |
| :--- | :--- | :--- | :--- | :--- | :--- |
| 1 | A | **Moh Rifky Marasetya** | `J0404231049` | **Lead** | Koordinasi arsitektur simulasi, penyelarasan Tech Stack Declaration, pengelolaan repositori GitHub, penggabungan dokumentasi, dan integrasi laporan NIST SP 800-61r2. |
| 2 | A | **Nayla Januarisa Prihadi** | `J0404241149` | **Red Team** (Attacker / Operator) | Anggota Red Team. Bersama Fauzan menjalankan pengujian S1, S2, dan S3 pada Attacker Node Kali Linux, mencatat perintah, waktu, hasil, dan bukti pengujian. |
| 3 | A | **Muhammad Adam Ilyasa** | `J0404241048` | **Blue Team** (Defender / Host dan SIEM) | Satu-satunya anggota Blue Team. Bertanggung jawab atas Target Node Ubuntu, kontrol hardening, API, MySQL, MQTT, UFW, log host, Monitoring Node Security Onion, sensor, Snort, Sguil, analisis deteksi, dan bukti before/after. |
| 4 | A | **Fauzan Nafis Jiyad Akbar** | `J0404241152` | **Red Team** (Attacker / Operator) | Anggota Red Team. Bersama Nayla mengelola Attacker Node Kali Linux, menjalankan pengujian S1, S2, dan S3, menyusun Attack Plan, serta mencatat hasil dan bukti. |

> **Revisi pembagian peran berlaku mulai Minggu 10:** Nayla dan Fauzan berada di Red Team. Muhammad Adam menjadi satu-satunya anggota Blue Team dan menangani seluruh pekerjaan pertahanan, baik host maupun monitoring. Moh Rifky tetap menjadi Lead.

## 2. Ringkasan Pemenuhan Standar Minimum Hardening

Dokumen ini mempertahankan catatan kontrol hardening Minggu 9 dan menambahkan status layanan yang diverifikasi pada Minggu 10. Status berikut membedakan kontrol yang sudah didukung bukti dari kontrol yang masih membutuhkan pengujian ulang.

- **Kategori A, Access & Authentication:** A-01, A-02, dan A-03 memiliki bukti hardening. A-04 sudah memiliki bukti konfigurasi `allow_anonymous false` dan pengujian dari sisi klien, tetapi log broker final belum cukup untuk menyatakan seluruh autentikasi MQTT terbukti lengkap.
- **Kategori B, Service & Attack Surface:** B-01, B-02, dan B-03 memiliki bukti. B-04 memiliki bukti MySQL listen pada `127.0.0.1:3306`. Langkah `mysql_secure_installation` tidak dinyatakan selesai karena bukti spesifiknya tidak tersedia pada arsip ini.
- **Kategori C, Data & Application:** C-01, C-02, dan C-03 memiliki bukti hardening. C-04 memisahkan kredensial ke EnvironmentFile dengan permission `0600` dan pengujian Bearer Token, tetapi bukti rotasi token final perlu dipastikan dari catatan pelaksanaan. C-05 least-privilege MySQL didukung `SHOW GRANTS` dan data dummy. C-06 validasi ADC `0-4095` didukung hasil HTTP `202` dan `422`.

**Catatan status:** keberadaan screenshot konfigurasi tidak otomatis membuktikan semua tahapan pengujian ulang. Status akhir setiap kontrol harus mengikuti output yang terlihat dan catatan pengujian aktual.

## 3. Matriks Utama Hardening Checklist (Target Node: `192.168.11.5`)

Seluruh berkas bukti disimpan di dalam direktori `docs/blueteam/evidence/hardening/` dengan penamaan berkas berurutan `01` s.d. `27`:

| ID | Kategori Wajib | Kontrol Hardening Spesifik | Kondisi *Before* (Baseline Awal) | Kondisi *After* (Hasil Verifikasi Aktual) | Berkas Bukti Wajib di `docs/blueteam/evidence/hardening/` (*Before vs After*) | Status Minggu 9 |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **A-01** | **A. Access & Authentication** | **Penonaktifan Login Root, Kewajiban SSH Key-Based Auth, & Pembatasan Sesi SSH** (`/etc/ssh/sshd_config.d/00-bilicheck-hardening.conf`) | `permitrootlogin without-password`<br>`passwordauthentication yes`<br>`maxauthtries 6`<br>`logingracetime 120`<br>`x11forwarding yes` | `permitrootlogin no`<br>`passwordauthentication no`<br>`pubkeyauthentication yes`<br>`kbdinteractiveauthentication no`<br>`maxauthtries 3`, `logingracetime 30`<br>`x11forwarding no`, `allowusers socadmin` | **Before:** `03-sebelum-status-ssh-dan-port.png`, `06-sebelum-laporan-baseline-dibuat.png`<br>**After:** `13-sesudah-uji-login-ssh-key.png`, `20-sesudah-log-journald-persisten-dan-log-ssh.png` | **Selesai, Ada Bukti** |
| **A-02** | **A. Access & Authentication** | **Proteksi Brute-Force SSH dengan Fail2Ban** (`/etc/fail2ban/jail.d/bilicheck.local`) | Paket `fail2ban` belum terpasang; tidak ada pemblokiran otomatis terhadap kegagalan login berulang. | `fail2ban.service` aktif (`running`), `Number of jail: 1` (`sshd`), `backend = systemd`, memantau unit `sshd.service` (`maxretry = 5`, `findtime = 10m`, `bantime = 1h`). | **Before:** `05-sebelum-status-firewall-dan-layanan.png`, `06-sebelum-laporan-baseline-dibuat.png`<br>**After:** `11-sesudah-instalasi-paket-keamanan.png`, `16-sesudah-fail2ban-ssh-aktif.png` | **Selesai, Ada Bukti** |
| **A-03** | **A. Access & Authentication** | **Penguncian Akun Root Lokal, Pembatasan Hak Sudo, & Kebijakan Kualitas Password PAM** (`/etc/security/pwquality.conf`) | Hanya mengandalkan pengaturan bawaan instalasi; kebijakan kompleksitas password PAM belum dikonfigurasi khusus. | Akun `root` terkunci (`passwd -S root` -> `L`), grup `sudo` dibatasi hanya `socadmin`, izin `/etc/shadow` & `/etc/gshadow` `640` (`root:shadow`), `/etc/sudoers` `440`, serta `libpam-pwquality` dikonfigurasi (`minlen = 14`, `retry = 3`). | **Before:** `04-sebelum-akun-sudo-dan-izin-file.png`<br>**After:** `11-sesudah-instalasi-paket-keamanan.png`, `12-sesudah-kunci-root-dan-izin-sudo.png`, `22-sesudah-kebijakan-kualitas-password-pam.png` | **Selesai, Ada Bukti** |
| **A-04** | **A. Access & Authentication** | **Autentikasi Wajib pada Broker MQTT Mosquitto** | Broker MQTT pada konfigurasi awal berpotensi menerima koneksi anonim. | Konfigurasi memperlihatkan `allow_anonymous false` dan `password_file`. Screenshot klien memperlihatkan percobaan anonim ditolak dan command publish menggunakan akun lab. Log server yang memastikan autentikasi final masih perlu dilengkapi. | `docs/blueteam/evidence/application/10-konfigurasi-autentikasi-mqtt.png`; `11-hasil-uji-mqtt-anonim-ditolak-sebagian.png`; `12-hasil-uji-mqtt-akun-lab-client-side.png`; `13-log-mosquitto-service-startup-saja.png` | **Sebagian, menunggu bukti log broker final** |
| **B-01** | **B. Service & Attack Surface** | **Firewall Host UFW dengan Kebijakan `Default Deny Incoming` & Whitelist Port** | `Status: inactive` (Tidak ada penyaringan paket masuk pada tingkat host). | `Status: active`, `Logging: on (low)`, `Default: deny (incoming), allow (outgoing), disabled (routed)`. Hanya membuka port `22/tcp` (IPv4 & IPv6). Hasil scan `nmap` dari Kali membuktikan port `80, 1883, 8080` berstatus `filtered`. | **Before:** `05-sebelum-status-firewall-dan-layanan.png`, `14-sebelum-aturan-firewall-ufw-diaktifkan.png`<br>**After:** `15-sesudah-firewall-ufw-aktif-izinkan-ssh.png` | **Selesai, Ada Bukti** |
| **B-02** | **B. Service & Attack Surface** | **Pembaruan Patch Keamanan OS (`apt full-upgrade`) & Aktivasi Pembaruan Otomatis (`unattended-upgrades`)** | Terdapat paket sistem yang belum diperbarui dan mekanisme pembaruan keamanan otomatis belum diverifikasi. | Seluruh paket keamanan diperbarui, timer `apt-daily.timer` & `apt-daily-upgrade.timer` aktif, serta `/etc/apt/apt.conf.d/20auto-upgrades` bernilai `Update-Package-Lists "1"` dan `Unattended-Upgrade "1"`. | **Before:** `01-sebelum-identitas-server-ubuntu.png`, `06-sebelum-laporan-baseline-dibuat.png`<br>**After:** `09-sesudah-update-paket-os-dan-jaringan.png`, `21-sesudah-konfigurasi-update-otomatis.png` | **Selesai, Ada Bukti** |
| **B-03** | **B. Service & Attack Surface** | **Hardening Parameter Jaringan & Kernel (`sysctl`) serta Audit Minimasi Port/Layanan Aktif** (`/etc/sysctl.d/99-bilicheck-hardening.conf`) | Parameter proteksi jaringan dan kernel masih menggunakan nilai standar; inventarisasi layanan belum diaudit. | `kernel.randomize_va_space = 2`, `kptr_restrict = 2`, `dmesg_restrict = 1`, `fs.protected_hardlinks = 1`, `fs.protected_symlinks = 1`, `net.ipv4.tcp_syncookies = 1`, `accept_redirects = 0`, `send_redirects = 0`, `accept_source_route = 0`, `rp_filter = 1`. Hanya port `22/tcp` yang *listening* ke jaringan luar. | **Before:** `03-sebelum-status-ssh-dan-port.png`, `05-sebelum-status-firewall-dan-layanan.png`<br>**After:** `17-sesudah-hardening-kernel-sysctl.png`, `24-sesudah-daftar-layanan-berjalan.png`, `25-sesudah-daftar-port-terbuka.png` | **Selesai, Ada Bukti** |
| **B-04** | **B. Service & Attack Surface** | **Isolasi Binding Database MySQL ke Localhost** | Database berisiko terekspos melalui interface jaringan. | MySQL terlihat listen pada `127.0.0.1:3306`. Bukti ini mendukung pembatasan akses jaringan ke database. Penyelesaian `mysql_secure_installation` tidak diklaim karena bukti spesifik tidak tersedia. | `docs/blueteam/evidence/application/08-verifikasi-binding-mysql.png`; `09-show-grants-dan-data-telemetry-dummy.png` | **Selesai untuk binding localhost; secure installation belum diverifikasi** |
| **C-01** | **C. Data & Application** | **Aktivasi Security Auditing (`auditd`) untuk Pemantauan Perubahan File Identitas & Konfigurasi** (`/etc/audit/rules.d/bilicheck.rules`) | `sudo: auditctl: command not found` (Layanan `auditd` belum terpasang dan tidak ada aturan audit perubahan berkas). | `auditd.service` aktif (`enabled 1`, `pid 589/1728`) dengan **11 aturan audit aktif** yang memantau akses tulis/atribut (`-p wa`) pada `/etc/passwd`, `/etc/group`, `/etc/shadow`, `/etc/gshadow` (`identity`), `/etc/sudoers` (`scope`), `/etc/ssh/sshd_config` (`ssh_config`), `/etc/netplan` (`network_config`), `/etc/ufw` (`firewall_config`), dan `/etc/sysctl.d` (`sysctl_config`). | **Before:** `06-sebelum-laporan-baseline-dibuat.png`<br>**After:** `18-sesudah-layanan-auditd-dan-aturan-aktif.png` | **Selesai, Ada Bukti** |
| **C-02** | **C. Data & Application** | **Penegakan Mandatory Access Control (`AppArmor`) pada Layanan Sistem** | Profil AppArmor belum ditinjau secara menyeluruh pasca-instalasi paket keamanan tambahan. | Modul `apparmor` aktif dengan **117 profil dimuat** (`23 profiles in enforce mode`, termasuk `rsyslogd`, `tcpdump`, `unix-chkpwd`, `snap-confine`) dan `0 unconfined processes` yang memiliki profil terdefinisi. | **Before:** `06-sebelum-laporan-baseline-dibuat.png`<br>**After:** `19-sesudah-status-apparmor-aktif.png` | **Selesai, Ada Bukti** |
| **C-03** | **C. Data & Application** | **Pemeriksaan Integritas Filesystem (`AIDE`), Persistent Logging (`systemd-journald`), & Sinkronisasi Waktu NTP (`Asia/Jakarta`)** | Log jurnal belum dikonfigurasi persisten dengan batas retensi khusus, zona waktu masih `UTC (+00:00)`, dan database integritas AIDE belum ada. | Zona waktu diselaraskan ke `Asia/Jakarta (WIB, +0700)` dengan `NTP service: active` (`185.125.190.56`), log `journald` disimpan persisten (`/etc/systemd/journald.conf.d/99-bilicheck.conf`, `44.1M`), serta database **AIDE (`0.18.6`)** memverifikasi **102.127 entri filesystem**. | **Before:** `06-sebelum-laporan-baseline-dibuat.png`<br>**After:** `10-sesudah-sinkronisasi-zona-waktu-ntp.png`, `20-sesudah-log-journald-persisten-dan-log-ssh.png`, `23-sesudah-cek-integritas-file-aide.png` | **Selesai, Ada Bukti** |
| **C-04** | **C. Data & Application** | **Proteksi Kredensial Aplikasi Menggunakan EnvironmentFile dan Bearer Token** | Kredensial aplikasi perlu dipisahkan dari source code dan dibatasi akses filenya. | Systemd memuat `/etc/bilicheck-api/bilicheck-api.env` dan `/etc/bilicheck-api/database.env`. Screenshot menunjukkan kedua file dimiliki `root:root` dengan permission `0600`. Uji API memperlihatkan `401` tanpa token serta hasil `202` dan `422` pada pengujian dengan variabel token. Catatan rotasi token final perlu dipastikan sebelum status dinyatakan selesai penuh. | `docs/blueteam/evidence/application/04-proteksi-kredensial-env-api.png`; `07-uji-api-tanpa-token-401-dan-permintaan-lanjutan-belum-lengkap.png`; `07b-uji-api-token-valid-202-dan-adc-4096-422.png` | **Sebagian, konfirmasi rotasi token final diperlukan** |
| **C-05** | **C. Data & Application** | **Least-Privilege pada Akun Database Aplikasi MySQL** | Akun database khusus aplikasi dan pembatasan hak akses minimum belum diterapkan. | Akun `bilicheck_app`@`127.0.0.1` dibatasi pada hak `INSERT` di `bilicheck_db.telemetry`. `SHOW GRANTS` dan data dummy mendukung penerapan hak minimum yang didokumentasikan. | `docs/blueteam/evidence/application/05-penerapan-least-privilege-dan-data-mysql.png`; `09-show-grants-dan-data-telemetry-dummy.png` | **Selesai, didukung bukti** |
| **C-06** | **C. Data & Application** | **Validasi Input Telemetri ADC pada API Replika BiliCheck** | Endpoint penerima telemetri belum menerapkan validasi tipe data dan rentang ADC 12-bit. | API menerima nilai `2048` dengan HTTP `202` dan menolak nilai `4096` dengan HTTP `422`. Batas validasi yang didokumentasikan adalah `0-4095`. | `docs/blueteam/evidence/application/01-validasi-input-rawadc-pada-api.png`; `07b-uji-api-token-valid-202-dan-adc-4096-422.png`; `docs/blueteam/evidence/hardening/26-validasi-input-rawadc-pada-api.png` | **Selesai, didukung bukti** |

## 4. Bukti Perbandingan Konfigurasi Utama (*Before vs After Diff*)

Berdasarkan hasil perintah `diff -u baseline-before-hardening.txt after-hardening.txt` pada `target-bilicheck` (proses pengambilan baseline terekam pada `06-sebelum-laporan-baseline-dibuat.png`), berikut adalah ringkasan perubahan konfigurasi efektif yang telah terverifikasi:

```text
--- /home/socadmin/bilicheck-evidence/baseline-before-hardening.txt
+++ /home/socadmin/bilicheck-evidence/after-hardening.txt
@@ SSH EFFECTIVE CONFIG @@
 port 22
-logingracetime 120
-maxauthtries 6
-permitrootlogin without-password
+logingracetime 30
+maxauthtries 3
+permitrootlogin no
 pubkeyauthentication yes
-passwordauthentication yes
+passwordauthentication no
 kbdinteractiveauthentication no
-x11forwarding yes
+x11forwarding no
+allowusers socadmin

@@ FIREWALL (UFW) @@
-Status: inactive
+Status: active
+Logging: on (low)
+Default: deny (incoming), allow (outgoing), disabled (routed)
+To                         Action      From
+22/tcp                     ALLOW IN    Anywhere   # SSH via VirtualBox port forwarding
+22/tcp (v6)                ALLOW IN    Anywhere (v6)

@@ FAIL2BAN & AUDITD @@
+Number of jail: 1 (Jail list: sshd)
-sudo: auditctl: command not found
+enabled 1 (11 watch rules loaded: identity, scope, ssh_config, network_config, firewall_config, sysctl_config)
+APT::Periodic::Update-Package-Lists "1";
+APT::Periodic::Unattended-Upgrade "1";
+Time zone: Asia/Jakarta (WIB, +0700) | System clock synchronized: yes | NTP service: active
```

## 5. Kesiapan Monitoring Node (Security Onion `192.168.11.200`) & Bukti Baseline Traffic

Selain melakukan *hardening* pada Target Node, Blue Team telah menyelesaikan konfigurasi penuh pada **Monitoring Node Security Onion (`SecOnion`)** dan membuktikan bahwa trafik jaringan lab beserta pemindaian awal telah berhasil tertangkap dan terdeteksi secara *real-time*:

| No. | Aspek Verifikasi Monitoring Node | Hasil Pemeriksaan Aktual | Berkas Bukti di `docs/blueteam/evidence/monitoring-seconion/` | Status |
| :--- | :--- | :--- | :--- | :--- |
| 1 | **Konfigurasi Antarmuka & Tabel Routing Pasca-Reboot** | `eth0` berstatus `UP` tanpa IP (mode *promiscuous sniffing* pada `bilicheck-lab`), `eth1` memiliki IP manajemen `192.168.11.200/24`, dan `eth2` memiliki IP NAT `10.0.2.15/24` dengan *default route* melalui `10.0.2.2`. | `01-konfigurasi-ip-dan-routing-seconion.png` | **Selesai (Terverifikasi)** |
| 2 | **Pemulihan Layanan SIEM Elastic Stack** | Pemeriksaan `sudo so-elastic-status` mengonfirmasi seluruh kontainer berjalan normal: `so-elasticsearch [ OK ]`, `so-logstash [ OK ]`, `so-kibana [ OK ]`, dan `so-freqserver [ OK ]`. | `02-status-elastic-stack-dan-sensor-nsm-aktif.png` | **Selesai (Terverifikasi)** |
| 3 | **Aktivasi Komponen Sensor NSM & Engine NIDS Snort** | Sensor `seconion-import` terpetakan ke `eth0`. Pemeriksaan `sudo nsm_sensor_ps-status` menunjukkan `pcap_agent [ OK ]`, `snort_agent-1 [ OK ]`, **`snort-1 (alert data) [ OK ]`**, dan `barnyard2-1 [ OK ]` dengan parameter `35:IDS_ENGINE_ENABLED="yes"`. | `02-status-elastic-stack-dan-sensor-nsm-aktif.png`,<br>`03-verifikasi-engine-ids-snort-aktif.png` | **Selesai (Terverifikasi)** |
| 4 | **Penangkapan Baseline Traffic ICMP Dua Arah (`tcpdump`)** | Penangkapan paket pada `eth0` berhasil merekam 5 *ICMP echo request* (`192.168.11.100 -> 192.168.11.5`) dan 5 *ICMP echo reply* (`192.168.11.5 -> 192.168.11.100`) dengan ringkasan **`10 packets captured, 0 packets dropped by kernel`**. | `04-tangkap-paket-icmp-lengkap-tcpdump.png`,<br>`05-ringkasan-tangkap-paket-icmp.png` | **Selesai (Terverifikasi)** |
| 5 | **Deteksi Event Real-Time pada Konsol Sguil (`SGUIL-0.9.0`)** | Konsol Sguil berhasil menampilkan **alert *real-time* terbaru pada tanggal `2026-10-22` (`07:35:07` - `07:46:11` UTC)** dari IP sumber `192.168.11.100` menuju `192.168.11.5`, meliputi:<br>• `ET SCAN Potential SSH Scan` (Port `22/tcp`, Alert ID `5.1182` & `5.1183`)<br>• `ET SCAN Suspicious inbound` ke port MySQL `3306` (`5.1191`), `1521` (`5.1193`), `1433` (`5.1196`), `5432` (`5.1199`), & VNC `5902/5815`<br>• `GPL ICMP_INFO PING *NIX` (Alert ID `5.1201`, `CNT: 73`). | `06-deteksi-alert-scan-dan-ping-di-sguil.png` | **Selesai (Terverifikasi)** |

## 6. Catatan Verifikasi Monitoring Minggu 10

Dokumen bagian 5 di atas mencatat hasil monitoring historis yang tersedia pada checkpoint sebelumnya. Bukti Minggu 10 perlu dibaca secara terpisah:

- Elastic Stack dan agent sensor NSM terlihat berstatus `[ OK ]`.
- Konfigurasi `IDS_ENGINE_ENABLED="yes"` terlihat pada screenshot.
- Pemeriksaan proses Snort pada screenshot Minggu 10 tidak menampilkan proses yang dapat dipastikan.
- Log sensor menampilkan pesan bahwa file statistik Snort tidak ditemukan.
- Hasil `snort -T` yang tersedia hanya memperlihatkan proses test berjalan dan belum memperlihatkan ringkasan akhir.

Karena itu, status proses engine Snort pada sesi Minggu 10 tetap **belum terverifikasi**. Jangan menyamakan status agent dengan bukti proses engine yang berjalan.