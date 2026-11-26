# Purple Team Log - SOC BiliCheck

**Mata kuliah:** TEK1314 - Keamanan Siber  
**Kelompok/Kelas:** Kelompok 11, Kelas A  
**Proyek:** BiliCheck, replika backend IoT  
**Pembaruan:** Konsolidasi bukti sampai 25 November 2026

## 1. Tujuan dan ruang lingkup

Dokumen ini mengintegrasikan hasil Red Team dan Blue Team untuk skenario S1, S2A, S2B, dan S3. Status setiap tahap didasarkan pada bukti yang tersedia. Dokumen tidak mengubah hasil yang belum terverifikasi menjadi keberhasilan dan tidak menyatakan siklus selesai apabila respons atau verifikasi belum didukung bukti.

Seluruh pengujian dilakukan terhadap Target replika `192.168.11.5` pada jaringan internal `bilicheck-lab` (`192.168.11.0/24`) menggunakan data sintetis. Sistem produksi, perangkat sensor fisik, Firebase asli, jaringan kampus, dan infrastruktur mitra berada di luar ruang lingkup.

## 2. Identitas node dan pembagian peran

| Komponen/anggota | Identitas | Tanggung jawab |
|---|---|---|
| Target Ubuntu | `target-bilicheck`, `192.168.11.5` | Backend replika |
| Red Team | Nayla Januarisa Prihadi dan Fauzan Nafis Jiyad Akbar | Menjalankan skenario, mencatat command, timestamp, hasil dan bukti |
| Blue Team | Muhammad Adam Ilyasa | Mengelola pertahanan Target dan Monitoring Node, memeriksa log, analisis dan mitigasi |
| Lead | Moh Rifky Marasetya | Koordinasi, integrasi laporan dan repository |
| Kali Linux | `attacker-kali`, `192.168.11.100` | Attacker Node |
| Security Onion | `SecOnion`, `192.168.11.200` | Monitoring Node |

## 3. Cara membaca status lima tahap

| Tahap | Kriteria bukti |
|---|---|
| 1. Serang | Aktivitas Red Team, command aktual, sumber, target, port, waktu, output |
| 2. Deteksi | PCAP, alert, atau log host/aplikasi yang benar-benar terlihat; jika tidak ditemukan, catat sumber yang telah diperiksa |
| 3. Analisis | Teknik, dampak yang dapat dibuktikan, root cause, kontrol yang bekerja, dan keterbatasan |
| 4. Respons/Mitigasi | Perubahan kontrol yang benar-benar dilakukan atau alasan kontrol baseline dipertahankan, termasuk bukti konfigurasi |
| 5. Verifikasi | Pengujian ulang dengan skenario dan parameter yang setara setelah respons |

Status `Sebagian` berarti satu atau lebih tahapan belum memiliki bukti memadai. Status itu bukan berarti kegiatan tidak dilakukan, melainkan bukti belum mendukung kesimpulan penuh.

## 4. S1 - Reconnaissance menggunakan Nmap

**Target:** `192.168.11.5`  
**Attacker:** `192.168.11.100`  
**Waktu scan:** 25 November 2026, `03:22:42` sampai `03:24:15 UTC`

### Tahap 1. Serang

Red Team menjalankan scan Nmap terbatas ke port `22,80,1883,3306,5000,8080,9099` pada Target replika. Hasil yang tercatat: port 22, 1883, dan 5000 berstatus open; port 80, 3306, 8080, dan 9099 berstatus filtered.

Bukti yang dirujuk:
- `docs/redteam/evidence/minggu10/03-s1-nmap-scan-terbatas.png`
- Catatan hasil scan 25 November pada `docs/blueteam/vulnerability_assessment_report.md`

### Tahap 2. Deteksi

PCAP dan screenshot Sguil historis tersedia. Sguil menunjukkan event dari pasangan IP Kali dan Ubuntu pada 22 Oktober dan 4 November. Screenshot yang ditinjau tidak memperlihatkan event yang cocok dengan waktu scan 25 November.

