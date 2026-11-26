# Purple Team Validation Minggu 13 - SOC BiliCheck

## 1. Tujuan

Dokumen ini menghubungkan aktivitas Red Team, bukti deteksi Blue Team, analisis, respons, dan verifikasi ulang. Status setiap tahap mengikuti bukti yang tersedia. Ketiadaan screenshot atau alert dicatat sebagai belum terverifikasi, bukan dianggap berhasil.

## 2. Lima tahap Purple Team Loop

1. Serang atau uji.
2. Deteksi.
3. Analisis.
4. Respons atau mitigasi.
5. Verifikasi ulang.

## 3. S1 Reconnaissance

| Tahap | Hasil M13 |
|---|---|
| 1. Serang | Kali menjalankan Nmap 7.99 pada target `192.168.11.5`, port 22, 80, 1883, 3306, 5000, 8080, 9099. Waktu UTC sekitar 11:57:45 sampai 11:59:20 pada 26 November 2026. |
| 2. Deteksi | Security Onion `eth0` menangkap PCAP baru. Pemeriksaan file menunjukkan 343 frame IPv4 TCP antara Kali dan Ubuntu. Screenshot dan hash PCAP tersedia. |
| 3. Analisis | Sensor menerima trafik scan. Port 22, 1883, dan 5000 open; port 80, 3306, 8080, 9099 filtered. PCAP tidak sama dengan bukti alert. |
| 4. Respons | Firewall dan binding service yang ada dipertahankan pada sesi final posture. Tidak ada perubahan rule khusus S1 yang terbukti dilakukan. |
| 5. Verifikasi | Packet visibility terverifikasi. Alert Sguil yang cocok dengan timestamp scan belum tersedia karena screenshot atau export event M13-08 tidak ada di arsip. |

**Status S1:** Sebagian. Red Team dan packet capture terbukti. Korelasi alert Sguil belum terverifikasi.

## 4. S2A Autentikasi dan validasi API

| Tahap | Hasil M13 |
|---|---|
| 1. Serang/uji | Empat request dari Kali: tanpa token, token dummy yang tidak berlaku, token baru dengan ADC 2048, dan token baru dengan ADC 4096. |
| 2. Deteksi | Log service API pada Ubuntu mencatat respons 401, 401, 202, 422. |
| 3. Analisis | API menolak request tanpa kredensial dan token dummy, menerima ADC 2048, dan menolak ADC 4096. |
| 4. Respons | Token pada EnvironmentFile dirotasi. File diperiksa memiliki owner `root:root` dan mode `600`; service API aktif kembali. |
| 5. Verifikasi | Respons HTTP cocok antara Kali dan log service. Namun, token lama aktual tidak dipakai pada pengujian berlabel old token, dan token baru ikut terekspos dalam catatan sesi. Rotasi final perlu diulang. |

**Status S2A:** Sebagian. Fungsi autentikasi dan validasi input terbukti pada pengujian. Pengelolaan token final belum ditutup.

## 5. S2B Autentikasi MQTT dan publish-subscribe

| Tahap | Hasil M13 |
|---|---|
| 1. Serang/uji | Kali mencoba koneksi anonim, lalu menguji akun `esp32_bilicheck` untuk subscribe dan publish pada topic dummy. |
| 2. Deteksi | Output klien dan log broker Mosquitto menunjukkan CONNACK 5 untuk anonim, CONNACK 0 untuk user lab, SUBSCRIBE, PUBLISH, dan delivery. |
| 3. Analisis | Broker menolak sesi anonim dan menerima sesi terautentikasi. Subscriber menerima payload `BC-TEST-001` dengan `rawAdc=2048`. |
| 4. Respons | Konfigurasi `allow_anonymous false` dan `password_file` dipastikan ada. Kredensial lab perlu ditingkatkan sebelum penggunaan non-lab. |
| 5. Verifikasi | Anonim tetap ditolak dan authenticated pub-sub berhasil. |

**Status S2B:** Terbukti untuk autentikasi broker dan pengiriman pesan pada topic uji. Integrasi ke API atau MySQL tidak termasuk klaim.

## 6. S3 SSH dan Fail2Ban

| Tahap | Hasil M13 |
|---|---|
| 1. Serang/uji | Lima percobaan SSH menggunakan key tidak terdaftar, kemudian satu percobaan username tidak valid dari Kali. |
| 2. Deteksi | Journal SSH mencatat koneksi pre-authentication ditutup dan satu `Invalid user invalid_m13_test`. Fail2Ban menunjukkan failed 1 pada retest. |
| 3. Analisis | Key tidak terdaftar ditolak oleh SSH. Event username tidak valid dihitung pada jail setelah konfigurasi diperbarui. Status `Total banned` tetap 0. |
| 4. Respons | `journalmatch` pada jail diubah untuk mencakup `_SYSTEMD_UNIT=ssh.service + _COMM=sshd`. Konfigurasi lolos uji sintaks dan Fail2Ban direstart. |
| 5. Verifikasi | Satu event invalid-user dihitung. Tidak ada bukti bahwa Kali masuk daftar banned. Snapshot final memperlihatkan failed 0 dan banned 0. |

**Status S3:** Sebagian. Penolakan autentikasi dan penghitungan satu event terbukti. Pemblokiran IP tidak terbukti.

## 7. Ringkasan lintas skenario

| Skenario | Serang | Deteksi | Analisis | Respons | Verifikasi | Status akhir |
|---|---|---|---|---|---|---|
| S1 Nmap | Terbukti | PCAP terbukti, Sguil belum terverifikasi | Terbukti | Rule baseline dipertahankan | Packet capture tersedia | Sebagian |
| S2A API | Terbukti | Log service tersedia | Terbukti | Rotasi dilakukan, perlu rotasi ulang | 401, 401, 202, 422 | Sebagian |
| S2B MQTT | Terbukti | Log klien dan broker tersedia | Terbukti | Anonymous off dan password file digunakan | CONNACK 5 dan CONNACK 0 | Terbukti untuk pub-sub |
| S3 SSH | Terbukti | Invalid user dihitung | Terbukti terbatas | Journalmatch diperbarui | Failed 1, banned 0 | Sebagian |

## 8. Tindak lanjut

1. Buka Sguil dan ambil bukti real-time untuk jendela scan M13. Catat jika alert tidak ditemukan.
2. Jika perlu membuktikan ban Fail2Ban, rancang retest terkontrol menggunakan pola log yang dikenali filter dan amati `Banned IP list`. Jangan mengandalkan kegagalan key yang hanya menghasilkan `Connection closed [preauth]`.
3. Rotasi ulang token API yang terekspos dan simpan rahasia hanya pada file environment lokal.
4. Ganti password MQTT lab dengan rahasia kuat dan hindari penggunaan kredensial default atau sederhana.
5. Evaluasi rule SSH Anywhere dengan memperhatikan mekanisme port forwarding VirtualBox.
