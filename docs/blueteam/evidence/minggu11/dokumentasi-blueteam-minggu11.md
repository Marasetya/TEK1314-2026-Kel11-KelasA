# Dokumentasi Blue Team Minggu 11

**Mata kuliah:** TEK1314 - Keamanan Siber  
**Kelompok/Kelas:** Kelompok 11, Kelas A  
**Proyek:** SOC BiliCheck, replika backend IoT  
**Periode pemeriksaan:** 25 November 2026  
**Penanggung jawab Blue Team:** Muhammad Adam Ilyasa

## 1. Tujuan dan ruang lingkup

Dokumentasi ini mencatat pemeriksaan pertahanan dan monitoring yang dilakukan pada lingkungan laboratorium BiliCheck. Fokus Blue Team Minggu 11 adalah pemeriksaan event Security Onion melalui Sguil, audit autentikasi SSH, pemeriksaan status Fail2Ban, serta peninjauan bukti API dan MQTT yang disampaikan Red Team.

Seluruh pemeriksaan dibatasi pada replika di jaringan internal VirtualBox `bilicheck-lab` (`192.168.11.0/24`). Tidak ada pengujian terhadap VPS produksi, Firebase asli, perangkat sensor fisik, jaringan kampus, atau infrastruktur mitra. Payload yang dibahas merupakan data dummy.

## 2. Lingkungan pemeriksaan

| Komponen | Hostname | IP lab | Fungsi |
|---|---|---|---|
| Ubuntu Target | `target-bilicheck` | `192.168.11.5` | Backend API, MySQL, MQTT, SSH |
| Kali Linux | `attacker-kali` | `192.168.11.100` | Attacker Node dan sumber pengujian |
| Security Onion | `SecOnion` | `192.168.11.200` | Monitoring, PCAP, IDS, dan Sguil |

## 3. Pemeriksaan S1: Security Onion dan Sguil

### 3.1 Akses ke Monitoring Node

Konektivitas Kali ke Security Onion diperiksa menggunakan `ping` dengan hasil 4 paket terkirim, 4 paket diterima, dan 0% packet loss. Akses awal ke TCP port 443 mengalami timeout. Pemeriksaan pada Security Onion menunjukkan Apache mendengarkan pada port 443, sedangkan kebijakan INPUT firewall menggunakan `DROP`.

Akses Analyst kemudian ditambahkan melalui utilitas `so-allow` untuk alamat `192.168.11.100`. Pilihan akses yang tercatat adalah Analyst dengan port `22/tcp`, `443/tcp`, dan `7734/tcp`. Setelah aturan diterapkan, pemeriksaan dari Kali menunjukkan TCP port 443 pada `192.168.11.200` berstatus `open`.

Perubahan tersebut merupakan pengaturan akses administratif laboratorium ke Monitoring Node. Perubahan ini tidak boleh diartikan sebagai perubahan aturan deteksi serangan pada Target Node.

### 3.2 Event historis yang terlihat di Sguil

Sguil berhasil dijalankan dan terhubung ke `localhost`. Tampilan RealTime Events memperlihatkan event historis dari `192.168.11.100` menuju `192.168.11.5`.

| Timestamp pada Sguil (GMT) | Destination port | Signature yang terlihat |
|---|---:|---|
| 2026-10-22 07:35:07 | 22 | `ET SCAN Potential SSH Scan` |
| 2026-10-22 07:37:55 | 3306 | `ET SCAN Suspicious Inbound...` |
| 2026-10-22 07:37:56 | 1521 | `ET SCAN Suspicious Inbound...` |
| 2026-10-22 07:37:57 | 5902 | `ET SCAN Potential VNC Scan...` |
| 2026-10-22 07:37:57 | 1433 | `ET SCAN Suspicious Inbound...` |
| 2026-10-22 07:37:57 | 5815 | `ET SCAN Potential VNC Scan...` |
| 2026-10-22 07:37:57 | 5432 | `ET SCAN Suspicious Inbound...` |
| 2026-10-22 07:48:11 | ICMP | `GPL ICMP_INFO PING...` |
| 2026-11-04 14:22:56 | 5000 | `ET SCAN MS Terminal Serv...` |
| 2026-11-04 14:23:41 | 5000 | `ET POLICY Outbound MSS...` |

