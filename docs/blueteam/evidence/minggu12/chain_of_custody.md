# Chain of Custody Minggu 12

## 1. Identitas kasus

| Parameter | Nilai |
|---|---|
| Case ID | SOC-BILICHECK-DFIR-M12-20261126 |
| Proyek | SOC BiliCheck, replika backend IoT |
| Host sumber utama | `target-bilicheck` |
| IP Ubuntu Target | `192.168.11.5` |
| IP Kali Linux | `192.168.11.100` |
| IP Security Onion | `192.168.11.200` |
| Akun pengumpul pada Ubuntu | `socadmin` |
| Direktori kasus Ubuntu | `/home/socadmin/DFIR_M12_20261126T014140Z` |

## 2. Catatan penguasaan dan pemindahan bukti

| Tahap | Kegiatan | Waktu / sumber | Catatan |
|---|---|---|---|
| COC-01 | Akuisisi log dan metadata Ubuntu | `2026-11-26 01:40:32 UTC` | Timestamp dari output akuisisi awal. Direktori sumber dibuat pada Ubuntu Target. |
| COC-02 | Pembuatan PCAP tambahan | Nama file menunjukkan `2026-11-26 02:44:24 UTC` | Capture menggunakan `enp0s8`, filter host `192.168.11.100` dan TCP port 22. |
| COC-03 | Paket pada PCAP tambahan | `2026-11-26 02:45:11 UTC` | Output tcpdump terlihat pada waktu lokal `09:45:11 WIB`, setara dengan `02:45:11 UTC`. Sembilan paket ditangkap dan nol paket dropped. |
| COC-04 | Pemindahan bukti ke direktori proyek Windows | Waktu pencatatan skrip: `2026-11-26T03:26:58Z` | Folder akuisisi Ubuntu belum disalin ke Windows pada saat skrip dijalankan. Lengkapi transfer dan jalankan ulang skrip dengan parameter -UbuntuEvidencePath. |
| COC-05 | Penghitungan manifest final | Setelah penyalinan selesai pada proses skrip yang sama | Hash dihitung ulang setelah file disalin ke direktori proyek. Manifest Ubuntu awal disimpan terpisah. |

## 3. Identitas file dan integritas

File bukti didaftarkan pada `evidence_register.md`. Nilai SHA-256 final tercatat pada `hashes/SHA256SUMS_M12.csv` dan `hashes/SHA256SUMS_M12.txt`.

Hash PCAP yang dihitung pada Ubuntu Target:

```text
3cf22256fbf085acce4d5a23482d5c5965b9e96ca6df0dc01f7a5bd1cf0c9fbd
```

Hash tersebut harus dibandingkan dengan hasil penghitungan ulang pada salinan Windows. Status cocok atau tidak cocok dicatat dalam `hashes/PCAP_SHA256_VERIFICATION.md`.

## 4. Prinsip penanganan

1. File sumber tidak diedit langsung. Skrip menyalin bukti ke direktori proyek dan membuat cadangan jika file tujuan dengan nama sama memiliki isi berbeda.
2. Bukti historis, bukti akuisisi Ubuntu, dan PCAP tambahan tanggal 26 November dibedakan.
3. Timestamp pada file, timestamp pada log, dan waktu pemindahan dicatat sebagai jenis waktu yang berbeda.
4. Password, private key, token API, dan rahasia MQTT tidak dimasukkan ke laporan.
5. Jika file tidak tersedia, statusnya dicatat sebagai belum tersedia. File kosong tidak dibuat untuk memenuhi struktur.

## 5. Batasan chain of custody

Catatan ini dibentuk dari output terminal, dokumen Minggu 11, dan file yang disalin ke direktori proyek. Chain of custody formal yang ditandatangani oleh dua pihak atau menggunakan media write-blocked tidak tersedia dalam bukti yang diberikan. Karena itu, dokumen ini merupakan catatan penanganan bukti laboratorium, bukan sertifikasi forensik independen.
