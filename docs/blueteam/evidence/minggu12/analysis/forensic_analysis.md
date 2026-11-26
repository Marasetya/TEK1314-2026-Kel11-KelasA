# Forensic Analysis Minggu 12

## 1. Tujuan analisis

Analisis memeriksa bukti dari Ubuntu Target, Sguil Security Onion, dokumentasi pengujian Minggu 11, dan PCAP tambahan 26 November 2026. Fokusnya adalah mengidentifikasi aktivitas jaringan, menilai autentikasi SSH, memeriksa status Fail2Ban, serta membatasi kesimpulan sesuai bukti yang benar-benar tersedia.

## 2. Sistem dan sumber bukti

| Komponen | Identitas | Fungsi |
|---|---|---|
| Ubuntu Target | `target-bilicheck`, `192.168.11.5` | SSH, API, MQTT, sistem target |
| Kali Linux | `attacker-kali`, `192.168.11.100` | Sumber pengujian Minggu 11 dan pengujian TCP tambahan |
| Security Onion | `SecOnion`, `192.168.11.200` | Monitoring, Snort, Sguil, PCAP jika tersedia |

File utama yang berhasil dikumpulkan dari Ubuntu pada output yang dibagikan meliputi `system_journal.txt`, `ssh_events.txt`, `fail2ban_events.txt`, `mqtt_events.txt`, `apache_events.txt`, `_var_log_auth.log`, `_var_log_syslog`, dan `_var_log_mosquitto_mosquitto.log`. Ukuran `apache_events.txt` yang terlihat adalah 17 byte. Isi file tersebut perlu dibaca sebelum menyatakan apakah log aplikasi tersedia atau kosong.

## 3. Analisis SSH

Log SSH pada 25 November 2026 memperlihatkan koneksi dari `192.168.11.100` dan percobaan dengan nama pengguna `soc_test` yang tidak valid. Pada 03:22:48.864887 UTC terdapat `Connection closed` dari Kali, masih di dalam jendela scan Nmap 03:22:42-03:24:15 UTC. Pada 03:45:35 dan 03:45:57 terdapat beberapa `Invalid user soc_test` dan penutupan koneksi pada tahap pre-authentication. Entri `Invalid user soc_test` kembali terlihat pada 07:46:37 UTC.

Tidak ada baris `Accepted` dari `192.168.11.100` pada potongan log yang dibagikan. Entri `Accepted publickey for socadmin` yang terlihat berasal dari `10.0.2.2`, sehingga tidak boleh dikaitkan dengan Kali. Baris `Connection closed` tidak dengan sendirinya membuktikan kegagalan password atau keberhasilan login.

## 4. Analisis Fail2Ban

Snapshot akuisisi Minggu 12 menunjukkan satu jail, yaitu `sshd`, dengan `Currently failed: 0`, `Total failed: 0`, `Currently banned: 0`, dan `Total banned: 0`. Daftar banned IP kosong.

Dokumentasi Blue Team Minggu 11 menyimpan snapshot sebelumnya yang menunjukkan `Currently failed: 1`, `Total failed: 1`, dan jumlah banned tetap 0. Kedua snapshot adalah kondisi pada waktu pemeriksaan yang berbeda. Log SSH Minggu 12 berisi entri `Invalid user`, tetapi statistik jail pada saat akuisisi tidak menghitung kegagalan tersebut. Bukti belum cukup untuk memastikan penyebabnya. Periksa konfigurasi filter, backend journal, nama unit ssh/sshd, dan waktu aktif jail sebelum menyimpulkan efektivitas Fail2Ban.

Kesimpulan yang didukung adalah service memiliki jail `sshd` dan tidak ada IP yang diblokir pada snapshot Minggu 12. Efektivitas Fail2Ban dalam mendeteksi seluruh percobaan SSH belum terbukti.

## 5. Analisis Security Onion dan Sguil

Sguil 0.9.0 terhubung ke `localhost` dengan user `analyst`. Tabel RealTime Events yang diperlihatkan berisi event historis dari `192.168.11.100` ke `192.168.11.5`, termasuk alert `ET SCAN Potential SSH Scan` pada 22 Oktober 2026 pukul 07:35:07 UTC.

