# IP Plan - Kelompok 11 (TEK1314 Keamanan Siber)

## 1. Identitas Kelompok

| Parameter | Nilai |
|---|---|
| Mata Kuliah | TEK1314 - Keamanan Siber |
| Kelas | A |
| Kelompok | 11 |
| Pertemuan | 4 |
| Tahap | Design Phase |

|---|---|
| Moh Rifky Marasetya | J0404231049 |
| Muhammad Adam Ilyasa | J0404241048 | 
| Nayla Januarisa Prihadi | J0404241149 | 
| Fauzan Nafis Jiyad Akbar | J0404241152

## 2. Network Segment

| Parameter | Nilai |
|---|---|
| Network | 192.168.11.0/24 |
| Subnet Mask | 255.255.255.0 |
| Network Address | 192.168.11.0 |
| Host Range | 192.168.11.1 - 192.168.11.254 |
| Broadcast | 192.168.11.255 |

Segmen jaringan 192.168.11.0/24 digunakan sebagai network segment untuk seluruh node pada rancangan PBL Kelompok 11.

## 3. Host Addressing

| No. | Hostname | IP Address | OS Direncanakan | Fungsi |
|---|---|---|---|---|
| 1 | Target Server | 192.168.11.5 | Ubuntu Server CLI | Target/Korban |
| 2 | Attacker Node | 192.168.11.100 | Kali Linux | Red Team / Pengujian Keamanan |
| 3 | Monitoring Node | 192.168.11.200 | Security Onion | Monitoring Jaringan / Blue Team |

## 4. Role dan Fungsi Node

### 4.1 Target Server

- Hostname: Target Server
- IP Address: 192.168.11.5
- OS Direncanakan: Ubuntu Server CLI
- Peran: Target/Korban
- Skenario: Server Web

Target Server direncanakan sebagai node korban yang akan digunakan sebagai lingkungan target pada tahap implementasi proyek berikutnya.

### 4.2 Attacker Node

- Hostname: Attacker Node
- IP Address: 192.168.11.100
- OS Direncanakan: Kali Linux
- Peran: Red Team

Attacker Node digunakan sebagai lingkungan pengujian keamanan dari sisi Red Team terhadap Target Server.

### 4.3 Monitoring Node

- Hostname: Monitoring Node
- IP Address: 192.168.11.200
- OS Direncanakan: Security Onion
- Peran: Blue Team / Monitoring

Monitoring Node digunakan untuk melakukan pemantauan terhadap trafik jaringan pada segmen jaringan kelompok.

## 5. Network Communication Plan

| Source | Destination | Jenis Trafik | Keterangan |
|---|---|---|---|
| Attacker Node | Target Server | Network Traffic | Trafik pengujian dari Red Team |
| Target Server | Monitoring Node | Monitoring Traffic | Trafik yang dipantau oleh Security Onion |
| Attacker Node | Monitoring Node | Monitoring Traffic | Trafik pengujian yang menjadi objek monitoring |

Security Onion ditempatkan sebagai Monitoring Node untuk memantau trafik pada segmen jaringan yang digunakan oleh kelompok.

## 6. OS Selection

### Target OS yang Dipilih

**Ubuntu Server CLI**

Ubuntu Server CLI dipilih sebagai Target OS karena merupakan salah satu opsi yang diberikan pada panduan PBL Pertemuan 4 dan sesuai untuk perancangan Target Server pada skenario proyek.

### Alternatif yang Dipertimbangkan

| Alternatif | Status |
|---|---|
| Ubuntu Server CLI | **Dipilih** |
| DVWA ISO | Alternatif |
| Metasploitable | Alternatif |

**Keputusan akhir:** Target Node direncanakan menggunakan Ubuntu Server CLI.

> Catatan: Pada Pertemuan 4, pemilihan OS Target hanya sampai tahap penentuan. Instalasi Target Server belum dilakukan pada tahap ini.

## 7. Catatan Port dan Potensi Eksposur

Karena skenario yang dipilih adalah Web Server, port yang menjadi perhatian pada tahap perancangan adalah:

| Port | Protokol | Layanan | Keterangan |
|---:|---|---|---|
| 80 | TCP | HTTP | Layanan Web |
| 443 | TCP | HTTPS | Layanan Web dengan koneksi terenkripsi |

Port tersebut menjadi port yang akan diperhatikan dalam rancangan pengujian keamanan dan monitoring pada tahap implementasi berikutnya.

> Catatan: Daftar port merupakan hasil identifikasi awal Red Team pada tahap Design Phase. Pengujian dan eksploitasi belum dilakukan pada Pertemuan 4.

## 8. Ringkasan IP Plan

| Node | IP Address | OS | Role |
|---|---|---|---|
| Target Server | 192.168.11.5 | Ubuntu Server CLI | Target/Korban |
| Attacker Node | 192.168.11.100 | Kali Linux | Red Team |
| Monitoring Node | 192.168.11.200 | Security Onion | Blue Team / Monitoring |

## 9. Status Design

- [✓] Network segment ditentukan
- [✓] IP statis setiap node ditentukan
- [✓] Attacker Node ditentukan
- [✓] Target Node ditentukan
- [✓] Monitoring Node ditentukan
- [✓] Target OS ditentukan
- [✓] Port awal skenario Web diidentifikasi
- [✓] Penempatan Security Onion ditentukan
