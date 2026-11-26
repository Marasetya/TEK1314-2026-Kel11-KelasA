# Video Demo & Artefak Akhir (Fase 3) — Kelompok 11

Direktori ini menyimpan artefak akhir proyek *Project-Based Learning* (PBL) **Mata Kuliah Keamanan Siber (TEK1314) – Minggu ke-14 & 15 (Fase 3)** untuk **Kelas A – Kelompok 11**, dengan topik:

> **Hardening, Simulasi Serangan, dan Purple Team Loop pada Replika Infrastruktur Backend Sistem IoT Medis BiliCheck**

---

## Daftar Berkas pada Direktori Ini

| Nama Berkas | Deskripsi | Keterangan |
|---|---|---|
| `video_demo.mp4` | Rekaman demonstrasi 1 siklus penuh *Purple Team Loop* | Durasi **06:45** (memenuhi ketentuan 5–8 menit), format `.mp4`. |
| `incident_report.pdf` | Laporan Akhir Insiden (*Incident Report Final*) | Mengikuti struktur kerangka kerja **NIST SP 800-61r2**. |

> **Catatan Ukuran Berkas GitHub (>100 MB):** Apabila ukuran berkas `video_demo.mp4` melebihi batas unggah GitHub (100 MB), video dapat diakses melalui tautan berikut:
> - **Tautan Alternatif Video Demo:** `[Tempelkan tautan Google Drive / YouTube Unlisted di sini]`

---

## Spesifikasi & Lingkungan Demonstrasi

Seluruh demonstrasi pada video dijalankan di dalam lingkungan laboratorium virtual terisolasi **`bilicheck-lab` (`192.168.11.0/24`)** menggunakan data sintetis/dummy (`BC-TEST-001` / `dev01`) tanpa menyentuh server produksi maupun perangkat fisik Mitra, dengan identitas host dan alamat IP yang sesuai dengan `docs/design/ip_plan.md`:

| Nama Host | Peran | Alamat IP | Sistem Operasi |
|---|---|---|---|
| `attacker-kali` | Red Team (*Attacker Node*) | `192.168.11.100/24` | Kali Linux 2026.2 |
| `target-bilicheck` | Target & Pertahanan (*Target Node*) | `192.168.11.5/24` | Ubuntu Server 24.04.5 LTS |
| `SecOnion` | Blue Team Monitoring (*Monitoring Node*) | `eth0` Promiscuous (tanpa IP)<br>`192.168.11.200/24` (`eth1`) | Security Onion Classic 2.x |

---

## Penanda Waktu (*Timestamp*) Video Demo

Video demo menampilkan alur lengkap **4 tahap siklus *Purple Team Loop*** dengan rincian waktu sebagai berikut:

| Tahap | Isi Demonstrasi | Rentang Waktu |
|---|---|---|
| **Tahap 1** | Serangan dijalankan oleh Red Team dari `attacker-kali` (`192.168.11.100`) menuju `target-bilicheck` (`192.168.11.5`): **S1** (Nmap port scan), **S2A** (REST API auth & input `rawAdc`), **S2B** (Broker MQTT anonim & terautentikasi), dan **S3** (Audit autentikasi SSH). | `00:00 – 01:45` |
| **Tahap 2** | Deteksi dan analisis bukti oleh Blue Team pada **Security Onion** (PCAP 343 frame TCP & konsol Sguil), log layanan `bilicheck-api.service` & `mosquitto.log`, serta analisis kendala `journalmatch` pada Fail2Ban `jail sshd`. | `01:46 – 03:30` |
| **Tahap 3** | Penerapan kontrol mitigasi oleh Blue Team: rotasi `BILICHECK_API_TOKEN` pada `/etc/bilicheck-api/bilicheck-api.env` (izin `0600`), penguatan kredensial & topik Mosquitto, serta penambahan `journalmatch = _SYSTEMD_UNIT=ssh.service + _COMM=sshd` pada Fail2Ban. | `03:31 – 05:00` |
| **Tahap 4** | Pengulangan serangan yang sama (*retest*) pascamitigasi dan verifikasi kegagalan serangan: penolakan token API lama (`HTTP 401`), penolakan koneksi MQTT anonim (`CONNACK 5`), serta pemblokiran IP otomatis oleh Fail2Ban (`192.168.11.100`). | `05:01 – 06:45` |

---

## Tim Penyusun (Kelas A – Kelompok 11)

- **Moh Rifky Marasetya** (`J0404231049`) — Lead
- **Nayla Januarisa Prihadi** (`J0404241149`) — Red Team
- **Fauzan Nafis Jiyad Akbar** (`J0404241152`) — Red Team
- **Muhammad Adam Ilyasa** (`J0404241048`) — Blue Team