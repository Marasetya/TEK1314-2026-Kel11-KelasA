# IP Address Plan & Arsitektur Jaringan Lab — SOC BiliCheck

## 1. Identitas Kelompok & Informasi Dokumen

| Atribut Dokumen | Keterangan Lengkap |
| :--- | :--- |
| **Mata Kuliah** | TEK1314 — Keamanan Siber (D4 Teknologi Rekayasa Komputer, Sekolah Vokasi IPB) |
| **Kelompok & Kelas** | **Kelompok 11 — Kelas A** |
| **Nama Proyek IoT** | **BiliCheck** (*Bilirubin Clinical Health Evaluator* — Tim AGRION / Kelompok 9 Proyek Sistem IoT) |
| **Repositori GitHub** | `TEK1314-2026-Kel11-KelasA` (`https://github.com/Marasetya/TEK1314-2026-Kel11-KelasA`) |
| **Nama Berkas & Path** | `ip_plan.md` (Lokasi di repositori: `docs/design/ip_plan.md`) |
| **Jenis Dokumen (Apa Ini?)** | **Dokumen Perencanaan Pengalamatan IP, Konfigurasi Antarmuka VirtualBox, Tabel Routing, & Pemetaan Port Layanan Lab** |
| **Fungsi & Tujuan Dokumen** | Menetapkan skema pengalamatan IPv4 statis pada jaringan internal terisolasi (`bilicheck-lab`), memetakan antarmuka jaringan (NIC) dan MAC Address ketiga mesin virtual, serta mendokumentasikan hasil verifikasi konektivitas dan eksposur port aktual sebelum memasuki pengujian Fase 2. |
| **Tahap Pelaporan** | Checkpoint Minggu ke-9 (Transisi Fase 1 Hardening & Baseline Menuju Fase 2 Vulnerability Assessment) |
| **Tanggal Pemutakhiran** | 22 Oktober 2026 |

### Daftar Anggota Kelompok 11 (Kelas A) & Pembagian Peran SOC

| No. | Kelas | Nama Mahasiswa | NIM | Role (Peran Tim) | Tanggung Jawab Utama pada Proyek SOC BiliCheck |
| :--- | :--- | :--- | :--- | :--- | :--- |
| 1 | A | **Moh Rifky Marasetya** | `J0404231049` | **Lead** | Koordinasi arsitektur simulasi, penyelarasan *Tech Stack Declaration*, pengelolaan repositori GitHub, dan integrasi laporan NIST SP 800-61r2 |
| 2 | A | **Nayla Januarisa Prihadi** | `J0404241149` | **Blue Team** *(Defender / SIEM)* | Konfigurasi *Monitoring Node* Security Onion (`192.168.11.200`), pemulihan Elastic Stack, aktivasi sensor NIDS Snort/Sguil, dan analisis *baseline traffic* |
| 3 | A | **Muhammad Adam Ilyasa** | `J0404241048` | **Blue Team** *(Defender / Host)* | Penanggung jawab *Laptop Utama*, instalasi & eksekusi 9 kontrol *hardening* pada *Target Node* Ubuntu Server 24.04.5 (`192.168.11.5`), serta dokumentasi bukti *before/after* |
| 4 | A | **Fauzan Nafis Jiyad Akbar** | `J0404241152` | **Red Team** *(Attacker)* | Penanggung jawab *Laptop Backup*, konfigurasi *Attacker Node* Kali Linux (`192.168.11.100`), pemetaan permukaan layanan (*reconnaissance*), dan perancangan 3 skenario *Attack Plan* Fase 2 |

## 2. Spesifikasi Segmen Jaringan Laboratorium Terisolasi

Seluruh kegiatan simulasi keamanan siber dilakukan di dalam jaringan internal Oracle VM VirtualBox agar lalu lintas pengujian antara *Attacker Node* dan *Target Node* dapat disadap secara utuh oleh *Monitoring Node* (Security Onion) tanpa menyentuh jaringan kampus maupun infrastruktur produksi milik Mitra (Bapak Aep Setiawan, S.Si., M.Si.).

