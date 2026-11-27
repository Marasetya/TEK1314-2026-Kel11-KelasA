# TEK1314-2026-Kel11-KelasA

**Hardening, Simulasi Serangan, dan Purple Team Loop pada Replika Infrastruktur Backend Sistem IoT Medis BiliCheck**

Repositori ini berisi seluruh artefak perancangan, penguatan sistem (*hardening*), simulasi pengujian keamanan, investigasi forensik digital, serta laporan akhir proyek *Project-Based Learning* (PBL) **Fase 1 hingga Fase 3** pada mata kuliah **Keamanan Siber (TEK1314)**.

---

## Identitas Kelompok

| Parameter | Keterangan |
|---|---|
| **Mata Kuliah** | TEK1314 – Keamanan Siber |
| **Kelas / Kelompok** | Kelas A – Kelompok 11 |
| **Program Studi** | Teknologi Rekayasa Komputer, Sekolah Vokasi, IPB University |
| **Dosen Pengampu** | Shelvie Nidya Neyman |
| **Tahun Akademik** | 2026 |

### Anggota Tim & Pembagian Peran

| Nama Lengkap | NIM | Peran | Tanggung Jawab Utama |
|---|---|---|---|
| **Moh Rifky Marasetya** | `J0404231049` | **Lead** | Koordinasi arsitektur simulasi, penyelarasan *tech stack*, pengelolaan repositori GitHub, dan finalisasi laporan serta salindia. |
| **Nayla Januarisa Prihadi** | `J0404241149` | **Red Team** | Penyusunan *attack plan*, eksekusi pemindaian Nmap dan audit endpoint API/MQTT/SSH, pencatatan stempel waktu UTC, dan *retest* pascamitigasi. |
| **Fauzan Nafis Jiyad Akbar** | `J0404241152` | **Red Team** | Pengelolaan *Attacker Node* Kali Linux, eksekusi simulasi serangan terkontrol (S1–S3), dokumentasi keluaran pengujian, dan validasi ulang. |
| **Muhammad Adam Ilyasa** | `J0404241048` | **Blue Team** | *Baseline hardening* 14 kontrol pada *Target Node* Ubuntu, pengelolaan *Monitoring Node* Security Onion, investigasi DFIR (SHA-256), serta eksekusi *Containment, Eradication, and Recovery*. |

---

## Ringkasan Sistem & Prinsip Simulasi

Sistem yang dilindungi adalah replika infrastruktur *backend* dari **BiliCheck** (*Bilirubin Clinical Health Evaluator & Concentration Kit*), perangkat skrining optik berbasis Internet of Things (IoT) untuk deteksi dini risiko hiperbilirubinemia (*ikterus neonatorum*) pada bayi baru lahir yang dikembangkan oleh **Tim AGRION** bersama Mitra **Bapak Aep Setiawan, S.Si., M.Si.**

Seluruh pengujian dilaksanakan secara **terisolasi dan tanpa risiko (*zero-risk simulation*)** di dalam jaringan internal VirtualBox `bilicheck-lab` (`192.168.11.0/24`) menggunakan data sintetis/dummy (`BC-TEST-001` / `dev01`), tanpa menyentuh server produksi Domcloud/Firebase, basis data pasien asli, maupun perangkat keras fisik ESP32-WROOM milik Mitra.

---

## Arsitektur Jaringan & Rencana Alamat IP (`bilicheck-lab`)

- **Segmen Jaringan Simulasi:** `192.168.11.0/24` (*VirtualBox Internal Network* `bilicheck-lab`, tanpa *default gateway* eksternal)
- **Segmen Manajemen:** `10.0.2.0/24` (*NAT* terpisah untuk pembaruan paket OS dan administrasi *host*)

| Host | Peran | Alamat IP | Sistem Operasi & Layanan Utama |
|---|---|---|---|
| **`attacker-kali`** | Red Team (*Attacker Node*) | `192.168.11.100/24` (`eth0`) | Kali Linux 2026.2; Nmap 7.99, OpenSSH Client, cURL 8.20.0, `mosquitto_pub`, `mosquitto_sub`. |
| **`target-bilicheck`** | Replika Backend & Pertahanan (*Target Node*) | `192.168.11.5/24` (`enp0s8` – lab)<br>`10.0.2.15/24` (`enp0s3` – NAT) | Ubuntu Server 24.04.5 LTS; OpenSSH 9.6p1 (`:22`), Eclipse Mosquitto 2.0.18 (`:1883`), `BiliCheckLabAPI/1.0` (`:5000`), MySQL 8.0 (`127.0.0.1:3306`), UFW, Fail2Ban, `auditd`, AppArmor, AIDE. |
| **`SecOnion`** | Blue Team Monitoring / NSM (*Monitoring Node*) | Tanpa IP / Promiscuous UP (`eth0` – *sniffing*)<br>`192.168.11.200/24` (`eth1` – SIEM)<br>`10.0.2.15/24` (`eth2` – NAT) | Security Onion Classic 2.x; Snort NIDS, `barnyard2`, `pcap_agent` (`tcpdump`), Sguil 0.9.0, dan Elastic Stack. |

---