Alert ID `5.1183` memperlihatkan source port 42127, destination port 22, TCP flag SYN, sequence number 3185100199, acknowledgement number 0, IP total length 44, TTL 52, dan payload `None`. Rule yang tampak menyebut `ET SCAN Potential SSH Scan OUTBOUND` dan threshold 5 kejadian dalam 120 detik. Data ini menunjukkan paket SYN yang memulai komunikasi TCP. Data tersebut tidak membuktikan autentikasi SSH atau akses akun.

Sguil juga memperlihatkan event historis tanggal 22 Oktober dan 4 November. Bukti yang diberikan belum memperlihatkan event yang cocok dengan scan Nmap 25 November 03:22:42-03:24:15 UTC. Karena itu, korelasi deteksi Sguil pada tanggal 25 November berstatus belum terverifikasi.

## 6. Analisis PCAP tambahan tanggal 26 November

PCAP `M12_SUPPLEMENTAL_20261126T024424Z.pcap` direkam pada interface `enp0s8` Ubuntu Target dengan filter host `192.168.11.100` dan TCP port 22. Output tcpdump menyebut 9 paket ditangkap, 9 paket diterima oleh filter, dan 0 paket dropped oleh kernel.

Urutan paket yang terlihat:

1. Kali mengirim SYN dari port sumber 60046 ke Ubuntu port 22.
2. Ubuntu membalas SYN-ACK.
3. Kali mengirim ACK.
4. Terlihat paket ACK tambahan dan FIN-ACK dari Kali.
5. Ubuntu mengirim ACK.
6. Ubuntu mengirim banner `SSH-2.0-OpenSSH_9.6p1 Ubuntu-3ubuntu13.20`.
7. Kali mengirim RST.

Output Kali `nc -vz -w 3 192.168.11.5 22` menyatakan port 22 `open`. Dengan demikian, capture mendukung bahwa TCP port SSH dapat dijangkau dari Kali pada pengujian tambahan 26 November. Capture tidak menunjukkan proses autentikasi atau login berhasil. PCAP ini bukan rekaman historis event Sguil 22 Oktober atau scan Nmap 25 November.

Hash PCAP yang diberikan dari Ubuntu:

```text
3cf22256fbf085acce4d5a23482d5c5965b9e96ca6df0dc01f7a5bd1cf0c9fbd
```

Skrip PowerShell akan membandingkan nilai tersebut dengan hash dari file yang disalin ke Windows. Status verifikasi hanya dinyatakan cocok jika nilai hasil hitung ulang identik.

## 7. Analisis API dan MQTT

Dokumentasi Red Team Minggu 11 mencatat request API tanpa token menghasilkan HTTP 401 pada 03:33:35 UTC, token valid dengan `rawAdc=2048` menghasilkan HTTP 202 pada 03:35:04 UTC, dan `rawAdc=4096` menghasilkan HTTP 422 pada 03:35:23 UTC. Ini merupakan bukti sisi klien. Bukti server-side independen belum ditunjukkan dalam materi yang diberikan. File `apache_events.txt` yang terkumpul berukuran 17 byte dan belum cukup untuk menyatakan log API lengkap.

Dokumentasi Red Team juga menunjukkan koneksi MQTT anonim ditolak dengan `CONNACK (5)`, sedangkan akun lab mendapat `CONNACK (0)`, `SUBACK`, dan subscriber menerima payload dummy `BC-TEST-001` pada topic pengujian. Itu membuktikan hasil publish-subscribe dari sisi klien. Log Mosquitto server telah dikumpulkan, tetapi isi rinci perlu diperiksa sebelum menyatakan korelasi server-side. Tidak ada bukti bahwa payload MQTT diteruskan ke API atau disimpan ke MySQL.

## 8. Kesimpulan forensik

Bukti mendukung adanya aktivitas reconnaissance dan probing autentikasi terhadap Ubuntu Target, serta keterjangkauan TCP port SSH. Bukti yang diberikan tidak membuktikan login tidak sah yang berhasil, kompromi host, eksfiltrasi data, perubahan data yang tidak sah, atau gangguan layanan. Area yang memerlukan pemeriksaan lanjutan adalah korelasi alert Security Onion pada 25 November, perbedaan statistik Fail2Ban dengan log SSH, dan ketersediaan log aplikasi serta PCAP historis.