| Parameter Jaringan | Segmen Jaringan Simulasi Lab (Utama) | Segmen Jaringan Manajemen & Update (NAT) |
| :--- | :--- | :--- |
| **Nama Jaringan VirtualBox** | `bilicheck-lab` (*Internal Network*) | *VirtualBox NAT Network* (Per-VM Adapter) |
| **Network Address & Prefix** | `192.168.11.0/24` | `10.0.2.0/24` |
| **Subnet Mask** | `255.255.255.0` | `255.255.255.0` |
| **Rentang IP Host Valid** | `192.168.11.1` – `192.168.11.254` | `10.0.2.15` (Alokasi DHCP internal VirtualBox) |
| **Broadcast Address** | `192.168.11.255` | `10.0.2.255` |
| **Default Gateway** | *Tidak ditetapkan* (`never-default` / tanpa gateway agar trafik lab tidak bocor keluar) | `10.0.2.2` (Digunakan khusus pada Ubuntu `enp0s3` & Security Onion `eth2` untuk pembaruan paket) |
| **Layanan DNS** | Resolusi lokal / pengalamatan IP langsung | `127.0.0.53` (`systemd-resolved` pada Ubuntu) & `8.8.8.8` (pada Security Onion) |

## 3. Alokasi Alamat IP, MAC Address, dan Konfigurasi Adapter VirtualBox

Tabel berikut merangkum konfigurasi jaringan aktual yang telah diterapkan dan diverifikasi melalui tangkapan layar terminal pada ketiga mesin virtual Kelompok 11:

| Peran Node & Nama VM | Sistem Operasi & Hostname | Antarmuka (NIC) | Mode Adapter VirtualBox | Alamat IPv4 Aktual | MAC Address Terverifikasi | Mode Promiscuous & Fungsi Spesifik |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **Target Node**<br>*(Replika Backend)*<br>VM: `Ubuntu-SOC-BiliCheck-Target` | Ubuntu Server 24.04.5 LTS<br>Kernel `6.8.0-147-generic`<br>Hostname: `target-bilicheck` | `enp0s3`<br>*(Adapter 1)* | **NAT**<br>+ Port Forwarding `127.0.0.1:2222 -> :22` | `10.0.2.15/24`<br>*(DHCP)* | `08:00:27:61:e5:44` | **Deny** — Jalur akses administrasi SSH dari host Windows (`port 2222`) dan unduhan patch keamanan OS (`apt upgrade`). |
| **Target Node**<br>*(Replika Backend)* | Ubuntu Server 24.04.5 LTS<br>Hostname: `target-bilicheck` | `enp0s8`<br>*(Adapter 2)* | **Internal Network**<br>Nama: `bilicheck-lab` | **`192.168.11.5/24`**<br>*(Statis via Netplan)* | `08:00:27:47:9f:01` | **Deny** — Antarmuka utama target simulasi tempat layanan backend BiliCheck diuji oleh Red Team dan dipantau oleh Blue Team. |
| **Attacker Node**<br>*(Red Team)*<br>VM: `attacker-kali` | Kali Linux 2026.2<br>Hostname: `attacker-kali` | `eth0`<br>*(Adapter 1)* | **Internal Network**<br>Nama: `bilicheck-lab` | **`192.168.11.100/24`**<br>*(Statis via nmcli)* | `08:00:27:32:c8:da` *(lokal)* | **Deny** — Sumber pembangkit trafik *reconnaissance* (`nmap`), audit protokol IoT, dan pengujian autentikasi terkontrol. |
| **Monitoring Node**<br>*(Blue Team SIEM)*<br>VM: `SO-BiliCheck-Monitoring` | Security Onion Classic<br>(Ubuntu 16.04.6 LTS)<br>Hostname: `SecOnion` | `eth0`<br>*(Adapter 1)* | **Internal Network**<br>Nama: `bilicheck-lab` | **Tanpa IP (`UP`)**<br>*(Mode Manual/Sniffing)* | `08:00:27:43:8b:2b` | **Allow All** — Antarmuka penyadapan pasif (*promiscuous mode on, arp off, offload off*) yang terhubung ke sensor `seconion-import` (Snort NIDS & `tcpdump`). |
| **Monitoring Node**<br>*(Blue Team SIEM)* | Security Onion Classic<br>Hostname: `SecOnion` | `eth1`<br>*(Adapter 2)* | **Internal Network**<br>Nama: `bilicheck-lab` | **`192.168.11.200/24`**<br>*(Statis via interfaces)* | `08:00:27:a9:0d:c0` | **Deny** — Antarmuka manajemen Security Onion pada subnet laboratorium. |
| **Monitoring Node**<br>*(Blue Team SIEM)* | Security Onion Classic<br>Hostname: `SecOnion` | `eth2`<br>*(Adapter 3)* | **NAT** | `10.0.2.15/24`<br>*(DHCP)* | `08:00:27:13:d4:5b` | **Deny** — Jalur *default route* (`via 10.0.2.2 dev eth2`) untuk pemeliharaan kontainer Elastic Stack dan sinkronisasi sistem. |

