# TEK1314-2026-Kel11-KelasA

# PBL Keamanan Siber-Kelompok 11

## Identitas

| Parameter | Keterangan |
|---|---|
| Mata Kuliah | TEK1314 - Keamanan Siber |
| Kelas | A |
| Kelompok | 11 |
| Pertemuan | 4 |
| Tahap | Design Phase |

## Skenario Proyek

Proyek PBL Kelompok 11 menggunakan skenario keamanan jaringan berbasis
Web Server. Lingkungan proyek dirancang dengan tiga node utama, yaitu
Attacker Node, Target Node, dan Monitoring Node.

Attacker Node direncanakan menggunakan Kali Linux dan berfungsi sebagai
lingkungan pengujian dari sisi Red Team. Target Node direncanakan
menggunakan Ubuntu Server CLI sebagai server korban yang akan menjadi target
pengujian. Monitoring Node menggunakan Security Onion dan berfungsi sebagai
sistem monitoring untuk mengamati trafik jaringan pada segmen yang digunakan
oleh kelompok.

Network segment yang digunakan oleh Kelompok 11 adalah:

`192.168.11.0/24`

Rancangan IP setiap node adalah:

| Node | IP Address | OS |
|---|---|---|
| Target Server | 192.168.11.5 | Ubuntu Server CLI |
| Attacker Node | 192.168.11.100 | Kali Linux |
| Monitoring Node | 192.168.11.200 | Security Onion |

## Network Design

Detail rancangan jaringan dapat dilihat pada:

- docs/design/topology.png
- docs/design/ip_plan.md

## Target OS Selection

Target OS yang dipilih untuk tahap implementasi berikutnya adalah:

**Ubuntu Server CLI**

Alternatif yang dipertimbangkan adalah DVWA ISO dan Metasploitable.

## Status Pertemuan 4

- Network topology: Completed
- IP Address Plan: Completed
- Target OS Selection: Completed
- Target Server Installation: Not started
