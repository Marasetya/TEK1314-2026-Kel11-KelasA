# Red Team Attack Plan dan Catatan Pelaksanaan - SOC BiliCheck

**Mata kuliah:** TEK1314 - Keamanan Siber  
**Kelompok/Kelas:** Kelompok 11, Kelas A  
**Proyek:** BiliCheck, replika backend IoT  
**Status dokumen:** Rencana dan catatan hasil aktual S1-S3, diperbarui berdasarkan bukti sampai 25 November 2026

## 1. Tujuan dokumen

Dokumen ini menetapkan ruang lingkup, skenario, tool, target, hasil pengujian, dan bukti Red Team. Rencana dibedakan dari hasil aktual. Perintah contoh tidak dianggap sebagai perintah yang dijalankan kecuali didukung catatan sesi atau output.

## 2. Identitas dan pembagian peran

| Anggota | Peran | Tanggung jawab |
|---|---|---|
| Moh Rifky Marasetya | Lead | Koordinasi arsitektur, repository, integrasi laporan, dan konsistensi artefak |
| Nayla Januarisa Prihadi | Red Team | Pengujian S1-S3 dari Kali bersama Fauzan, pencatatan command, waktu, hasil, dan bukti |
| Muhammad Adam Ilyasa | Blue Team | Satu-satunya anggota Blue Team, pengelolaan Target dan Monitoring Node, log, deteksi, serta mitigasi |
| Fauzan Nafis Jiyad Akbar | Red Team | Pengujian S1-S3 dari Kali bersama Nayla, Attack Plan, dan bukti hasil uji |

## 3. Lingkungan dan batas keselamatan

| Node | Hostname | IP lab | Fungsi |
|---|---|---|---|
| Ubuntu Target | `target-bilicheck` | `192.168.11.5` | Target replika backend |
| Kali Linux | `attacker-kali` | `192.168.11.100` | Attacker Node |
| Security Onion | `SecOnion` | `192.168.11.200` | Monitoring Node |

Seluruh pengujian hanya diarahkan ke `192.168.11.5` pada jaringan internal VirtualBox `bilicheck-lab` (`192.168.11.0/24`). Dilarang melakukan scanning atau pengujian terhadap VPS produksi, Firebase asli, perangkat sensor mitra, jaringan kampus, atau host di luar lab. Gunakan data dummy. Pengujian DoS/flood dan eksekusi malware sungguhan tidak dilakukan.

## 4. Kriteria pencatatan hasil

Setiap skenario dicatat dengan lima tahap Purple Team Loop:

1. Serang: command aktual, sumber, target, port, waktu, dan output.
2. Deteksi: PCAP, alert, atau log yang benar-benar terlihat.
3. Analisis: hasil, dampak yang dapat dibuktikan, kontrol yang bekerja, dan keterbatasan.
4. Respons: perubahan kontrol yang benar-benar diterapkan atau alasan mempertahankan kontrol yang sudah ada.
5. Verifikasi: pengujian ulang dengan skenario dan parameter yang setara setelah respons.

Ketiadaan alert dicatat sebagai hasil monitoring yang tidak ditemukan pada sumber yang diperiksa, bukan diubah menjadi klaim deteksi berhasil.

## 5. Skenario S1: Reconnaissance menggunakan Nmap

**Kategori:** reconnaissance dan service discovery  
**Target:** `192.168.11.5`  
**Tool:** Nmap 7.99

Perintah yang tercatat pada sesi 25 November 2026:

```bash
nmap -n -Pn -sT -sV -T3 --reason \
  -p 22,80,1883,3306,5000,8080,9099 \
  192.168.11.5 \
  -oN ~/bilicheck-evidence/minggu11/m11-s1-nmap-current.txt
```

Waktu scan yang tercatat adalah `2026-11-25 03:22:42 UTC` sampai `03:24:15 UTC`.

| Port | Hasil | Catatan |
|---|---|---|
| 22/tcp | open | OpenSSH |
| 80/tcp | filtered | Tidak memperoleh respons pada scan |
| 1883/tcp | open | MQTT |
| 3306/tcp | filtered | MySQL tidak dapat dijangkau dari Kali; listener Target tercatat localhost |
| 5000/tcp | open | API BiliCheck, fingerprint HTTP `BiliCheckLabAPI/1.0` |
| 8080/tcp | filtered | Tidak memperoleh respons pada scan |
| 9099/tcp | filtered | Tidak memperoleh respons pada scan |

