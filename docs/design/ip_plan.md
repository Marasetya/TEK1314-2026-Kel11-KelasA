# IP Address Plan & Arsitektur Jaringan Lab, SOC BiliCheck

> **Keterangan file:** Dokumen pengalamatan IP, adapter VirtualBox, routing, alur komunikasi, dan inventaris port layanan.
> **Isi utama:** Topologi tiga VM, alamat IP, interface, jalur NAT, jaringan internal, dan status layanan.
> **Penanggung jawab:** Koordinasi Lead Moh Rifky Marasetya dengan verifikasi teknis seluruh anggota. Pengamanan host dan monitoring oleh Adam.
> **Status dokumen:** Dokumen desain yang diperbarui mengikuti kondisi lab yang terverifikasi.

## 1. Identitas Kelompok & Informasi Dokumen

| Atribut Dokumen | Keterangan Lengkap |
| :--- | :--- |
| **Mata Kuliah** | TEK1314, Keamanan Siber (D4 Teknologi Rekayasa Komputer, Sekolah Vokasi IPB) |
| **Kelompok & Kelas** | **Kelompok 11, Kelas A** |
| **Nama Proyek IoT** | **BiliCheck** (*Bilirubin Clinical Health Evaluator*, Tim AGRION / Kelompok 9 Proyek Sistem IoT) |
| **Repositori GitHub** | `TEK1314-2026-Kel11-KelasA` (`https://github.com/Marasetya/TEK1314-2026-Kel11-KelasA`) |
| **Nama Berkas & Path** | `ip_plan.md` (Lokasi di repositori: `docs/design/ip_plan.md`) |
| **Jenis Dokumen (Apa Ini?)** | **Dokumen Perencanaan Pengalamatan IP, Konfigurasi Antarmuka VirtualBox, Tabel Routing, & Pemetaan Port Layanan Lab** |
| **Fungsi & Tujuan Dokumen** | Menetapkan skema pengalamatan IPv4 statis pada jaringan internal terisolasi (`bilicheck-lab`), memetakan antarmuka jaringan (NIC) dan MAC Address ketiga mesin virtual, serta mendokumentasikan hasil verifikasi konektivitas dan eksposur port aktual sebelum memasuki pengujian Fase 2. |
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

## 2. Spesifikasi Segmen Jaringan Laboratorium Terisolasi

Seluruh kegiatan simulasi keamanan siber dilakukan di dalam jaringan internal Oracle VM VirtualBox agar lalu lintas pengujian antara *Attacker Node* dan *Target Node* dapat disadap secara utuh oleh *Monitoring Node* (Security Onion) tanpa menyentuh jaringan kampus maupun infrastruktur produksi milik Mitra (Bapak Aep Setiawan, S.Si., M.Si.).

| Parameter Jaringan | Segmen Jaringan Simulasi Lab (Utama) | Segmen Jaringan Manajemen & Update (NAT) |
| :--- | :--- | :--- |
| **Nama Jaringan VirtualBox** | `bilicheck-lab` (*Internal Network*) | *VirtualBox NAT Network* (Per-VM Adapter) |
| **Network Address & Prefix** | `192.168.11.0/24` | `10.0.2.0/24` |
| **Subnet Mask** | `255.255.255.0` | `255.255.255.0` |
| **Rentang IP Host Valid** | `192.168.11.1` - `192.168.11.254` | `10.0.2.15` (Alokasi DHCP internal VirtualBox) |
| **Broadcast Address** | `192.168.11.255` | `10.0.2.255` |
| **Default Gateway** | *Tidak ditetapkan* (`never-default` / tanpa gateway agar trafik lab tidak bocor keluar) | `10.0.2.2` (Digunakan khusus pada Ubuntu `enp0s3` & Security Onion `eth2` untuk pembaruan paket) |
| **Layanan DNS** | Resolusi lokal / pengalamatan IP langsung | `127.0.0.53` (`systemd-resolved` pada Ubuntu) & `8.8.8.8` (pada Security Onion) |