Bukti:
- `docs/blueteam/evidence/monitoring-seconion/07-m10-s1-nmap-packet-capture.png`
- `docs/blueteam/evidence/monitoring-seconion/08-m10-s1-sguil-event-dengan-korelasi-perlu-dikonfirmasi.png`
- `docs/blueteam/evidence/minggu11/01-m11-s1-sguil-korelasi.png`

**Status deteksi:** event historis terbukti, korelasi dengan scan 25 November belum terverifikasi.

### Tahap 3. Analisis

Port yang terbuka sesuai layanan yang digunakan dalam replika, yaitu SSH, MQTT, dan API. MySQL tidak dapat dijangkau dari Kali pada scan yang tercatat dan listener Target berada pada localhost. Hasil scan tidak dengan sendirinya membuktikan adanya eksploitasi atau kompromi host.

Keterbatasan utama pada sisi monitoring adalah belum adanya event Sguil yang dapat dicocokkan dengan timestamp, source, destination, dan port scan yang sama.

### Tahap 4. Respons/Mitigasi

UFW dan pembatasan listener layanan merupakan bagian dari hardening baseline yang telah terdokumentasi. Perubahan konfigurasi spesifik yang dilakukan sebagai respons langsung terhadap scan 25 November belum tercatat. Karena itu, log ini tidak menyatakan bahwa rule baru telah diterapkan untuk S1.

### Tahap 5. Verifikasi

Output retest pascarespons dengan parameter yang sama belum tersedia dalam paket dokumen yang ditinjau. Verifikasi S1 tetap perlu dilengkapi melalui retest terkontrol dan pemeriksaan PCAP/Sguil pada rentang waktu yang sama.

**Status keseluruhan S1: Sebagian.**

## 5. S2A - Autentikasi dan validasi API

**Endpoint:** `POST http://192.168.11.5:5000/api/telemetry`

### Tahap 1. Serang/uji

Red Team menguji request tanpa token, request dengan token valid dan `rawAdc=2048`, serta request dengan token valid dan `rawAdc=4096`.

Hasil aktual tanggal 25 November:

| Waktu UTC | Pengujian | Hasil |
|---|---|---|
| 03:33:35 | Tanpa token | HTTP 401 |
| 03:35:04 | Token valid, ADC 2048 | HTTP 202 |
| 03:35:23 | Token valid, ADC 4096 | HTTP 422 |

Bukti: `docs/redteam/evidence/minggu11/02-m11-s2-api-401-202-422-final.png`.

### Tahap 2. Deteksi

Respons aplikasi menunjukkan hasil autentikasi dan validasi. Bukti ini merupakan output sisi klien. Korelasi event Security Onion yang mengidentifikasi status logika API tidak ditunjukkan pada bukti yang ditinjau.

### Tahap 3. Analisis

API menolak request tanpa token, menerima nilai ADC dalam rentang yang diuji, dan menolak nilai ADC di luar rentang 0 sampai 4095. Respons `stored=true` merupakan isi respons API dan tidak dianggap sebagai pemeriksaan independen tabel MySQL.

Catatan sesi menyebut token pernah terekspos pada catatan terminal. Token harus diperlakukan sebagai rahasia yang perlu dirotasi sebelum penggunaan lanjutan.

### Tahap 4. Respons/Mitigasi

Bearer Token, pemisahan kredensial ke EnvironmentFile, serta permission `0600 root:root` tercatat sebagai kontrol pada baseline. Status rotasi token final belum dapat dipastikan dari bukti yang ditinjau. Perubahan setelah pengujian harus dicatat terpisah apabila benar-benar diterapkan.

### Tahap 5. Verifikasi

