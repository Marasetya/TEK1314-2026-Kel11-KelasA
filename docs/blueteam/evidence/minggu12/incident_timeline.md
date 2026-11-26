# Incident Timeline Minggu 12

Semua waktu pada tabel ini menggunakan UTC, kecuali bagian yang secara eksplisit diberi label WIB. Event historis Sguil dipisahkan dari pengujian 25 November dan capture tambahan 26 November.

| Waktu UTC | Aktivitas | Sumber bukti | Interpretasi dan batasan |
|---|---|---|---|
| 2026-10-22 07:35:07 | Sguil menampilkan alert `ET SCAN Potential SSH Scan`, Alert ID `5.1182` dan `5.1183`, dari `192.168.11.100:42127` ke `192.168.11.5:22`. | Transkripsi Sguil dan screenshot 04/05. | Event historis. Bukan bukti deteksi scan 25 November. |
| 2026-10-22 07:37:55-07:37:57 | Sguil menampilkan signature scan pada port 3306, 1521, 1433, 5432, 5802, dan 5815. | Transkripsi Sguil; dokumentasi Blue Team M11. | Ada perbedaan catatan untuk salah satu port: transkripsi tampilan terbaru menyebut 5802, sedangkan dokumentasi M11 menyebut 5902. Perlu cek gambar asli sebelum menetapkan satu nilai. |
| 2026-10-22 07:46:11 | Sguil menampilkan event `GPL ICMP_INFO PING *NIX`, Alert ID `5.1201`. | Transkripsi tampilan Sguil terbaru. | Dokumentasi M11 sebelumnya mencatat 07:48:11. Perbedaan timestamp perlu diperiksa dari screenshot sumber. |
| 2026-11-04 14:22:56 | Sguil menampilkan signature terpotong `ET SCAN MS Terminal Serv...` pada port 5000. | Transkripsi Sguil dan dokumentasi Blue Team M11. | Simpan teks sesuai tampilan. Jangan menyimpulkan layanan sebenarnya hanya dari nama signature yang terpotong. |
| 2026-11-04 14:23:41 | Sguil menampilkan signature terpotong `ET POLICY Outbound MSS...` pada port 5000. | Transkripsi Sguil dan dokumentasi Blue Team M11. | Event historis. Tidak otomatis berkaitan dengan pengujian 25 November. |
| 2026-11-25 02:51:58.934793 | SSH menerima public key untuk akun `socadmin` dari `10.0.2.2`. | `ssh_events.txt` dan output grep. | Sumbernya bukan IP Kali `192.168.11.100`. |
| 2026-11-25 03:22:42-03:24:15 | Jendela waktu pemindaian Nmap Minggu 11. | Dokumentasi pengujian Minggu 11. | Rentang pengujian yang digunakan untuk korelasi. |
| 2026-11-25 03:22:48.864887 | SSH mencatat `Connection closed` dari `192.168.11.100`, source port 54078. | `ssh_events.txt`. | Berada dalam rentang scan Nmap. Korelasi waktu dan IP ada, tetapi jenis proses yang menghasilkan koneksi belum dipastikan hanya dari baris ini. |
| 2026-11-25 03:33:35 | Pengujian API tanpa token menghasilkan HTTP 401. | Dokumentasi Red Team M11, bukti sisi klien. | Bukan log server independen. |
| 2026-11-25 03:35:04 | Pengujian API dengan token valid dan `rawAdc=2048` menghasilkan HTTP 202. | Dokumentasi Red Team M11, bukti sisi klien. | Respons aplikasi pada sesi uji. |
| 2026-11-25 03:35:23 | Pengujian API dengan `rawAdc=4096` menghasilkan HTTP 422. | Dokumentasi Red Team M11, bukti sisi klien. | Respons validasi pada sesi uji. |
| 2026-11-25 03:45:35.071965 | SSH mencatat `Invalid user soc_test` dari `192.168.11.100`, port sumber 56324, kemudian koneksi ditutup pada tahap pre-authentication. | `ssh_events.txt`. | Percobaan menggunakan nama akun tidak valid. Tidak ada bukti login berhasil dari baris ini. |
| 2026-11-25 03:45:57.108279-03:45:57.493428 | Tiga entri `Invalid user soc_test` dari `192.168.11.100`, source port 35860, 35876, dan 35884. | `ssh_events.txt`. | Percobaan berulang menggunakan akun tidak valid. |
| 2026-11-25 04:02:39.737176 | SSH mencatat koneksi ditutup dari `192.168.11.100`, source port 36722. | `ssh_events.txt`. | Baris ini tidak menyatakan autentikasi berhasil. |
| 2026-11-25 04:03:29.636451 | SSH mencatat koneksi ditutup dari `192.168.11.100`, source port 54742. | `ssh_events.txt`. | Baris ini tidak menyatakan autentikasi berhasil. |
| 2026-11-25 05:56:41.076087 | SSH menerima public key untuk `socadmin` dari `10.0.2.2`. | `ssh_events.txt`. | Sumber bukan IP Kali. |
| 2026-11-25 07:45:04.534563 | SSH menerima public key untuk `socadmin` dari `10.0.2.2`. | `ssh_events.txt`. | Sumber bukan IP Kali. |
| 2026-11-25 07:46:37.199837 | SSH mencatat `Invalid user soc_test` dari `192.168.11.100`, source port 38750, kemudian koneksi ditutup pada tahap pre-authentication. | `ssh_events.txt`. | Percobaan menggunakan akun tidak valid. |
| 2026-11-26 01:40:32 | Waktu akuisisi awal Ubuntu yang terlihat pada output terminal. | Screenshot 01 dan metadata akuisisi. | Waktu akuisisi log, bukan waktu semua kejadian dalam log. |
| 2026-11-26 02:44:24 | Nama file PCAP tambahan menunjukkan waktu pembuatan nama file saat perintah capture dimulai. | Nama file PCAP. | `M12_SUPPLEMENTAL_20261126T024424Z.pcap`. Bukan capture historis. |
| 2026-11-26 02:45:11 | PCAP merekam TCP SYN dari Kali ke port 22, SYN-ACK dari Ubuntu, ACK, penutupan sesi, banner SSH server, lalu RST dari Kali. | PCAP tambahan dan output tcpdump. | Waktu output tcpdump adalah 09:45:11 WIB. Sesi membuktikan keterjangkauan TCP SSH, bukan login. |

## Catatan waktu dan korelasi

1. Event Sguil 22 Oktober dan 4 November adalah historis. Bukti yang dikirim belum memperlihatkan alert Sguil yang cocok dengan rentang scan 25 November.
2. Koneksi SSH pukul 03:22:48 UTC dari `192.168.11.100` berada dalam rentang scan Nmap, tetapi satu baris `Connection closed` tidak membuktikan bahwa koneksi itu sendiri merupakan paket scan Nmap tertentu.
3. Percobaan `soc_test` pukul 03:45 dan 07:46 terjadi di luar rentang Nmap 03:22:42-03:24:15 UTC.
4. Capture 26 November adalah capture tambahan dari pengujian `nc`, bukan PCAP untuk event Sguil 22 Oktober atau scan 25 November.
