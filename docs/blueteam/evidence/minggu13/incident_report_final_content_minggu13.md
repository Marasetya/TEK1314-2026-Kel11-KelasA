# Bahan Pengisian Incident Report Final - Pembaruan Minggu 13

Dokumen ini berisi teks dan tabel yang dapat dipindahkan ke `Incident Report Final.docx`. Isi disusun dari bukti M13 dalam arsip `docs(3).zip`. Jangan menghapus catatan keterbatasan yang tercantum di sini.

## 1. Ringkasan Eksekutif

Kelompok 11 mengamankan replika backend BiliCheck pada jaringan internal VirtualBox, menjalankan pengujian terbatas untuk skenario reconnaissance, autentikasi dan validasi API, autentikasi MQTT, serta audit autentikasi SSH. Pada Minggu 13, tim melakukan remediation terpilih dan retest. Hasil retest menunjukkan API menolak request tanpa token dan token dummy yang tidak berlaku dengan HTTP 401, menerima payload ADC 2048 dengan HTTP 202, serta menolak ADC 4096 dengan HTTP 422. Broker MQTT menolak koneksi anonim dengan CONNACK 5 dan menerima publish-subscribe akun lab. SSH menolak key yang tidak terdaftar; satu event username tidak valid tercatat dan dihitung oleh Fail2Ban, tetapi IP Kali tidak terbukti diblokir. Security Onion menerima paket Nmap melalui interface sniffing dan menghasilkan PCAP yang dapat dianalisis. Korelasi alert Sguil untuk scan M13 belum terverifikasi karena bukti event M13 tidak tersedia pada arsip yang diperiksa. Tidak ditemukan bukti pada paket yang ditinjau mengenai login tidak sah yang berhasil, eksfiltrasi, atau kompromi host.

**Hasil akhir tiga area utama:** S2A API terbukti untuk perilaku respons yang diuji, dengan keterbatasan pengujian token lama aktual dan kebutuhan rotasi ulang. S2B MQTT terbukti untuk autentikasi dan publish-subscribe pada topic uji. S1 menunjukkan packet visibility, tetapi alert Sguil belum terverifikasi. S3 membuktikan penolakan autentikasi dan pencatatan satu invalid-user event, bukan pemblokiran IP.

## 2. Preparation dan hardening

Gunakan tabel 10 kontrol dari `blueteam/hardening_checklist.md` sebagai sumber daftar kontrol. Bukti M13 menambahkan kondisi akhir UFW, port, service, API, MQTT, dan Fail2Ban. Jangan mengisi kolom Before dan After dengan nomor screenshot yang tidak sesuai. Referensikan nama file bukti yang benar.

Temuan posture akhir yang dapat dimasukkan:

- UFW active dengan default incoming deny dan outgoing allow.
- API listen pada `192.168.11.5:5000`.
- MQTT listen pada `192.168.11.5:1883`.
- MySQL bind pada `127.0.0.1:3306`.
- SSH listen pada seluruh interface dan UFW mengizinkan port 22 dari Anywhere pada IPv4 dan IPv6.
- Fail2Ban jail `sshd` aktif, tetapi snapshot akhir menunjukkan banned 0.

## 3. Detection & Analysis

### 3.1 S1 Reconnaissance

**Hasil aktual Red Team:** Pada 26 November 2026, Kali menjalankan Nmap 7.99 dari `192.168.11.100` ke `192.168.11.5` pada port 22, 80, 1883, 3306, 5000, 8080, dan 9099. Hasil menunjukkan 22, 1883, dan 5000 open; port 80, 3306, 8080, dan 9099 filtered.

**Deteksi di Security Onion:** Interface sniffing `eth0` menangkap PCAP yang berisi 343 frame IPv4 TCP antara Kali dan Ubuntu pada rentang 11:57:47 sampai 11:59:21 UTC. Status sensor pcap_agent, snort_agent-1, snort-1, dan barnyard2-1 terlihat `[ OK ]` pada screenshot pemeriksaan. Bukti alert Sguil M13 tidak ditemukan pada arsip ini, sehingga korelasi alert tidak dapat dinyatakan berhasil.

**Klasifikasi:** Aktivitas reconnaissance dan service discovery pada jaringan lab.

**Analisis akar masalah:** Sejumlah service replika memang harus tersedia untuk pengujian, sehingga port layanan tertentu terlihat open. Port MySQL tetap bind ke localhost dan tidak terlihat sebagai open dari Kali. Keterlihatan paket oleh sensor tidak otomatis berarti rule IDS menghasilkan alert.

### 3.2 S2 MQTT dan API

**Hasil aktual Red Team:** API menghasilkan 401 untuk request tanpa token, 401 untuk string token dummy yang tidak berlaku, 202 untuk token baru dengan ADC 2048, dan 422 untuk ADC 4096. MQTT anonim ditolak dengan CONNACK 5; akun `esp32_bilicheck` berhasil melakukan publish-subscribe dan subscriber menerima payload dummy.

**Deteksi Blue Team:** Log API mencatat empat request dengan kode yang sama. Log broker Mosquitto mencatat penolakan anonim, koneksi akun lab, SUBSCRIBE, PUBLISH, dan pengiriman pesan.

**Klasifikasi:** Pengujian kontrol autentikasi dan validasi input pada API serta broker MQTT.