**Hasil Red Team:** reconnaissance terbatas terhadap Target selesai dan hasil port tersedia pada catatan sesi serta laporan VA. Label layanan dari Nmap tidak digunakan untuk mengganti identifikasi aplikasi ketika fingerprint HTTP menunjukkan API BiliCheck.

**Koordinasi Blue Team:** screenshot Sguil yang tersedia memperlihatkan event historis dari 22 Oktober dan 4 November. Bukti tersebut belum cocok dengan rentang scan 25 November. Korelasi scan terbaru belum terverifikasi.

**Status skenario:** hasil serangan tersedia; deteksi/korelasi dan retest pascarespons belum lengkap dalam bukti yang ditinjau.

Bukti terkait:
- `docs/redteam/evidence/minggu10/03-s1-nmap-scan-terbatas.png`
- `docs/blueteam/evidence/monitoring-seconion/07-m10-s1-nmap-packet-capture.png`
- `docs/blueteam/evidence/monitoring-seconion/08-m10-s1-sguil-event-dengan-korelasi-perlu-dikonfirmasi.png`
- `docs/blueteam/evidence/minggu11/01-m11-s1-sguil-korelasi.png`

## 6. Skenario S2A: Pengujian API Telemetry

**Kategori:** pengujian autentikasi dan validasi aplikasi  
**Target:** `http://192.168.11.5:5000/api/telemetry`  
**Tool:** cURL 8.20.0

Payload valid menggunakan `deviceId=dev01` dan `rawAdc=2048`. Hasil aktual pada 25 November 2026:

| Waktu UTC | Pengujian | Hasil |
|---|---|---|
| 03:33:35 | Tanpa token | HTTP 401 Unauthorized |
| 03:35:04 | Token valid, ADC 2048 | HTTP 202 Accepted |
| 03:35:23 | Token valid, ADC 4096 | HTTP 422 Unprocessable Entity |

Contoh request tanpa token:

```bash
curl -i -X POST http://192.168.11.5:5000/api/telemetry \
  -H 'Content-Type: application/json' \
  -d '{"deviceId":"dev01","rawAdc":2048}'
```

Token pada pengujian dimasukkan secara lokal dan tidak ditulis di dokumen. Catatan sesi menyebut token pernah terekspos pada catatan terminal sebelumnya. Token tersebut harus diperlakukan sebagai rahasia yang perlu dirotasi sebelum penggunaan lanjutan. Bukti yang diperiksa belum memastikan hasil tiga tes tersebut berasal dari token setelah rotasi final.

**Hasil Red Team:** endpoint menolak request tanpa token, menerima payload dengan token valid dan nilai ADC sesuai, serta menolak nilai ADC 4096. Respons API untuk request valid menyatakan `accepted=true`, `validation=passed`, dan `stored=true`. Hal tersebut dicatat sebagai respons aplikasi, bukan pemeriksaan independen database.

**Status skenario:** perilaku API pada sesi yang didokumentasikan terbukti; retest setelah rotasi token final belum terverifikasi.

Bukti:
- `docs/redteam/evidence/minggu11/02-m11-s2-api-401-202-422-final.png`
- `docs/redteam/evidence/minggu11/dokumentasi-redteam-minggu11.md`

## 7. Skenario S2B: MQTT Authenticated Publish-Subscribe

**Kategori:** pengujian autentikasi komunikasi perangkat  
**Broker:** `192.168.11.5:1883`  
**Akun lab:** `esp32_bilicheck`  
**Topic:** `bilicheck/puskesmas01/dev-test/telemetry`

Hasil yang tercatat:

1. Percobaan tanpa kredensial ditolak dengan `CONNACK (5)` dan `Connection Refused: not authorised`.
2. Publisher dengan akun lab diterima dengan `CONNACK (0)` dan mengirim PUBLISH.
3. Subscriber dengan akun lab diterima dengan `CONNACK (0)` dan `SUBACK`.
4. Subscriber menerima pesan pada topic yang sama dengan payload `{"deviceId":"BC-TEST-001","rawAdc":2048}`.

Konfigurasi broker yang dicatat adalah `allow_anonymous false` dan `password_file /etc/mosquitto/passwd`. Password tidak dicantumkan dalam repository.

**Batas hasil:** pengujian membuktikan komunikasi publisher ke subscriber pada broker MQTT. Belum ada bukti consumer yang meneruskan payload ke API atau menyimpan data ke MySQL.

