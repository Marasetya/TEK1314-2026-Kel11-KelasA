# Evidence Register Minggu 12

Register ini dibentuk dari file yang ditemukan secara fisik oleh skrip. File yang belum disalin tidak dicatat seolah-olah sudah tersedia.

| ID | Lokasi relatif | Ukuran (byte) | Deskripsi | Status |
|---|---|---:|---|---|
| E-001 | `screenshots/01-ubuntu-acquisition.png` | 303527 | Screenshot akuisisi bukti dan identitas Ubuntu Target. | Tersedia |
| E-002 | `screenshots/02-ssh-auth-log.png` | 337097 | Screenshot pemeriksaan log autentikasi SSH. | Tersedia |
| E-003 | `screenshots/03-fail2ban-status.png` | 276663 | Screenshot status Fail2Ban dan jail sshd. | Tersedia |
| E-004 | `screenshots/04-security-onion-sguil-events.png` | 292386 | Screenshot event historis Sguil. | Tersedia |
| E-005 | `screenshots/05-security-onion-packet-details.png` | 307489 | Screenshot detail rule dan header paket pada Sguil. | Tersedia |
| E-006 | `screenshots/06-pcap-analysis.png` | 376463 | Screenshot hasil analisis PCAP tambahan. | Tersedia |

## Bukti yang masih perlu diperiksa atau dilengkapi

- PCAP historis dari Security Onion untuk event 22 Oktober 2026 belum terbukti tersedia.
- Hasil pencarian Sguil khusus 25 November 2026 belum terkonfirmasi. Event historis tidak digunakan sebagai pengganti.
- Isi rinci `mqtt_events.txt` dan `apache_events.txt` perlu diperiksa sebelum membuat klaim server-side. Keberadaan file saja tidak membuktikan isi tertentu.
- Screenshot 07 dibuat setelah manifest final tampil. Jika belum ada, jangan membuat file tiruan.