Nama signature dicatat sesuai teks yang terlihat pada tabel Sguil. Sebagian teks terpotong oleh lebar kolom.

### 3.3 Hasil korelasi terhadap scan 25 November

Dokumentasi Sguil yang tersedia memperlihatkan event bertanggal 22 Oktober dan 4 November 2026. Bukti tersebut menunjukkan bahwa Sguil memiliki event historis yang melibatkan pasangan IP Kali dan Ubuntu Target. Namun, screenshot yang tersedia tidak memperlihatkan event pada rentang scan Nmap 25 November 2026 pukul `03:22:42` sampai `03:24:15 UTC`.

Oleh karena itu, bukti historis tidak digunakan untuk menyatakan bahwa scan 25 November telah menghasilkan alert yang cocok. Status korelasi scan terbaru adalah **belum terverifikasi dari bukti yang tersimpan**. Jika pencarian lanjutan tidak menemukan event yang cocok, hasilnya dicatat sebagai keterbatasan deteksi atau blind spot pada sesi tersebut.

### 3.4 Bukti

- `docs/blueteam/evidence/minggu11/01-m11-s1-sguil-korelasi.png`
- Bukti PCAP historis pada `docs/blueteam/evidence/monitoring-seconion/`

## 4. Pemeriksaan S3: SSH dan Fail2Ban

### 4.1 Audit metode autentikasi

Audit dari Kali menggunakan perintah berikut:

```bash
nmap -n -Pn -p 22 --script ssh-auth-methods --script-args='ssh.user=soc_test' 192.168.11.5
```

Hasil yang terlihat:

```text
PORT   STATE SERVICE
22/tcp open  ssh

Host script results:
| ssh-auth-methods:
|   Supported authentication methods:
|_    publickey
```

Hasil tersebut menunjukkan bahwa SSH mengumumkan metode autentikasi `publickey`. Port 22 tetap terbuka untuk kebutuhan administrasi sesuai ruang lingkup lab.

### 4.2 Pemeriksaan Fail2Ban

Output pada screenshot Blue Team Minggu 11 menunjukkan:

```text
Status for the jail: sshd
|- Filter
|  |- Currently failed: 1
|  |- Total failed:     1
|  `- Journal matches:  _SYSTEMD_UNIT=sshd.service + _COMM=sshd
`- Actions
   |- Currently banned: 0
   |- Total banned:     0
   `- Banned IP list:
