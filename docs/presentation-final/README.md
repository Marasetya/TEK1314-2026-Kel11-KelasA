# Slide Presentasi Akhir (Fase 3) — Kelompok 11

Direktori ini berisi salindia (*slide*) presentasi akhir untuk evaluasi **Praktikum Minggu ke-14 & 15 (Fase 3)** pada mata kuliah **Keamanan Siber (TEK1314)**, **Kelas A – Kelompok 11**.

---

## Informasi Berkas

| Parameter | Keterangan |
|---|---|
| **Nama Berkas** | `slides.pdf` |
| **Format** | PDF (sesuai ketentuan pengumpulan, bukan `.pptx`) |
| **Jumlah Slide** | 10–12 slide |
| **Judul Proyek** | *Hardening, Simulasi Serangan, dan Purple Team Loop pada Replika Infrastruktur Backend Sistem IoT Medis BiliCheck* |
| **Dosen Pengampu** | Shelvie Nidya Neyman |

---

## Ringkasan Alur Materi Presentasi

Salindia pada `slides.pdf` merangkum keseluruhan hasil kerja proyek dari **Fase 1 hingga Fase 3** secara ringkas dan berbasis bukti teknis:

1. **Identitas Kelompok & Gambaran Sistem BiliCheck:** Konteks sistem skrining optik hiperbilirubinemia berbasis ESP32-WROOM, MQTT, REST API, dan MySQL, serta prinsip simulasi terisolasi (*zero-risk simulation*) menggunakan data dummy `BC-TEST-001`.
2. **Deklarasi Tech Stack & Arsitektur Jaringan:** Pemetaan sistem asli ke replika laboratorium `bilicheck-lab` (`192.168.11.0/24`) yang terdiri atas `attacker-kali` (`.100`), `target-bilicheck` (`.5`), dan `SecOnion` (`eth0` promiscuous & `.200`).
3. **Preparation & Baseline Hardening (14/14 Kontrol):** Ringkasan pertahanan berlapis pada **Kategori A** (*Access & Authentication*), **Kategori B** (*Service & Attack Surface*), dan **Kategori C** (*Data, Logging, & System Integrity*).
4. **Simulasi Serangan & Deteksi (S1, S2A, S2B, S3):** Hasil pengujian Red Team menggunakan Nmap, cURL, Mosquitto CLI, dan OpenSSH beserta korelasi deteksi PCAP/Sguil pada Security Onion dan jurnal sistem.
5. **Analisis Akar Masalah & Remediasi Blue Team:** Penanganan rotasi `BILICHECK_API_TOKEN` (`0600`), penguatan autentikasi broker MQTT, dan penyelarasan `journalmatch` unit `ssh.service` pada Fail2Ban Ubuntu 24.04.5 LTS.
6. **Verifikasi Purple Team Loop & Rekomendasi:** Perbandingan status keamanan sebelum dan sesudah mitigasi, pembelajaran utama (*lessons learned*), serta rekomendasi pengamanan untuk sistem IoT medis BiliCheck versi produksi.

---

## Tautan Artefak Terkait

- **README Utama Repositori:** [`../../README.md`](../../README.md)
- **Incident Report Final (PDF):** [`../final/incident_report.pdf`](../final/incident_report.pdf)
- **Video Demo Siklus Purple Team Loop:** [`../final/video_demo.mp4`](../final/video_demo.mp4)
- **Log Iterasi Purple Team Loop:** [`../purple-team-log.md`](../purple-team-log.md)