## 3. Alokasi Alamat IP, MAC Address, dan Konfigurasi Adapter VirtualBox

Tabel berikut merangkum konfigurasi jaringan aktual yang telah diterapkan dan diverifikasi melalui tangkapan layar terminal pada ketiga mesin virtual Kelompok 11:

| Peran Node & Nama VM | Sistem Operasi & Hostname | Antarmuka (NIC) | Mode Adapter VirtualBox | Alamat IPv4 Aktual | MAC Address Terverifikasi | Mode Promiscuous & Fungsi Spesifik |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **Target Node**<br>*(Replika Backend)*<br>VM: `Ubuntu-SOC-BiliCheck-Target` | Ubuntu Server 24.04.5 LTS<br>Kernel `6.8.0-147-generic`<br>Hostname: `target-bilicheck` | `enp0s3`<br>*(Adapter 1)* | **NAT**<br>+ Port Forwarding `127.0.0.1:2222 -> :22` | `10.0.2.15/24`<br>*(DHCP)* | `08:00:27:61:e5:44` | **Deny**, Jalur akses administrasi SSH dari host Windows (`port 2222`) dan unduhan patch keamanan OS (`apt upgrade`). Bukti: `07-virtualbox-adapter1-nat-dan-port-forwarding.png`. |
| **Target Node**<br>*(Replika Backend)* | Ubuntu Server 24.04.5 LTS<br>Hostname: `target-bilicheck` | `enp0s8`<br>*(Adapter 2)* | **Internal Network**<br>Nama: `bilicheck-lab` | **`192.168.11.5/24`**<br>*(Statis via Netplan)* | `08:00:27:47:9f:01` | **Deny**, Antarmuka utama target simulasi tempat layanan backend BiliCheck diuji oleh Red Team dan dipantau oleh Blue Team. Bukti: `08-virtualbox-adapter2-jaringan-internal-lab.png`. |
| **Attacker Node**<br>*(Red Team)*<br>VM: `attacker-kali` | Kali Linux 2026.2<br>Hostname: `attacker-kali` | `eth0`<br>*(Adapter 1)* | **Internal Network**<br>Nama: `bilicheck-lab` | **`192.168.11.100/24`**<br>*(Statis via nmcli)* | `08:00:27:32:c8:da` *(lokal)* | **Deny**, Sumber pembangkit trafik *reconnaissance* (`nmap`), audit protokol IoT, dan pengujian autentikasi terkontrol. Bukti: `01-konfigurasi-ip-dan-routing-kali.png`. |
| **Monitoring Node**<br>*(Blue Team SIEM)*<br>VM: `SO-BiliCheck-Monitoring` | Security Onion Classic<br>(Ubuntu 16.04.6 LTS)<br>Hostname: `SecOnion` | `eth0`<br>*(Adapter 1)* | **Internal Network**<br>Nama: `bilicheck-lab` | **Tanpa IP (`UP`)**<br>*(Mode Manual/Sniffing)* | `08:00:27:43:8b:2b` | **Allow All**, Antarmuka penyadapan pasif (*promiscuous mode on, arp off, offload off*) yang terhubung ke sensor `seconion-import` (Snort NIDS & `tcpdump`). |
| **Monitoring Node**<br>*(Blue Team SIEM)* | Security Onion Classic<br>Hostname: `SecOnion` | `eth1`<br>*(Adapter 2)* | **Internal Network**<br>Nama: `bilicheck-lab` | **`192.168.11.200/24`**<br>*(Statis via interfaces)* | `08:00:27:a9:0d:c0` | **Deny**, Antarmuka manajemen Security Onion pada subnet laboratorium. Bukti: `01-konfigurasi-ip-dan-routing-seconion.png`. |
| **Monitoring Node**<br>*(Blue Team SIEM)* | Security Onion Classic<br>Hostname: `SecOnion` | `eth2`<br>*(Adapter 3)* | **NAT** | `10.0.2.15/24`<br>*(DHCP)* | `08:00:27:13:d4:5b` | **Deny**, Jalur *default route* (`via 10.0.2.2 dev eth2`) untuk pemeliharaan kontainer Elastic Stack dan sinkronisasi sistem. |

