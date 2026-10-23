# Hardening Checklist & Verifikasi Baseline Monitoring, SOC BiliCheck

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
| **Tanggal Pemutakhiran** | 23 Oktober 2026 |

### Daftar Anggota Kelompok 11 (Kelas A) & Pembagian Peran SOC

| No. | Kelas | Nama Mahasiswa | NIM | Role (Peran Tim) | Tanggung Jawab Utama pada Proyek SOC BiliCheck |
| :--- | :--- | :--- | :--- | :--- | :--- |
| 1 | A | **Moh Rifky Marasetya** | `J0404231049` | **Lead** | Koordinasi arsitektur simulasi, penyelarasan *Tech Stack Declaration*, pengelolaan repositori GitHub, dan integrasi laporan NIST SP 800-61r2 |
| 2 | A | **Nayla Januarisa Prihadi** | `J0404241149` | **Blue Team** *(Defender / SIEM)* | Konfigurasi *Monitoring Node* Security Onion (`192.168.11.200`), pemulihan Elastic Stack, aktivasi sensor NIDS Snort/Sguil, dan analisis *baseline traffic* |
| 3 | A | **Muhammad Adam Ilyasa** | `J0404241048` | **Blue Team** *(Defender / Host)* | Penanggung jawab *Laptop Utama*, instalasi & eksekusi 9 kontrol *hardening* pada *Target Node* Ubuntu Server 24.04.5 (`192.168.11.5`), serta dokumentasi bukti *before/after* |
| 4 | A | **Fauzan Nafis Jiyad Akbar** | `J0404241152` | **Red Team** *(Attacker)* | Penanggung jawab *Laptop Backup*, konfigurasi *Attacker Node* Kali Linux (`192.168.11.100`), pemetaan permukaan layanan (*reconnaissance*), dan perancangan 3 skenario *Attack Plan* Fase 2 |

## 2. Ringkasan Pemenuhan Standar Minimum Hardening (Bagian VIII Panduan PBL)

Sesuai ketentuan **Bagian VIII Panduan PBL Pertemuan 9**, setiap kelompok wajib menerapkan dan mendokumentasikan **minimal 8 kontrol hardening** yang tersebar di tiga kategori (**minimal 2 kontrol per kategori**) dengan bukti *before/after* yang nyata.

Pada Checkpoint Minggu ke-9 ini, Blue Team Kelompok 11 telah **menyelesaikan dan memverifikasi 9 kontrol hardening aktual pada Target Node Ubuntu Server 24.04.5 LTS (`192.168.11.5`)** yang mencakup ketiga kategori wajib (3 kontrol di Kategori A, 3 kontrol di Kategori B, dan 3 kontrol di Kategori C), serta menyiapkan **3 kontrol tambahan tingkat layanan aplikasi (A-04, B-04, C-04)** yang akan diterapkan bersamaan dengan instalasi daemon aplikasi BiliCheck pada Fase 2. Selain itu, **1 kontrol aplikasi baru (C-05, Validasi Input Telemetri ADC pada API Replika)** sedang dalam implementasi dan baru akan ditandai selesai setelah pengujian aktual berhasil:

* **Kategori A, Access & Authentication**: **3 Kontrol Selesai** (`A-01` OpenSSH Key-Based & Root Disable, `A-02` Fail2Ban Intrusion Prevention, `A-03` Penguncian Root, Sudoers Least-Privilege & PAM `pwquality`) + **1 Kontrol Rencana Fase 2** (`A-04` Autentikasi Broker MQTT Mosquitto).
* **Kategori B, Service & Attack Surface**: **3 Kontrol Selesai** (`B-01` Firewall Host UFW Default-Deny, `B-02` Patching Keamanan OS & `unattended-upgrades`, `B-03` Kernel Network Hardening `sysctl` & Minimasi Port Aktif) + **1 Kontrol Rencana Fase 2** (`B-04` Isolasi Localhost Binding MySQL `127.0.0.1:3306`).
* **Kategori C, Data, Logging & System Integrity**: **3 Kontrol Selesai** (`C-01` Security Auditing `auditd` 11 Rules pada File Sensitif, `C-02` Mandatory Access Control `AppArmor` Enforce Mode, `C-03` Pemeriksaan Integritas Filesystem `AIDE`, Persistent Logging `journald`, & Sinkronisasi NTP) + **1 Kontrol Rencana Fase 2** (`C-04` Proteksi Kredensial `.env` & Least-Privilege Database) + **1 Kontrol Dalam Implementasi** (`C-05` Validasi Input Telemetri ADC `0–4095` pada API Replika).