```

Nilai `maxretry=5` tercatat pada konfigurasi jail yang diperiksa. Statistik di atas merupakan keadaan pada saat screenshot diambil. Bukti yang tersimpan tidak membuktikan bahwa seluruh percobaan key gagal yang disebut pada catatan pengujian terjadi sebelum snapshot ini.

| Parameter | Hasil pada screenshot | Interpretasi |
|---|---:|---|
| Jail `sshd` | Aktif | Pemantauan SSH oleh Fail2Ban berjalan pada saat pemeriksaan |
| Currently failed | 1 | Satu kegagalan tercatat pada saat snapshot |
| Total failed | 1 | Penghitung kegagalan pada snapshot tersebut |
| Currently banned | 0 | Tidak ada IP yang sedang diblokir |
| Total banned | 0 | Tidak ada tindakan ban yang tercatat pada snapshot tersebut |

Dengan demikian, bukti mendukung bahwa jail `sshd` aktif dan kegagalan autentikasi tercatat. Bukti tidak mendukung klaim bahwa IP Kali telah diblokir oleh Fail2Ban.

### 4.3 Bukti

- `docs/blueteam/evidence/minggu11/04-m11-s3-ssh-fail2ban-final.png`
- Catatan pengujian Red Team pada `docs/redteam/evidence/minggu11/dokumentasi-redteam-minggu11.md`

## 5. Peninjauan bukti API dan MQTT dari Red Team

Bagian ini merupakan peninjauan bukti yang dihasilkan Red Team. Hasil pada sisi klien tidak disamakan dengan log server yang dikumpulkan secara mandiri oleh Blue Team.

### 5.1 API telemetry

Dokumentasi Red Team dan screenshot `02-m11-s2-api-401-202-422-final.png` menunjukkan tiga hasil:

| Pengujian | Hasil |
|---|---|
| Request tanpa token | HTTP `401 Unauthorized` |
| Token valid dan `rawAdc=2048` | HTTP `202 Accepted` |
| Token valid dan `rawAdc=4096` | HTTP `422 Unprocessable Entity` |

Hasil tersebut mendukung perilaku autentikasi dan validasi aplikasi pada sesi pengujian. Token yang pernah terekspos pada catatan terminal harus diperlakukan sebagai rahasia yang perlu dirotasi sebelum penggunaan lanjutan. Bukti yang ditinjau belum memastikan bahwa pengujian tersebut dilakukan menggunakan token hasil rotasi final.

### 5.2 MQTT authenticated publish-subscribe

Dokumentasi Red Team dan screenshot `03-m11-s2-mqtt-authenticated-pub-sub.png` menunjukkan `CONNACK (0)`, `SUBACK`, `Received PUBLISH`, topic `bilicheck/puskesmas01/dev-test/telemetry`, dan payload dummy `BC-TEST-001` dengan `rawAdc=2048` diterima subscriber.

Hasil tersebut mendukung keberhasilan komunikasi publish-subscribe terautentikasi pada sisi klien. Konfigurasi broker yang tercatat sebelumnya menunjukkan `allow_anonymous false` dan penggunaan `password_file`. Namun, folder bukti Blue Team Minggu 11 belum memuat log broker final yang berdiri sendiri untuk menguatkan setiap tahap autentikasi. Integrasi MQTT ke API atau MySQL tidak dinyatakan karena belum ada bukti consumer end-to-end.

## 6. Ringkasan hasil Blue Team Minggu 11

| Area | Hasil yang didukung bukti | Status |
|---|---|---|
| Akses Kali ke Security Onion | Ping berhasil dan akses Analyst ke TCP 443 dibuka melalui `so-allow` | Terbukti |
| Sguil | Event historis Kali ke Target terlihat | Terbukti untuk event historis |
| Korelasi scan 25 November | Event dengan waktu dan port yang cocok belum terlihat pada bukti screenshot | Belum terverifikasi |
| SSH | Metode autentikasi `publickey` terlihat | Terbukti |
| Fail2Ban | Jail aktif, satu kegagalan terlihat, jumlah banned nol pada snapshot | Terbukti sesuai snapshot |
| API | Hasil 401, 202, dan 422 tersedia dari bukti Red Team | Terbukti pada sisi klien |
| MQTT | Subscriber menerima payload setelah autentikasi menurut bukti Red Team | Terbukti pada sisi klien |
| Proses Snort | Bukti proses dan validasi akhir belum lengkap pada paket dokumentasi yang ditinjau | Belum terverifikasi |

## 7. Catatan pelaksanaan dan tindak lanjut

1. Pertahankan bukti historis Sguil dan beri label historis. Jangan mengubah tanggal event agar sesuai tanggal scan terbaru.
2. Korelasikan PCAP dan Sguil menggunakan timestamp UTC, source IP, destination IP, serta destination port yang sama.
3. Ambil status Fail2Ban setelah sesi uji yang ditentukan jika diperlukan untuk laporan lanjutan. Gunakan angka hasil aktual.
4. Pisahkan status Elastic, sensor NSM, proses Snort, dan hasil validasi konfigurasi.
5. Gunakan output Red Team untuk menjelaskan respons API dan penerimaan subscriber, serta tandai bahwa itu merupakan bukti sisi klien.
6. Jangan menampilkan token API, password MQTT, hash password, private key, atau isi file rahasia.

## 8. Kesimpulan

Pemeriksaan Blue Team Minggu 11 berhasil mengakses Security Onion sebagai Analyst, menjalankan Sguil, menemukan event historis dari pasangan IP Kali dan Ubuntu, serta memverifikasi bahwa SSH menawarkan metode `publickey` dan jail Fail2Ban `sshd` aktif. Pada snapshot Fail2Ban yang tersedia, tidak ada IP yang sedang atau pernah diblokir menurut penghitung jail.

Bukti Red Team melengkapi penilaian layanan aplikasi melalui hasil API `401`, `202`, dan `422`, serta keberhasilan subscriber MQTT menerima payload terautentikasi. Korelasi event Sguil dengan scan 25 November dan status proses Snort belum dapat dinyatakan terverifikasi dari bukti yang tersimpan. Kesimpulan ini mempertahankan pemisahan antara hasil yang terbukti, hasil historis, dan pemeriksaan yang masih memerlukan bukti tambahan.
