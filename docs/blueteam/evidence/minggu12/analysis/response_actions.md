# Response Actions Minggu 12

## 1. Tindakan yang telah dilakukan

| Tindakan | Status | Dasar |
|---|---|---|
| Mengumpulkan informasi identitas, waktu, interface, port, dan service Ubuntu Target | Dilakukan | File metadata hasil akuisisi. |
| Mengumpulkan journal sistem, SSH, Fail2Ban, Mosquitto, Apache, serta log file yang tersedia | Dilakukan | File pada direktori kasus Ubuntu. Kelengkapan setiap sumber tetap perlu diperiksa. |
| Mengambil status Fail2Ban dan jail `sshd` | Dilakukan | Snapshot M12 menunjukkan 0 failed dan 0 banned. |
| Memeriksa event historis Sguil dan header paket Alert ID `5.1183` | Dilakukan | Transkripsi dan screenshot Sguil. |
| Membuat PCAP tambahan untuk uji konektivitas SSH | Dilakukan | Capture `enp0s8` pada 26 November, 9 paket, 0 dropped. |
| Menghitung SHA-256 PCAP pada Ubuntu | Dilakukan | Nilai hash dicatat dari output terminal. |
| Menghitung ulang hash seluruh file di direktori proyek | Melalui skrip ini | Manifest Windows dibuat setelah file bukti disalin. |

## 2. Keputusan containment

Tidak dilakukan pemblokiran IP, perubahan firewall, penghentian service, penghapusan file, atau reset VM pada tahap akuisisi ini. Keputusan tersebut menjaga kondisi bukti agar tidak berubah sebelum pemeriksaan selesai. Tidak ada bukti kompromi berhasil yang mengharuskan klaim containment darurat dalam laporan ini.

## 3. Rekomendasi tindak lanjut

### 3.1 SSH dan Fail2Ban

1. Periksa backend Fail2Ban yang aktif dan kecocokan jail `sshd` dengan unit `ssh.service` atau `sshd.service` yang digunakan sistem.
2. Periksa filter Fail2Ban untuk pola `Invalid user` dan penutupan koneksi pre-authentication.
3. Jalankan uji terkendali pada jendela pemeliharaan dan catat log SSH, status jail, serta keputusan ban secara bersamaan.
4. Jangan menyatakan Fail2Ban efektif sebelum ada bukti percobaan cocok dengan filter dan tindakan ban yang tercatat.

### 3.2 Security Onion dan PCAP

1. Periksa retensi PCAP historis dan status sensor pcap tanpa mengubah bukti lama.
2. Verifikasi apakah event Sguil pada 25 November tersedia menggunakan filter IP dan jendela waktu yang tepat.
3. Simpan ekspor event dan PCAP dengan timestamp UTC serta checksum.
4. Bedakan alert historis dan capture tambahan yang dibuat setelah kejadian.

### 3.3 API dan MQTT

1. Temukan log service yang benar-benar menjalankan API pada port 5000. Jangan menganggap `apache_events.txt` mewakili log API jika API dijalankan oleh service lain.
2. Tinjau `mqtt_events.txt` dan `/var/log/mosquitto/mosquitto.log` untuk memeriksa penolakan anonim serta sesi terautentikasi.
3. Rotasi token API yang pernah tercatat utuh dalam catatan pengujian Minggu 11 sebelum pemakaian lanjutan, lalu dokumentasikan retest setelah rotasi.
4. Jangan menyimpulkan integrasi MQTT ke API atau MySQL tanpa bukti consumer end-to-end.

### 3.4 Pengelolaan bukti

1. Pertahankan salinan raw dan PCAP tanpa perubahan.
2. Simpan manifest SHA-256 dan catat setiap pemindahan file.
3. Sensor password, token, private key, dan informasi sensitif sebelum laporan dibagikan.
4. Lakukan remediation setelah akuisisi bukti selesai, kemudian dokumentasikan perubahan dan retest sebagai tahap terpisah.

## 4. Status respons

Status respons Minggu 12 adalah **akuisisi dan analisis awal selesai sebagian**. Bukti utama sudah dikumpulkan dan PCAP tambahan berhasil dibaca. Korelasi Sguil 25 November, PCAP historis, pemeriksaan penuh log MQTT/API, dan verifikasi efektivitas Fail2Ban masih memerlukan tindak lanjut.