**Analisis akar masalah:** API mengharuskan Bearer Token dan memvalidasi rentang ADC 0 sampai 4095. Broker Mosquitto menggunakan `allow_anonymous false` dan password file. Pengujian token lama menggunakan string dummy, bukan token lama aktual. Token baru sempat terekspos dalam catatan sesi dan membutuhkan rotasi ulang. Kredensial MQTT merupakan kredensial lab dan perlu diperkuat sebelum penggunaan non-lab.

### 3.3 S3 SSH Authentication Abuse

**Hasil aktual Red Team:** Lima percobaan menggunakan key tidak terdaftar ditolak oleh SSH dengan `Permission denied (publickey)`. Satu percobaan username tidak valid menghasilkan event `Invalid user invalid_m13_test`.

**Deteksi Blue Team:** Journal SSH menunjukkan event invalid-user. Setelah konfigurasi journalmatch diperbarui, status jail menunjukkan `Currently failed: 1` dan `Total failed: 1`. Status menunjukkan `Currently banned: 0` dan `Total banned: 0`.

**Klasifikasi:** Percobaan autentikasi SSH yang tidak berhasil.

**Analisis akar masalah:** Kebijakan SSH publickey-only menolak key yang tidak terdaftar. Filter Fail2Ban menghitung event invalid-user pada pengujian yang dilakukan. Koneksi pre-authentication akibat key mismatch tidak meningkatkan counter pada output uji tersebut. Ban IP tidak terbukti.

## 4. Containment, Eradication & Recovery

| Skenario | Containment | Eradication | Recovery |
|---|---|---|---|
| Reconnaissance | Monitoring dan firewall baseline tetap membatasi layanan yang diizinkan. | Tidak ada perubahan service khusus S1 yang terbukti dilakukan. | PCAP sensor dibuat dan port state diperiksa kembali. |
| MQTT & API | Request tanpa autentikasi ditolak dan koneksi MQTT anonim ditolak. | Token API dirotasi; konfigurasi MQTT diverifikasi menggunakan `allow_anonymous false`. | Request API dan pub-sub MQTT diuji ulang. Rotasi token perlu diulang karena token baru terekspos. |
| SSH | SSH menolak key yang tidak terdaftar. | Journalmatch jail diperbarui agar mencakup unit SSH aktual. | Invalid-user event dihitung satu kali. Ban IP tidak terbukti dan harus dicatat sebagai batasan. |

## 5. Verifikasi

| Skenario | Pengujian yang diulang | Hasil setelah mitigasi |
|---|---|---|
| Reconnaissance | Nmap ke port layanan replika dan capture pada Security Onion eth0 | Paket scan masuk ke PCAP. Alert Sguil yang cocok belum terverifikasi. |
| MQTT & API | API tanpa token, token dummy, token baru dengan ADC valid dan invalid; MQTT anonymous dan authenticated pub-sub | API menghasilkan 401, 401, 202, 422. MQTT anonymous ditolak dan akun lab berhasil bertukar pesan. |
| SSH | Key SSH tidak terdaftar dan username tidak valid | Key ditolak. Invalid-user tercatat dan dihitung satu kali. Tidak ada IP yang terbukti diblokir. |

## 6. Lessons Learned

1. Sinkronisasi waktu UTC pada setiap VM diperlukan untuk menghubungkan output scan, PCAP, journal, dan alert.
2. Pesan log SSH berbeda menurut jenis kegagalan. Key yang tidak cocok menghasilkan koneksi pre-authentication yang ditutup, sedangkan username tidak valid menghasilkan pesan `Invalid user` yang dapat dihitung oleh filter.
3. Status service Snort yang aktif dan PCAP yang berisi paket merupakan dua bukti berbeda dari alert Sguil. Keduanya tidak boleh disamakan.
4. Token dan password tidak boleh ditulis pada screenshot, command history, atau catatan yang masuk repository.
5. Pengujian sisi klien perlu dilengkapi log sisi server dan, jika klaimnya menyangkut penyimpanan, pemeriksaan independen pada database.

## 7. Kontrol yang masih perlu diperkuat

- Korelasi event Snort/Sguil untuk scan terbaru belum terverifikasi.
- Efektivitas pemblokiran Fail2Ban belum dibuktikan, walaupun satu invalid-user event dihitung.
- Token API perlu dirotasi ulang dan token lama aktual belum diuji langsung pada retest.
- Password MQTT lab perlu diganti dengan kredensial kuat dan TLS dipertimbangkan untuk penerapan non-lab.
- Rule SSH UFW yang mengizinkan Anywhere perlu ditinjau dengan tetap mempertahankan akses administrasi VirtualBox.

## 8. Rekomendasi untuk proyek IoT BiliCheck asli

1. Gunakan TLS untuk MQTT dan validasi sertifikat broker.
2. Terapkan pengelolaan secret terpusat, rotasi token, dan larangan mencatat token ke log.
3. Terapkan autentikasi perangkat per identitas dan ACL topic MQTT.
4. Batasi akses SSH ke sumber administrasi yang disetujui dan gunakan monitoring autentikasi.
5. Pusatkan log API, broker, host, dan IDS dengan sinkronisasi waktu.
6. Verifikasi pipeline alert secara end-to-end sebelum menyatakan monitoring efektif.
7. Pertahankan prinsip least privilege untuk database dan pastikan data dummy digunakan pada pengujian keamanan.

## 9. Batasan kesimpulan

Semua hasil berlaku untuk replika lab. Tidak ada klaim bahwa perangkat sensor, backend produksi, Firebase asli, atau data pasien riil diuji. Tidak ada bukti pada paket yang diperiksa tentang login tidak sah yang berhasil atau kompromi host.