## 3. Matriks Utama Hardening Checklist (Target Node: `192.168.11.5`)

Seluruh berkas bukti disimpan di dalam direktori `docs/blueteam/evidence/hardening/` dengan penamaan berkas berurutan `01` s.d. `25`:

| ID | Kategori Wajib | Kontrol Hardening Spesifik | Kondisi *Before* (Baseline Awal) | Kondisi *After* (Hasil Verifikasi Aktual) | Berkas Bukti Wajib di `docs/blueteam/evidence/hardening/` (*Before vs After*) | Status Minggu 9 |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **A-01** | **A. Access & Authentication** | **Penonaktifan Login Root, Kewajiban SSH Key-Based Auth, & Pembatasan Sesi SSH** (`/etc/ssh/sshd_config.d/00-bilicheck-hardening.conf`) | `permitrootlogin without-password`<br>`passwordauthentication yes`<br>`maxauthtries 6`<br>`logingracetime 120`<br>`x11forwarding yes` | `permitrootlogin no`<br>`passwordauthentication no`<br>`pubkeyauthentication yes`<br>`kbdinteractiveauthentication no`<br>`maxauthtries 3`, `logingracetime 30`<br>`x11forwarding no`, `allowusers socadmin` | **Before:** `03-sebelum-status-ssh-dan-port.png`, `06-sebelum-laporan-baseline-dibuat.png`<br>**After:** `13-sesudah-uji-login-ssh-key.png`, `20-sesudah-log-journald-persisten-dan-log-ssh.png` | **Selesai, Ada Bukti** |
| **A-02** | **A. Access & Authentication** | **Proteksi Brute-Force SSH dengan Fail2Ban** (`/etc/fail2ban/jail.d/bilicheck.local`) | Paket `fail2ban` belum terpasang; tidak ada pemblokiran otomatis terhadap kegagalan login berulang. | `fail2ban.service` aktif (`running`), `Number of jail: 1` (`sshd`), `backend = systemd`, memantau unit `sshd.service` (`maxretry = 5`, `findtime = 10m`, `bantime = 1h`). | **Before:** `05-sebelum-status-firewall-dan-layanan.png`, `06-sebelum-laporan-baseline-dibuat.png`<br>**After:** `11-sesudah-instalasi-paket-keamanan.png`, `16-sesudah-fail2ban-ssh-aktif.png` | **Selesai, Ada Bukti** |
| **A-03** | **A. Access & Authentication** | **Penguncian Akun Root Lokal, Pembatasan Hak Sudo, & Kebijakan Kualitas Password PAM** (`/etc/security/pwquality.conf`) | Hanya mengandalkan pengaturan bawaan instalasi; kebijakan kompleksitas password PAM belum dikonfigurasi khusus. | Akun `root` terkunci (`passwd -S root` -> `L`), grup `sudo` dibatasi hanya `socadmin`, izin `/etc/shadow` & `/etc/gshadow` `640` (`root:shadow`), `/etc/sudoers` `440`, serta `libpam-pwquality` dikonfigurasi (`minlen = 14`, `retry = 3`). | **Before:** `04-sebelum-akun-sudo-dan-izin-file.png`<br>**After:** `11-sesudah-instalasi-paket-keamanan.png`, `12-sesudah-kunci-root-dan-izin-sudo.png`, `22-sesudah-kebijakan-kualitas-password-pam.png` | **Selesai, Ada Bukti** |
| **A-04** | **A. Access & Authentication** | **Autentikasi Wajib pada Broker MQTT Mosquitto (`allow_anonymous false`)** | Broker MQTT secara bawaan mengizinkan koneksi/publikasi anonim tanpa kredensial. | Konfigurasi `/etc/mosquitto/conf.d/bilicheck-auth.conf` dengan `allow_anonymous false` dan `password_file /etc/mosquitto/passwd` untuk akun `esp32_bilicheck`. | *Akan diambil saat instalasi broker Mosquitto pada eksekusi Skenario 2 (Fase 2)* | **Direncanakan (Fase 2)** |
| **B-01** | **B. Service & Attack Surface** | **Firewall Host UFW dengan Kebijakan `Default Deny Incoming` & Whitelist Port** | `Status: inactive` (Tidak ada penyaringan paket masuk pada tingkat host). | `Status: active`, `Logging: on (low)`, `Default: deny (incoming), allow (outgoing), disabled (routed)`. Hanya membuka port `22/tcp` (IPv4 & IPv6). Hasil scan `nmap` dari Kali membuktikan port `80, 1883, 8080` berstatus `filtered`. | **Before:** `05-sebelum-status-firewall-dan-layanan.png`, `14-sebelum-aturan-firewall-ufw-diaktifkan.png`<br>**After:** `15-sesudah-firewall-ufw-aktif-izinkan-ssh.png` | **Selesai, Ada Bukti** |
| **B-02** | **B. Service & Attack Surface** | **Pembaruan Patch Keamanan OS (`apt full-upgrade`) & Aktivasi Pembaruan Otomatis (`unattended-upgrades`)** | Terdapat paket sistem yang belum diperbarui dan mekanisme pembaruan keamanan otomatis belum diverifikasi. | Seluruh paket keamanan diperbarui, timer `apt-daily.timer` & `apt-daily-upgrade.timer` aktif, serta `/etc/apt/apt.conf.d/20auto-upgrades` bernilai `Update-Package-Lists "1"` dan `Unattended-Upgrade "1"`. | **Before:** `01-sebelum-identitas-server-ubuntu.png`, `06-sebelum-laporan-baseline-dibuat.png`<br>**After:** `09-sesudah-update-paket-os-dan-jaringan.png`, `21-sesudah-konfigurasi-update-otomatis.png` | **Selesai, Ada Bukti** |
| **B-03** | **B. Service & Attack Surface** | **Hardening Parameter Jaringan & Kernel (`sysctl`) serta Audit Minimasi Port/Layanan Aktif** (`/etc/sysctl.d/99-bilicheck-hardening.conf`) | Parameter proteksi jaringan dan kernel masih menggunakan nilai standar; inventarisasi layanan belum diaudit. | `kernel.randomize_va_space = 2`, `kptr_restrict = 2`, `dmesg_restrict = 1`, `fs.protected_hardlinks = 1`, `fs.protected_symlinks = 1`, `net.ipv4.tcp_syncookies = 1`, `accept_redirects = 0`, `send_redirects = 0`, `accept_source_route = 0`, `rp_filter = 1`. Hanya port `22/tcp` yang *listening* ke jaringan luar. | **Before:** `03-sebelum-status-ssh-dan-port.png`, `05-sebelum-status-firewall-dan-layanan.png`<br>**After:** `17-sesudah-hardening-kernel-sysctl.png`, `24-sesudah-daftar-layanan-berjalan.png`, `25-sesudah-daftar-port-terbuka.png` | **Selesai, Ada Bukti** |
| **B-04** | **B. Service & Attack Surface** | **Isolasi Binding Database MySQL ke Localhost (`127.0.0.1:3306`) & `mysql_secure_installation`** | Risiko tereksposnya port `3306/tcp` ke jaringan eksternal dan keberadaan akun/database uji bawaan. | Menetapkan `bind-address = 127.0.0.1` pada `/etc/mysql/mysql.conf.d/mysqld.cnf`, menghapus akun anonim, dan menonaktifkan login root jarak jauh. | *Akan diambil saat instalasi MySQL Server pada Fase 2* | **Direncanakan (Fase 2)** |
| **C-01** | **C. Data & Application** | **Aktivasi Security Auditing (`auditd`) untuk Pemantauan Perubahan File Identitas & Konfigurasi** (`/etc/audit/rules.d/bilicheck.rules`) | `sudo: auditctl: command not found` (Layanan `auditd` belum terpasang dan tidak ada aturan audit perubahan berkas). | `auditd.service` aktif (`enabled 1`, `pid 589/1728`) dengan **11 aturan audit aktif** yang memantau akses tulis/atribut (`-p wa`) pada `/etc/passwd`, `/etc/group`, `/etc/shadow`, `/etc/gshadow` (`identity`), `/etc/sudoers` (`scope`), `/etc/ssh/sshd_config` (`ssh_config`), `/etc/netplan` (`network_config`), `/etc/ufw` (`firewall_config`), dan `/etc/sysctl.d` (`sysctl_config`). | **Before:** `06-sebelum-laporan-baseline-dibuat.png`<br>**After:** `18-sesudah-layanan-auditd-dan-aturan-aktif.png` | **Selesai, Ada Bukti** |
| **C-02** | **C. Data & Application** | **Penegakan Mandatory Access Control (`AppArmor`) pada Layanan Sistem** | Profil AppArmor belum ditinjau secara menyeluruh pasca-instalasi paket keamanan tambahan. | Modul `apparmor` aktif dengan **117 profil dimuat** (`23 profiles in enforce mode`, termasuk `rsyslogd`, `tcpdump`, `unix-chkpwd`, `snap-confine`) dan `0 unconfined processes` yang memiliki profil terdefinisi. | **Before:** `06-sebelum-laporan-baseline-dibuat.png`<br>**After:** `19-sesudah-status-apparmor-aktif.png` | **Selesai, Ada Bukti** |
| **C-03** | **C. Data & Application** | **Pemeriksaan Integritas Filesystem (`AIDE`), Persistent Logging (`systemd-journald`), & Sinkronisasi Waktu NTP (`Asia/Jakarta`)** | Log jurnal belum dikonfigurasi persisten dengan batas retensi khusus, zona waktu masih `UTC (+00:00)`, dan database integritas AIDE belum ada. | Zona waktu diselaraskan ke `Asia/Jakarta (WIB, +0700)` dengan `NTP service: active` (`185.125.190.56`), log `journald` disimpan persisten (`/etc/systemd/journald.conf.d/99-bilicheck.conf`, `44.1M`), serta database **AIDE (`0.18.6`)** memverifikasi **102.127 entri filesystem**. | **Before:** `06-sebelum-laporan-baseline-dibuat.png`<br>**After:** `10-sesudah-sinkronisasi-zona-waktu-ntp.png`, `20-sesudah-log-journald-persisten-dan-log-ssh.png`, `23-sesudah-cek-integritas-file-aide.png` | **Selesai, Ada Bukti** |
| **C-04** | **C. Data & Application** | **Pemisahan Kredensial Aplikasi ke `.env` (`chmod 600`) & Least-Privilege Database/Firestore RBAC** | Kredensial tertanam di dalam skrip aplikasi dan akun database belum dibatasi hak aksesnya. | Kredensial disimpan pada `.env` (`chmod 600`, masuk `.gitignore`) dan akun MySQL dibatasi hanya `SELECT, INSERT` (`bili_app@localhost`). | *Akan diambil saat instalasi MySQL Server dan pengujian Fase 2* | **Direncanakan (Fase 2)** |
| **C-05** | **C. Data & Application** | **Validasi Input Telemetri ADC pada API Replika BiliCheck** | Endpoint penerima telemetri belum menerapkan validasi tipe data dan rentang ADC 12-bit. | Endpoint API replika memvalidasi `rawAdc` sebagai bilangan bulat dalam rentang `0–4095`, menolak nilai di luar rentang atau tipe data yang salah, serta dijalankan menggunakan user layanan non-root. | **After:** `docs/blueteam/evidence/application/26-validasi-input-rawadc-pada-api.png`, `27-status-layanan-api-dan-port-5000.png`, `28-hasil-pengujian-validasi-input-rawadc.png` | **Dalam Implementasi (tandai Selesai setelah pengujian aktual berhasil)** |

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
| 5 | **Deteksi Event Real-Time pada Konsol Sguil (`SGUIL-0.9.0`)** | Konsol Sguil berhasil menampilkan **alert *real-time* terbaru pada tanggal `2026-10-22` (`07:35:07` – `07:46:11` UTC)** dari IP sumber `192.168.11.100` menuju `192.168.11.5`, meliputi:<br>• `ET SCAN Potential SSH Scan` (Port `22/tcp`, Alert ID `5.1182` & `5.1183`)<br>• `ET SCAN Suspicious inbound` ke port MySQL `3306` (`5.1191`), `1521` (`5.1193`), `1433` (`5.1196`), `5432` (`5.1199`), & VNC `5902/5815`<br>• `GPL ICMP_INFO PING *NIX` (Alert ID `5.1201`, `CNT: 73`). | `06-deteksi-alert-scan-dan-ping-di-sguil.png` | **Selesai (Terverifikasi)** |
