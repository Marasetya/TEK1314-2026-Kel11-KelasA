# Retest Report Minggu 13 - SOC BiliCheck

## 1. Ringkasan hasil

Retest Minggu 13 menghasilkan bukti baru untuk SSH, API, MQTT, dan reconnaissance. Hasil API dan MQTT sesuai dengan perilaku keamanan yang diharapkan pada lingkup uji. Sensor Security Onion terbukti menerima paket Nmap melalui interface sniffing dan menyimpan PCAP. Namun, bukti alert Sguil untuk scan tersebut belum tersedia. Fail2Ban menghitung satu event username tidak valid, tetapi tindakan pemblokiran IP tidak terbukti.

## 2. Matriks hasil retest

| ID | Pengujian | Input / aktivitas | Hasil aktual | Status |
|---|---|---|---|---|
| S1 | Nmap reconnaissance | Scan port 22, 80, 1883, 3306, 5000, 8080, 9099 dari Kali ke Ubuntu | 22, 1883, 5000 open; port lainnya filtered. PCAP sensor menangkap 343 frame. | Paket tertangkap. Alert Sguil belum terverifikasi. |
| S2A-1 | API tanpa token | POST payload ADC 2048 tanpa Authorization | HTTP 401 | Terbukti |
| S2A-2 | API token tidak berlaku | POST menggunakan string dummy `revoked-old-token-m11` | HTTP 401 | Token acak ditolak. Tidak membuktikan token lama aktual diuji. |
| S2A-3 | API token baru, nilai valid | Token baru, `rawAdc=2048` | HTTP 202 | Terbukti pada sesi uji |
| S2A-4 | API token baru, nilai invalid | Token baru, `rawAdc=4096` | HTTP 422 | Terbukti pada sesi uji |
| S2B-1 | MQTT anonim | CONNECT tanpa username/password | CONNACK 5, not authorised | Terbukti |
| S2B-2 | MQTT authenticated | Akun lab melakukan publish dan subscriber menerima payload | CONNACK 0, SUBACK, payload diterima | Terbukti pada topic uji |
| S3-1 | SSH key tidak terdaftar | Lima percobaan dengan key dummy | `Permission denied (publickey)` dan koneksi preauth ditutup | Penolakan SSH terbukti |
| S3-2 | Username SSH tidak valid | Satu percobaan `invalid_m13_test` | Journal mencatat `Invalid user`; Fail2Ban menunjukkan failed 1 | Deteksi filter terhitung |
| S3-3 | Fail2Ban ban action | Periksa daftar IP banned | Currently banned 0, Total banned 0 | Ban tidak terbukti |

## 3. Hasil terperinci

### 3.1 S1 Nmap dan packet capture

Kali menjalankan Nmap 7.99 terhadap `192.168.11.5`. Hasil service discovery dan port state tercatat pada `redteam/m13-s1-nmap-retest.txt`.

PCAP direkam langsung pada interface `eth0` Security Onion. Pemeriksaan file menghasilkan 343 frame IPv4 TCP dengan komunikasi dua arah antara `192.168.11.100` dan `192.168.11.5`. Waktu paket berada pada 11:57:47 sampai 11:59:21 UTC, beririsan dengan jendela scan Nmap 11:57:45 sampai 11:59:20 UTC.

SHA-256 PCAP:

```text
72d1b237c66706675d775a77861f43df290920e90576afbcb88d5b90283d2383
```

Kesimpulan terbatas: paket scan terlihat oleh interface sensor. Screenshot Sguil untuk event baru tidak tersedia pada arsip ini. Oleh sebab itu, status alert Snort/Sguil untuk retest S1 adalah belum terverifikasi.

### 3.2 S2A API

Output Kali menunjukkan HTTP 401 untuk request tanpa token, HTTP 401 untuk string token dummy yang tidak berlaku, HTTP 202 untuk token yang sedang aktif dengan ADC 2048, dan HTTP 422 untuk ADC 4096. Log `bilicheck-api.service` pada Ubuntu mencatat empat request dengan kode yang sama.

Keterbatasan: tes label token lama menggunakan string dummy, bukan token lama aktual. Token baru kemudian muncul pada catatan sesi. Untuk alasan keamanan, token tersebut harus dianggap terekspos dan dirotasi ulang. Pengujian ini belum dapat digunakan sebagai bukti bahwa token final yang aman sudah ditetapkan.

### 3.3 S2B MQTT

Broker menolak koneksi anonim dengan CONNACK 5. Akun `esp32_bilicheck` mendapat CONNACK 0, melakukan SUBSCRIBE, dan menerima payload pada topic uji. Log Mosquitto server mencatat penolakan anonim, koneksi akun lab, SUBSCRIBE, PUBLISH, dan delivery.

Kesimpulan dibatasi pada autentikasi broker dan pertukaran pesan. Integrasi payload dengan API atau MySQL tidak dibuktikan.

### 3.4 S3 SSH dan Fail2Ban

Konfigurasi `journalmatch` di `bilicheck.local` diperbarui agar mencakup unit `ssh.service` dan proses `sshd`. Validasi konfigurasi sukses. Lima percobaan key yang tidak terdaftar ditolak dan dicatat sebagai koneksi pre-authentication yang ditutup. Percobaan username tidak valid dicatat sebagai `Invalid user`.

Status Fail2Ban setelah pengujian username tidak valid menunjukkan satu failed event, tetapi tidak menunjukkan IP banned. Snapshot final berikutnya menunjukkan seluruh counter kembali nol. Dokumen tidak menyatakan bahwa IP Kali berhasil diblokir.

## 4. Perbandingan before-after

| Area | Before / temuan terdahulu | After / hasil M13 | Penilaian |
|---|---|---|---|
| Fail2Ban | Snapshot M12 nol failed dan nol banned, sementara raw SSH memiliki invalid user. | Setelah perubahan journalmatch, invalid-user M13 terlihat dan counter failed menjadi 1. Ban tetap 0. Snapshot akhir juga 0/0. | Penghitungan event terverifikasi pada satu uji; pemblokiran belum terbukti. |
| API | Token berpotensi terekspos dan retest pascarotasi belum tersedia. | Service aktif dengan EnvironmentFile root:root 600; respons 401, 401, 202, 422. | Validasi fungsi terbukti, rotasi final belum dapat ditutup karena token baru ikut terekspos dan token lama aktual tidak diuji. |
| MQTT | Anonymous ditolak pada uji sebelumnya; log server final terbatas. | Uji ulang anonymous ditolak, authenticated pub-sub berhasil, log broker tersedia. | Terbukti untuk autentikasi dan pub-sub. |
| Security Onion | Event Sguil historis tidak cocok dengan scan M11. | Sensor aktif pada saat pemeriksaan dan PCAP baru dari eth0 menangkap scan M13. | Packet visibility terbukti; alert Sguil belum terverifikasi. |
| Firewall | UFW aktif dengan default incoming deny dan rule layanan lab. | UFW tetap aktif; SSH tetap diizinkan Anywhere, API dari subnet lab, MQTT dari Kali. | Kebijakan dasar aktif; cakupan SSH perlu hardening lanjutan. |

## 5. Kesimpulan dan tindak lanjut

M13 menghasilkan perbaikan konfigurasi dan retest yang bermanfaat. Namun, proyek belum boleh menyatakan seluruh kontrol berhasil tanpa batasan. Sisa yang perlu ditutup adalah validasi alert Sguil S1, pengujian ban Fail2Ban yang benar-benar menunjukkan tindakan ban atau penjelasan keterbatasan, rotasi final token API, dan evaluasi password MQTT serta rule SSH.