## Ringkasan Hasil Akhir (Fase 1 – Fase 3)

1. **Preparation & Baseline Hardening (14/14 Kontrol Tuntas):**
   - **Kategori A (Access & Authentication – 4/4):** Penonaktifan login `root` dan autentikasi kata sandi SSH (`PermitRootLogin no`, `PasswordAuthentication no`, `PubkeyAuthentication yes`), aktivasi Fail2Ban `jail sshd`, kebijakan kompleksitas kata sandi PAM (`minlen = 14`), dan wajib autentikasi broker MQTT (`allow_anonymous false`).
   - **Kategori B (Service & Attack Surface – 4/4):** Firewall UFW *default deny incoming* (hanya membuka port TCP `22`, `1883`, `5000`), pembaruan otomatis keamanan OS, penguatan parameter kernel `sysctl`, dan isolasi binding MySQL eksklusif pada `127.0.0.1:3306`.
   - **Kategori C (Data, Logging, & System Integrity – 6/6):** Pemantauan `auditd` (11 aturan), AppArmor (23 profil *enforce*), integritas berkas AIDE (`102.127` entri), sinkronisasi waktu NTP/UTC, proteksi kredensial API pada `EnvironmentFile` (`0600 root:root`), hak akses minimum MySQL (`INSERT` only), dan validasi rentang sensor ADC 12-bit (`0–4095`).

2. **Simulasi Serangan, Remediasi, & Verifikasi Purple Team Loop:**

| Skenario | Vektor Pengujian | Temuan & Mitigasi | Status Akhir |
|---|---|---|---|
| **S1** | *Network Reconnaissance & Service Discovery* (Nmap) | Hanya port `22`, `1883`, `5000` terbuka; port `80`, `3306`, `8080`, `9099` tersaring (`filtered`). Lalu lintas terekam penuh di 343 frame PCAP Security Onion (SHA-256 terverifikasi). | **Aman / Tertutup** |
| **S2A** | Penyalahgunaan Autentikasi & Input REST API (`POST /api/telemetry`) | Permintaan tanpa token dan token lama ditolak (`HTTP 401`); injeksi nilai di luar rentang fisik `rawAdc = 4096` ditolak (`HTTP 422`); token hasil rotasi diterima (`HTTP 202`). | **Aman / Tertutup** |
| **S2B** | Akses Anonim & *Spoofing* Topik Broker MQTT (`:1883`) | Koneksi klien anonim ditolak total (`CONNACK 5: not authorised`); hanya klien terautentikasi pada `password_file` yang dapat mengakses topik `bilicheck/puskesmas01/dev-test/telemetry`. | **Aman / Tertutup** |
| **S3** | Audit Autentikasi SSH & *Authentication Abuse* (`:22`) | Hanya metode `publickey` diizinkan; penyelarasan `journalmatch = _SYSTEMD_UNIT=ssh.service + _COMM=sshd` pada Fail2Ban berhasil memblokir otomatis IP penyerang (`192.168.11.100`). | **Aman / Tertutup** |

---

## Navigasi Artefak Proyek (`docs/`)

### Fase 1 — Arsitektur, Hardening, & Rencana Pengujian
- **Topologi Jaringan & IP Plan:** [`docs/design/topology.png`](docs/design/topology.png) | [`docs/design/ip_plan.md`](docs/design/ip_plan.md)
- **Deklarasi Tech Stack:** [`docs/presentation-minggu9/tech_stack_declaration.md`](docs/presentation-minggu9/tech_stack_declaration.md)
- **Daftar Periksa Hardening (14 Kontrol):** [`docs/blueteam/hardening_checklist.md`](docs/blueteam/hardening_checklist.md)
- **Rencana Serangan Red Team:** [`docs/redteam/attack_plan.md`](docs/redteam/attack_plan.md)

### Fase 2 — Deteksi, DFIR, & Purple Team Loop
- **Aturan Pemantauan & Korelasi Alert:** [`docs/blueteam/monitoring_alert_rules.md`](docs/blueteam/monitoring_alert_rules.md)
- **Log Iterasi Purple Team Loop:** [`docs/purple-team-log.md`](docs/purple-team-log.md)
- **Laporan Incident Response & Forensik (Minggu 12):** [`docs/blueteam/incident_response_report_minggu12.md`](docs/blueteam/incident_response_report_minggu12.md)
- **Dokumentasi Remediasi & Retest (Minggu 13):** [`docs/blueteam/evidence/minggu13/README_MINGGU13.md`](docs/blueteam/evidence/minggu13/README_MINGGU13.md)

### Fase 3 — Laporan Akhir, Presentasi, & Video Demo (Minggu 14–15)
- **Incident Report Final (PDF):** [`docs/final/incident_report.pdf`](docs/final/incident_report.pdf)
- **Slide Presentasi Akhir (PDF):** [`docs/presentation-final/slides.pdf`](docs/presentation-final/slides.pdf)
- **Video Demo Siklus Purple Team Loop (`.mp4`):** [`docs/video`](ddocs/video) *(Skenario serangan, deteksi Security Onion, mitigasi Blue Team, dan verifikasi serangan ulang)*