## 4. Konfigurasi Tabel Routing Aktual per Node

Berdasarkan hasil verifikasi pasca-*reboot* pada masing-masing mesin virtual, konfigurasi tabel routing IPv4 telah dipastikan bebas dari konflik *default gateway* ganda:

1. **Target Node (`target-bilicheck`, `192.168.11.5`)**:
   * `default via 10.0.2.2 dev enp0s3 proto dhcp src 10.0.2.15 metric 100` (Akses keluar melalui NAT).
   * `192.168.11.0/24 dev enp0s8 proto kernel scope link src 192.168.11.5` (Trafik segmen lab diarahkan sepenuhnya melalui `enp0s8` berdasarkan `/etc/netplan/99-bilicheck-lab.yaml`).
2. **Attacker Node (`attacker-kali`, `192.168.11.100`)**:
   * `192.168.11.0/24 dev eth0 proto kernel scope link src 192.168.11.100 metric 100` (Terisolasi penuh pada jaringan internal dengan parameter `ipv4.never-default yes`).
3. **Monitoring Node (`SecOnion`, `192.168.11.200`)**:
   * `default via 10.0.2.2 dev eth2 proto static metric 100` (Rute keluar melalui NAT `eth2`).
   * `192.168.11.0/24 dev eth1 proto kernel scope link src 192.168.11.200` (Rute manajemen lab melalui `eth1`, sementara `eth0` beroperasi murni pada Layer 2 untuk menyadap paket).

## 5. Matriks Alur Komunikasi & Visibilitas Pemantauan SOC

| Node Sumber | Node Tujuan | Protokol & Port | Jenis Trafik Simulasi | Mekanisme Visibilitas di Security Onion |
| :--- | :--- | :--- | :--- | :--- |
| **Kali Linux**<br>(`192.168.11.100`) | **Ubuntu Target**<br>(`192.168.11.5`) | ICMP, TCP (`22, 80, 1883, 3306, 5000, 8080, 9099`) | Trafik uji konektivitas (*ping*), pemindaian port (`nmap`), dan skenario pengujian Red Team sesuai `attack_plan.md`. | Disadap secara pasif oleh antarmuka **`eth0`** (`Promiscuous Mode: Allow All`) pada vSwitch `bilicheck-lab` dan dianalisis oleh engine **Snort (`snort-1`)** menuju **Sguil**. |
| **Host Windows**<br>(`127.0.0.1:2222`) | **Ubuntu Target**<br>(`10.0.2.15:22`) | TCP Port `22` *(OpenSSH Key-Based)* | Administrasi sistem oleh Blue Team melalui *Port Forwarding* VirtualBox (`SSH-2222`). | Dicatat secara lokal pada `/var/log/auth.log`, `systemd-journald` persisten, dan diawasi oleh daemon `fail2ban` serta `auditd`. |
| **Mock ESP32 / Client**<br>(`192.168.11.x`) | **Ubuntu Target**<br>(`192.168.11.5`) | TCP Port `1883` *(MQTT)* & `80` *(HTTP)* | Trafik *baseline* telemetri nilai ADC/Lux bilirubin *dummy* serta pengujian otorisasi topik MQTT pada Fase 2. | Direkam melalui *packet capture* (`pcap_agent`) pada `eth0` Security Onion dan log lokal `/var/log/mosquitto/mosquitto.log`. |

## 6. Inventaris Layanan dan Port pada Target Node (`192.168.11.5`)

Tabel ini membedakan layanan yang sudah terlihat aktif pada bukti Minggu 10 dari layanan yang masih berupa rencana. Kondisi dapat berubah setelah sesi pengujian, sehingga pemeriksaan terminal terbaru tetap menjadi acuan utama.

