# Final Security Posture Minggu 13 - SOC BiliCheck

**Snapshot final yang dirujuk:** 26 November 2026, sekitar 12:09:44 UTC  
**Host:** `target-bilicheck`, Ubuntu Server 24.04.5 LTS  
**IP lab:** `192.168.11.5`

## 1. Ringkasan

Ubuntu Target mempertahankan UFW aktif dengan kebijakan incoming deny. API dan MQTT listen pada alamat jaringan internal lab, sedangkan MySQL listen pada localhost. SSH listen pada seluruh interface dan aturan UFW mengizinkan port 22 dari Anywhere pada IPv4 dan IPv6. Fail2Ban aktif, tetapi snapshot final memperlihatkan tidak ada failed event tersisa dan tidak ada IP yang diblokir.

Security Onion memiliki bukti sensor aktif pada saat pemeriksaan dan PCAP S1 yang berisi trafik dari Kali ke Ubuntu. Alert Sguil yang cocok dengan scan tersebut belum terverifikasi.

## 2. Status komponen

| Komponen | Kondisi akhir yang terlihat | Penilaian |
|---|---|---|
| Ubuntu OS | Ubuntu 24.04.5 LTS | Teridentifikasi |
| Waktu Ubuntu | NTP aktif dan clock synchronized pada baseline | Terbukti pada snapshot baseline |
| UFW | Active; default incoming deny, outgoing allow | Aktif |
| SSH | `0.0.0.0:22` dan `[::]:22`; rule UFW Anywhere untuk IPv4 dan IPv6 | Berjalan, cakupan rule perlu ditinjau |
| Fail2Ban | Jail `sshd` tersedia; journalmatch menggunakan `ssh.service`; final failed 0, banned 0 | Berjalan, ban tidak terbukti |
| API | `192.168.11.5:5000` | Listener aktif |
| MQTT | `192.168.11.5:1883` | Listener aktif; konfigurasi anonymous false dan password file tersedia |
| MySQL | `127.0.0.1:3306` dan `127.0.0.1:33060` | Tidak diekspos pada interface lab menurut snapshot socket |
| Security Onion | eth0 UP; pcap_agent, snort_agent-1, snort-1, barnyard2-1 `[ OK ]` pada screenshot | Status sensor terbukti pada saat snapshot |
| PCAP S1 | 343 frame, 42.654 byte, komunikasi dua arah Kali dan Ubuntu | Packet visibility terbukti |
| Sguil S1 | Screenshot/export M13 tidak tersedia dalam paket yang diperiksa | Alert belum terverifikasi |

## 3. Aturan firewall yang terlihat

| No. | Port / alamat | Sumber | Aksi | Catatan |
|---:|---|---|---|---|
| 1 | 22/tcp | Anywhere | ALLOW IN | Komentar rule menyebut SSH via VirtualBox port forwarding |
| 2 | 5000/tcp pada 192.168.11.5 | 192.168.11.0/24 | ALLOW IN | API lab |
| 3 | 1883/tcp | 192.168.11.100 | ALLOW IN | MQTT dari Kali |
| 4 | 22/tcp IPv6 | Anywhere IPv6 | ALLOW IN | Aturan SSH IPv6 terlihat |

Rekomendasi: tinjau pembatasan SSH berdasarkan interface dan sumber yang benar-benar diperlukan. Jangan menghapus rule SSH sebelum jalur administrasi NAT diuji agar tidak kehilangan akses ke Target.

## 4. Status risiko tersisa

| ID | Risiko | Status | Tindakan berikutnya |
|---|---|---|---|
| R-01 | Alert Sguil untuk scan M13 belum dibuktikan | Terbuka | Periksa Sguil berdasarkan timestamp UTC, source, destination, dan port. |
| R-02 | Fail2Ban belum membuktikan tindakan ban IP | Terbuka | Lakukan retest invalid-user terkontrol atau catat keterbatasan filter dan action. |
| R-03 | Token API baru terekspos pada catatan sesi | Terbuka | Rotasi ulang dan jaga token agar tidak tercetak atau masuk Git. |
| R-04 | Token lama aktual belum diuji langsung setelah rotasi | Terbuka | Lakukan pengujian lama vs baru dengan prosedur aman apabila token lama tersedia. |
| R-05 | Password MQTT lab sederhana | Terbuka untuk non-lab | Ganti dengan kredensial kuat dan gunakan TLS pada lingkungan yang sesuai. |
| R-06 | SSH diizinkan dari Anywhere | Perlu review | Terapkan prinsip least privilege tanpa mengganggu NAT management. |
| R-07 | Bukti log API tidak membuktikan penyimpanan independen ke MySQL | Batas bukti | Jika klaim penyimpanan diperlukan, lakukan query read-only yang aman pada data dummy. |

## 5. Kesimpulan posture

Kontrol API, MQTT, SSH key-only, UFW, dan sensor monitoring telah diuji sebagian dengan bukti baru. API dan MQTT menunjukkan respons sesuai perilaku yang diharapkan untuk empat kondisi API dan dua kondisi MQTT. SSH menolak key tidak terdaftar dan Fail2Ban menghitung satu event username tidak valid pada retest. Namun, tindakan ban IP belum dibuktikan. Security Onion menangkap paket scan melalui interface sniffing, tetapi alert Sguil untuk scan yang sama belum terverifikasi.

Dengan demikian, posture akhir tidak dapat dinyatakan bebas risiko. Sistem replika menunjukkan sejumlah kontrol keamanan bekerja, tetapi beberapa hasil akhir masih perlu dikonfirmasi atau dicatat sebagai keterbatasan. Penilaian ini hanya berlaku untuk lab, bukan sistem medis produksi.