## 4. Konfigurasi Tabel Routing Aktual per Node

Berdasarkan hasil verifikasi pasca-*reboot* pada masing-masing mesin virtual, konfigurasi tabel routing IPv4 telah dipastikan bebas dari konflik *default gateway* ganda:

1. **Target Node (`target-bilicheck` — `192.168.11.5`)**:
   * `default via 10.0.2.2 dev enp0s3 proto dhcp src 10.0.2.15 metric 100` (Akses keluar melalui NAT).
   * `192.168.11.0/24 dev enp0s8 proto kernel scope link src 192.168.11.5` (Trafik segmen lab diarahkan sepenuhnya melalui `enp0s8` berdasarkan `/etc/netplan/99-bilicheck-lab.yaml`).
2. **Attacker Node (`attacker-kali` — `192.168.11.100`)**:
   * `192.168.11.0/24 dev eth0 proto kernel scope link src 192.168.11.100 metric 100` (Terisolasi penuh pada jaringan internal dengan parameter `ipv4.never-default yes`).
3. **Monitoring Node (`SecOnion` — `192.168.11.200`)**:
   * `default via 10.0.2.2 dev eth2 proto static metric 100` (Rute keluar melalui NAT `eth2`).
   * `192.168.11.0/24 dev eth1 proto kernel scope link src 192.168.11.200` (Rute manajemen lab melalui `eth1`, sementara `eth0` beroperasi murni pada Layer 2 untuk menyadap paket).

## 5. Matriks Alur Komunikasi & Visibilitas Pemantauan SOC

| Node Sumber | Node Tujuan | Protokol & Port | Jenis Trafik Simulasi | Mekanisme Visibilitas di Security Onion |
| :--- | :--- | :--- | :--- | :--- |
| **Kali Linux**<br>(`192.168.11.100`) | **Ubuntu Target**<br>(`192.168.11.5`) | ICMP, TCP (`22, 80, 1883, 3306, 5000, 8080, 9099`) | Trafik uji konektivitas (*ping*), pemindaian port (`nmap`), dan skenario pengujian Red Team sesuai `attack_plan.md`. | Disadap secara pasif oleh antarmuka **`eth0`** (`Promiscuous Mode: Allow All`) pada vSwitch `bilicheck-lab` dan dianalisis oleh engine **Snort (`snort-1`)** menuju **Sguil**. |
| **Host Windows**<br>(`127.0.0.1:2222`) | **Ubuntu Target**<br>(`10.0.2.15:22`) | TCP Port `22` *(OpenSSH Key-Based)* | Administrasi sistem oleh Blue Team melalui *Port Forwarding* VirtualBox (`SSH-2222`). | Dicatat secara lokal pada `/var/log/auth.log`, `systemd-journald` persisten, dan diawasi oleh daemon `fail2ban` serta `auditd`. |
| **Mock ESP32 / Client**<br>(`192.168.11.x`) | **Ubuntu Target**<br>(`192.168.11.5`) | TCP Port `1883` *(MQTT)* & `80` *(HTTP)* | Trafik *baseline* telemetri nilai ADC/Lux bilirubin *dummy* serta pengujian otorisasi topik MQTT pada Fase 2. | Direkam melalui *packet capture* (`pcap_agent`) pada `eth0` Security Onion dan log lokal `/var/log/mosquitto/mosquitto.log`. |

## 6. Inventaris Layanan dan Port pada Target Node (`192.168.11.5`)

Sesuai prinsip pelaporan berbasis bukti, tabel berikut memisahkan secara tegas antara **port layanan yang sudah aktif pada Checkpoint Minggu ke-9** (berdasarkan output `sudo ss -lntup` dan pemindaian `nmap` dari Kali Linux) dan **port layanan aplikasi replika BiliCheck yang dijadwalkan untuk diaktifkan pada Fase 2**:

| Layanan / Daemon | Protokol & Port | Alamat Bind (Listen) | Status di Ubuntu (Minggu 9) | Hasil Scan Nmap dari Kali (`.100`) | Kebijakan Firewall UFW & Keterangan Bukti |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **OpenSSH Server (`sshd`)** | TCP `22` | `0.0.0.0:22` & `[::]:22` | **Aktif (`LISTEN`)** | **`open`** (`syn-ack`) | **Diizinkan (`ALLOW IN`)** — Dilindungi *SSH Key-Based Auth*, `PermitRootLogin no`, `PasswordAuthentication no`, dan `Fail2Ban`. Bukti: `30-listening-ports-after.png`. |
| **Systemd-Resolved** | UDP/TCP `53` | `127.0.0.53:53` & `127.0.0.54:53` | **Aktif Lokal (`LISTEN`)** | Tidak terekspos ke jaringan | Hanya mendengarkan pada antarmuka *loopback* lokal (`lo`) untuk resolusi DNS internal host. |
| **Postfix Mail (`master`)** | TCP `25` | `127.0.0.1:25` & `[::1]:25` | **Aktif Lokal (`LISTEN`)** | Tidak terekspos ke jaringan | Terpasang sebagai dependensi notifikasi paket keamanan (`bsd-mailx`); hanya terikat pada *localhost* (`127.0.0.1`) dan diblokir dari jaringan luar oleh UFW. |
| **Nginx / Web API BiliCheck** | TCP `80` / `5000` | Rencana: `192.168.11.5` | **Direncanakan (Fase 2)** | **`filtered`** (`no-response`) | Diblokir oleh kebijakan `Default: deny (incoming)` UFW. Akan dipasang dan dibuka terbatas untuk subnet `192.168.11.0/24` saat pengujian Skenario 2. |
| **Mosquitto MQTT Broker** | TCP `1883` | Rencana: `192.168.11.5:1883` | **Direncanakan (Fase 2)** | **`filtered`** (`no-response`) | Diblokir oleh UFW pada baseline OS. Akan diaktifkan dengan autentikasi wajib (`allow_anonymous false`) untuk simulasi telemetri ESP32 *dummy*. |
| **MySQL Server 8.0** | TCP `3306` | Rencana: `127.0.0.1:3306` | **Direncanakan (Fase 2)** | **`filtered`** (`no-response`) | Wajib di-bind ke `127.0.0.1` (localhost) dan **tidak pernah dibuka di UFW** agar tidak dapat diakses langsung dari jaringan eksternal. |
| **Firebase Local Emulator** | TCP `8080` & `9099` | Rencana: `192.168.11.5` | **Direncanakan (Fase 2)** | **`filtered`** (`no-response`) | Mereplikasi Cloud Firestore (`8080`) dan Firebase Auth (`9099`) secara lokal menggunakan 100% data pasien sintetis. |

## 7. Ringkasan Bukti Verifikasi Jaringan & Konektivitas (Minggu ke-9)

Seluruh rancangan pada dokumen ini telah diverifikasi secara langsung di laboratorium dengan bukti artefak berikut:

1. **Verifikasi Konfigurasi Adapter & IP Target Node**: Tersimpan pada `docs/blueteam/evidence/hardening/08-virtualbox-adapter1-nat dan 09-virtualbox-port-forwarding.png`, `B04-virtualbox-adapter2-bilicheck-lab.png`, dan `06-apt-update-upgrade dan 07-post-upgrade-network.png`.
2. **Verifikasi IP Statis & Konektivitas dari Attacker Node (`192.168.11.100`)**: Tersimpan pada `docs/redteam/evidence/01-kali-network.png` dan `02-ping-ubuntu dan 03-neighbor-ubuntu.png` (menunjukkan `4 packets transmitted, 4 received, 0% packet loss` serta resolusi ARP `192.168.11.5 dev eth0 lladdr 08:00:27:47:9f:01 REACHABLE`).
3. **Verifikasi Penyadapan Paket & Deteksi IDS pada Monitoring Node (`192.168.11.200`)**: Tersimpan pada `docs/blueteam/evidence/monitoring-seconion/SO-02-ip-routing-after-reboot.png`, `SO-05-icmp-baseline-capture-full.png` (`10 packets captured, 0 packets dropped`), dan `SO-08-sguil-realtime-events-detected.png` (deteksi *real-time* trafik ping dan scan dari `.100` ke `.5` pada aplikasi Sguil).