Hasil 401, 202, dan 422 tersedia pada satu sesi uji. Retest menggunakan token hasil rotasi final belum memiliki bukti yang dapat dipastikan. Tahap verifikasi pascarotasi tetap terbuka.

**Status keseluruhan S2A: Sebagian.**

## 6. S2B - Autentikasi MQTT dan publish-subscribe

**Broker:** `192.168.11.5:1883`  
**Akun lab:** `esp32_bilicheck`  
**Topic:** `bilicheck/puskesmas01/dev-test/telemetry`

### Tahap 1. Serang/uji

Percobaan tanpa kredensial ditolak broker dengan `CONNACK (5)` dan pesan `Connection Refused: not authorised`. Pengujian akun lab kemudian dilakukan menggunakan payload dummy `BC-TEST-001` dengan `rawAdc=2048`.

### Tahap 2. Deteksi

Output klien dan catatan pengujian memperlihatkan penolakan anonim, koneksi terautentikasi, subscription, dan penerimaan pesan. Bukti log broker pada folder Blue Team Minggu 11 belum dikemas sebagai output teks terpisah.

### Tahap 3. Analisis

Konfigurasi yang tercatat menggunakan `allow_anonymous false` dan `password_file`. Publisher dan subscriber terautentikasi berhasil terhubung. Subscriber menerima payload yang sama pada topic uji. Tidak ada bukti consumer yang meneruskan payload ke API atau menyimpan pesan ke MySQL.

### Tahap 4. Respons/Mitigasi

Konfigurasi autentikasi MQTT merupakan kontrol yang telah diterapkan dan ditunjukkan pada bukti konfigurasi broker. Catatan pengujian memuat penolakan koneksi anonim dan proses autentikasi akun lab. Tidak ada perubahan konfigurasi baru yang diklaim apabila tidak ada diff atau catatan implementasi yang mendukungnya.

### Tahap 5. Verifikasi

Subscriber menerima `CONNACK (0)`, `SUBACK`, `Received PUBLISH`, topic, dan payload `BC-TEST-001`. Percobaan anonim juga menghasilkan penolakan `CONNACK (5)`. Hasil ini membuktikan perilaku autentikasi dan delivery publish-subscribe pada sisi klien.

Bukti: `docs/redteam/evidence/minggu11/03-m11-s2-mqtt-authenticated-pub-sub.png`.

**Status keseluruhan S2B: Terbukti untuk autentikasi dan publish-subscribe pada sisi klien.** Integrasi ke API/MySQL tidak termasuk kesimpulan.

## 7. S3 - SSH Authentication Audit dan Fail2Ban

**Target:** `192.168.11.5:22`  
**Akun uji:** `soc_test`

### Tahap 1. Serang/uji

Red Team melakukan audit metode autentikasi menggunakan Nmap NSE. Hasil yang terlihat menunjukkan metode `publickey`. Catatan pengujian menyebut tiga percobaan dengan key uji yang tidak terdaftar ditolak dengan `Permission denied (publickey)`.

### Tahap 2. Deteksi

Screenshot Blue Team memperlihatkan jail Fail2Ban `sshd` aktif, `Currently failed: 1`, `Total failed: 1`, `Currently banned: 0`, dan `Total banned: 0`. Angka tersebut merupakan snapshot saat screenshot diambil.

### Tahap 3. Analisis

Kebijakan SSH key-only menolak metode autentikasi yang tidak diizinkan. Jail Fail2Ban aktif dan mencatat kegagalan. Namun, snapshot M11 menunjukkan tidak ada IP yang diblokir. Kegagalan autentikasi tidak boleh disamakan dengan keberhasilan tindakan ban.

### Tahap 4. Respons/Mitigasi

Kontrol SSH yang tercatat pada baseline mencakup `PermitRootLogin no`, `PubkeyAuthentication yes`, `PasswordAuthentication no`, `KbdInteractiveAuthentication no`, dan `AllowUsers socadmin`. Kontrol tersebut dipertahankan sebagai hardening baseline. Perubahan spesifik baru sebagai respons terhadap sesi M11 belum terdokumentasi.