| Layanan / Daemon | Protokol & Port | Alamat Bind yang Terlihat | Status berdasarkan bukti yang tersedia | Keterangan dan bukti |
| :--- | :--- | :--- | :--- | :--- |
| **OpenSSH Server (`sshd`)** | TCP `22` | `0.0.0.0:22` atau alamat listen yang tercatat pada baseline | **Aktif** | Metode SSH yang terlihat adalah `publickey`. Percobaan password ditolak. Jail Fail2Ban aktif, tetapi bukti Minggu 10 menunjukkan `Currently banned: 0` dan `Total banned: 0`. |
| **BiliCheck Lab API** | TCP `5000` | `192.168.11.5:5000` | **Aktif** | Endpoint health memberikan HTTP `200`. Uji telemetri memperlihatkan `401` tanpa token, `202` untuk ADC `2048`, dan `422` untuk ADC `4096`. |
| **Mosquitto MQTT Broker** | TCP `1883` | `192.168.11.5:1883` | **Aktif** | Konfigurasi menunjukkan `allow_anonymous false` dan `password_file`. Pengujian klien menunjukkan penolakan anonim. Perintah publish menggunakan akun lab terlihat, tetapi log server final untuk autentikasi akun lab masih perlu dilengkapi. |
| **MySQL Server** | TCP `3306` | `127.0.0.1:3306` | **Aktif, lokal** | Screenshot menunjukkan MySQL hanya listen pada localhost. Akun `bilicheck_app` memiliki hak `INSERT` pada tabel telemetri sesuai bukti `SHOW GRANTS`. |
| **Nginx Web Server** | TCP `80` | Belum ada bind aktual yang didukung bukti Minggu 10 | **Belum dibuktikan aktif** | Jangan menyatakan Nginx aktif hanya berdasarkan rancangan arsitektur. |
| **Firebase Local Emulator** | TCP `8080` dan `9099` | Belum ada bind aktual yang didukung bukti Minggu 10 | **Rencana / belum diverifikasi** | Emulator tidak boleh ditulis aktif sebelum ada bukti proses dan port listen. |

**Catatan:** hasil scan Nmap, output `ss`, status service, dan bukti aplikasi memiliki fungsi berbeda. Gunakan bukti yang sesuai untuk setiap klaim. MySQL yang listen pada localhost tidak sama dengan port MySQL yang terbuka untuk Kali.

## 7. Ringkasan Bukti Verifikasi Jaringan & Konektivitas (Minggu ke-9)

Seluruh rancangan pada dokumen ini telah diverifikasi secara langsung di laboratorium dengan berkas bukti aktual berikut:

1. **Verifikasi Konfigurasi Adapter & IP Target Node (`docs/blueteam/evidence/hardening/`)**: Tersimpan pada `02-sebelum-konfigurasi-jaringan.png`, `07-virtualbox-adapter1-nat-dan-port-forwarding.png`, `08-virtualbox-adapter2-jaringan-internal-lab.png`, dan `09-sesudah-update-paket-os-dan-jaringan.png`.
2. **Verifikasi IP Statis & Konektivitas dari Attacker Node (`docs/redteam/evidence/`)**: Tersimpan pada `01-konfigurasi-ip-dan-routing-kali.png` dan `02-uji-ping-dan-cek-arp-ke-ubuntu.png` (menunjukkan `4 packets transmitted, 4 received, 0% packet loss` serta resolusi ARP `192.168.11.5 dev eth0 lladdr 08:00:27:47:9f:01 REACHABLE`).
3. **Verifikasi Penyadapan Paket & Deteksi IDS pada Monitoring Node (`docs/blueteam/evidence/monitoring-seconion/`)**: Tersimpan pada `01-konfigurasi-ip-dan-routing-seconion.png`, `04-tangkap-paket-icmp-lengkap-tcpdump.png`, `05-ringkasan-tangkap-paket-icmp.png` (`10 packets captured, 0 packets dropped`), dan `06-deteksi-alert-scan-dan-ping-di-sguil.png` (deteksi *real-time* trafik ping dan scan dari `.100` ke `.5` pada aplikasi Sguil).