**Status skenario:** authenticated publish-subscribe terbukti pada sisi klien. Integrasi end-to-end tidak dinilai dalam bukti ini.

Bukti:
- `docs/redteam/evidence/minggu11/03-m11-s2-mqtt-authenticated-pub-sub.png`
- `docs/redteam/evidence/minggu11/dokumentasi-redteam-minggu11.md`
- `docs/blueteam/evidence/application/10-konfigurasi-autentikasi-mqtt.png`

## 8. Skenario S3: Audit Autentikasi SSH

**Kategori:** audit kontrol autentikasi layanan administrasi  
**Target:** `192.168.11.5:22`  
**Akun uji:** `soc_test`

Perintah audit metode autentikasi yang tercatat:

```bash
nmap -n -Pn -p 22 --script ssh-auth-methods \
  --script-args='ssh.user=soc_test' 192.168.11.5
```

Hasil yang terlihat adalah `publickey` sebagai metode autentikasi yang didukung. Catatan pengujian juga menyebut tiga percobaan dengan key uji yang tidak terdaftar ditolak dengan `Permission denied (publickey)`.

Screenshot Blue Team Minggu 11 menunjukkan Fail2Ban jail `sshd` aktif, `Currently failed: 1`, `Total failed: 1`, `Currently banned: 0`, dan `Total banned: 0`. Snapshot ini diperlakukan sebagai keadaan pada waktu pengambilan, bukan bukti statistik setelah seluruh percobaan key uji.

**Batas hasil:** tidak ada klaim bahwa Kali diblokir oleh Fail2Ban. Hydra tidak dijalankan pada SSH utama dan tidak ada percobaan dictionary attack terhadap sistem produksi.

**Status skenario:** metode `publickey` dan penolakan key yang tidak terdaftar didukung bukti; tindakan ban tidak terbukti.

Bukti:
- `docs/blueteam/evidence/minggu11/04-m11-s3-ssh-fail2ban-final.png`
- `docs/redteam/evidence/minggu10/07-08-s3-ssh-auth-methods-dan-password-ditolak-gabungan.png`

## 9. Ringkasan skenario dan status

| Skenario | Serang/pengujian | Deteksi/telemetry | Status ringkas |
|---|---|---|---|
| S1 Nmap | Output scan tersedia | Sguil historis ada, tetapi belum cocok dengan waktu scan 25 November | Sebagian |
| S2A API | HTTP 401, 202, 422 tersedia | Respons aplikasi terlihat; retest token final belum terverifikasi | Sebagian |
| S2B MQTT | Penolakan anonim dan authenticated pub-sub tersedia | Output broker/client menunjukkan perilaku autentikasi; bukti client delivery tersedia | Terbukti untuk pub-sub klien |
| S3 SSH | Audit publickey dan penolakan key uji tersedia | SSH/Fail2Ban mencatat kegagalan; ban nol pada snapshot | Sebagian |

## 10. Koordinasi dengan Blue dan Purple Team

Red Team menyerahkan command, hasil, waktu, alamat sumber dan tujuan, port, payload dummy, serta bukti screenshot atau output teks kepada Blue Team. Blue Team memeriksa log host dan Security Onion. Temuan dan keterbatasan kedua sisi disatukan dalam `docs/purple-team-log.md`.

Lima tahap Purple Team Loop dinilai secara terpisah. Ketiadaan alert yang cocok dicatat sebagai hasil pemeriksaan monitoring. Tahap respons dan verifikasi hanya ditandai selesai apabila perubahan kontrol dan pengujian ulang memiliki bukti.

## 11. Kesimpulan

Red Team telah menyelesaikan pengujian terbatas terhadap API dan MQTT pada Target replika. API memperlihatkan penolakan tanpa token, penerimaan payload valid, dan penolakan ADC di luar rentang. MQTT menolak koneksi anonim dan menerima komunikasi publish-subscribe terautentikasi, termasuk penerimaan payload dummy oleh subscriber. Audit SSH menunjukkan metode `publickey`.

Hasil S1 telah memberi gambaran permukaan layanan, tetapi korelasi event Sguil dengan scan terbaru belum terbukti. Status Fail2Ban yang tersedia menunjukkan tidak ada IP yang diblokir pada snapshot. Token API harus diperlakukan secara hati-hati dan retest setelah rotasi final belum terverifikasi. Seluruh catatan ini disiapkan agar hasil Red Team dapat direkonsiliasi dengan bukti Blue Team dan Purple Team secara konsisten.