### Tahap 5. Verifikasi

Audit metode SSH menunjukkan `publickey` dan percobaan dengan key yang tidak terdaftar ditolak. Fail2Ban pada snapshot tidak menunjukkan ban. Karena itu, verifikasi autentikasi SSH didukung bukti, sedangkan verifikasi pemblokiran IP tidak terjadi pada snapshot tersebut.

Bukti:
- `docs/blueteam/evidence/minggu11/04-m11-s3-ssh-fail2ban-final.png`
- `docs/redteam/evidence/minggu10/07-08-s3-ssh-auth-methods-dan-password-ditolak-gabungan.png`

**Status keseluruhan S3: Sebagian.**

## 8. Ringkasan status lima tahap

| Skenario | Serang | Deteksi | Analisis | Respons | Verifikasi | Status |
|---|---|---|---|---|---|---|
| S1 Nmap | Terbukti | Event historis ada, korelasi scan terbaru belum terverifikasi | Sebagian | Perubahan spesifik belum terdokumentasi | Retest pascarespons belum tersedia | Sebagian |
| S2A API | Terbukti | Respons aplikasi tersedia | Terbukti untuk tiga kondisi input | Kontrol baseline ada; rotasi token final belum dipastikan | Retest pascarotasi belum tersedia | Sebagian |
| S2B MQTT | Terbukti | Respons broker dan klien tersedia | Terbukti untuk autentikasi dan delivery klien | Kontrol autentikasi broker terdokumentasi | Penolakan anonim dan delivery terautentikasi terlihat | Terbukti untuk lingkup pub-sub klien |
| S3 SSH | Terbukti | Kegagalan tercatat pada snapshot | Terbukti untuk kebijakan publickey | Hardening baseline dipertahankan | Key tidak terdaftar ditolak; IP ban tidak terjadi pada snapshot | Sebagian |

## 9. Tindak lanjut terkoordinasi

1. S1: korelasikan PCAP dan Sguil menggunakan waktu UTC, source, destination, dan port. Jika event tidak ditemukan, catat hasil pemeriksaan sebagai blind spot. Simpan retest jika dilakukan.
2. S2A: pastikan token lama telah dirotasi secara aman, lalu ulangi tiga kondisi API. Simpan output tanpa nilai token.
3. S2B: pertahankan bukti subscriber dan tambahkan log broker jika tersedia. Jangan memperluas kesimpulan ke API atau MySQL tanpa bukti consumer.
4. S3: ambil status SSH dan Fail2Ban setelah sesi uji yang ditentukan. Gunakan angka aktual dan jangan memaksakan pemblokiran IP.
5. Monitoring: pisahkan status Elastic, sensor NSM, proses Snort, validasi konfigurasi, PCAP, dan alert Sguil.
6. Setelah setiap perubahan konfigurasi, simpan catatan before-after dan lakukan retest dengan parameter setara.

## 10. Kesimpulan

Purple Team Loop mengintegrasikan hasil pengujian Red Team dengan pemeriksaan, analisis, dan kontrol Blue Team. Bukti Minggu 11 mendukung pengujian API, autentikasi MQTT, penerimaan payload subscriber, audit metode SSH, dan keberadaan event historis Sguil. Tingkat pembuktian setiap tahap berbeda menurut skenario.

S2B telah terbukti untuk autentikasi dan publish-subscribe pada sisi klien. S1 masih memerlukan korelasi event untuk scan terbaru dan retest pascarespons. S2A memerlukan retest setelah rotasi token final. S3 membuktikan kebijakan `publickey`, tetapi tidak membuktikan IP Kali diblokir Fail2Ban. Status tersebut menjadi dasar tindak lanjut Minggu 12 dan Minggu 13 serta mencegah kesimpulan yang melampaui bukti.
