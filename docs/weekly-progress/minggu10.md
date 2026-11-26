# Logbook Minggu 10  -  SOC BiliCheck

> **Keterangan file:** Logbook ringkas seluruh pekerjaan, hasil, kendala, dan tindak lanjut Minggu 10.
> **Isi utama:** Baseline, pekerjaan Blue Team, pekerjaan Red Team, S1-S3, simulasi benign, alert rules, dan daftar bukti.
> **Penanggung jawab:** Lead Moh Rifky mengintegrasikan. Adam mengisi Blue Team. Nayla dan Fauzan mengisi Red Team.
> **Status dokumen:** Dokumen aktif berdasarkan bukti yang tersedia.

**Mata kuliah:** TEK1314  -  Keamanan Siber  
**Kelompok/Kelas:** Kelompok 11  -  Kelas A  
**Proyek:** BiliCheck  -  Replika Backend IoT  
**Fokus:** Malware & Sandbox / Monitoring Operations  
**Repository:** `TEK1314-2026-Kel11-KelasA`

> Logbook ini disusun dari screenshot yang tersedia pada arsip. Bagian yang tidak dapat dipastikan ditandai sebagai belum diverifikasi; sesuaikan dengan catatan terminal/output asli sebelum dikumpulkan.

## Pembagian Peran Minggu 10

| Anggota | Peran yang berlaku | Tanggung jawab |
| :--- | :--- | :--- |
| Moh Rifky Marasetya | Lead | Koordinasi, integrasi dokumen, repositori, dan laporan. |
| Nayla Januarisa Prihadi | Red Team | Pengujian dari Kali bersama Fauzan dan pencatatan hasil. |
| Fauzan Nafis Jiyad Akbar | Red Team | Pengujian dari Kali bersama Nayla, Attack Plan, dan bukti. |
| Muhammad Adam Ilyasa | Blue Team | Satu-satunya anggota Blue Team. Menangani seluruh host, aplikasi, database, MQTT, log, Security Onion, Snort, Sguil, dan mitigasi. |
## 1. Tujuan Minggu 10

Melakukan pengujian terkontrol pada Target Node replika, mengamati aktivitas melalui log host dan Security Onion, mendokumentasikan hasil Red/Blue/Purple Team, menyusun alert rules, serta memperbarui dokumen proyek.

## 2. Infrastruktur dan kondisi awal

| Node | Alamat lab | Fungsi |
|---|---|---|
| Target Ubuntu `target-bilicheck` | `192.168.11.5` | Backend/API replika |
| Kali `attacker-kali` | `192.168.11.100` | Red Team dan klien uji |
| Security Onion `SecOnion` | `192.168.11.200` | Packet capture, IDS, Sguil |

Semua pengujian harus tetap terbatas pada Target Node replika di jaringan internal `bilicheck-lab`. Tidak menggunakan VPS produksi, Firebase asli, perangkat mitra, atau jaringan kampus.

Bukti baseline Target yang tersedia menunjukkan UFW aktif, API berjalan, dan MySQL aktif. Namun screenshot tersebut belum menampilkan seluruh elemen baseline seperti hostname, IP, route, waktu, dan daftar seluruh port. Baseline Target masih perlu dilengkapi.

Screenshot versi tool Kali tersedia, tetapi tidak sekaligus memperlihatkan IP Kali. Ambil bukti IP terpisah bila requirement baseline harus dibuktikan lengkap.

## 3. Pekerjaan Blue Team Host dan Aplikasi (Adam)

### API

- Endpoint tanpa token menghasilkan HTTP `401` pada screenshot pengujian.
- Request dengan token dan `rawAdc=2048` menghasilkan HTTP `202` serta respons penyimpanan berhasil pada screenshot Red Team.
- Request dengan token dan `rawAdc=4096` menghasilkan HTTP `422`.
- Bukti ketiga hasil berada pada lebih dari satu screenshot; satu bukti gabungan yang bersih belum tersedia.
- Screenshot lama rotasi token memperlihatkan nilai token. Screenshot itu dipindahkan ke folder karantina di luar `docs` dan tidak boleh di-commit. Pastikan token terkait sudah dirotasi jika masih aktif.

### MySQL

- Screenshot menunjukkan MySQL bind pada `127.0.0.1:3306`.
- Screenshot `SHOW GRANTS` memperlihatkan akun aplikasi dengan hak `INSERT` pada tabel telemetri serta data dummy.
- Tidak ada klaim bahwa akun aplikasi memiliki akses baca; verifikasi tetap mengikuti keluaran `SHOW GRANTS`.

### MQTT

- Screenshot konfigurasi menunjukkan listener `192.168.11.5:1883`, `allow_anonymous false`, dan `password_file`.
- Screenshot klien menunjukkan percobaan anonim ditolak.
- Screenshot klien akun lab menggunakan user `esp32_bilicheck` dan command publish selesai tanpa error yang terlihat.
- Salah satu screenshot log Mosquitto memperlihatkan client otomatis melakukan subscribe/publish pada kondisi baseline anonim. Itu bukan bukti autentikasi sukses.
- Log server yang secara tegas memperlihatkan hasil autentikasi akhir masih perlu diambil ulang.
- Jangan mengklaim payload MQTT diteruskan ke API/database tanpa bukti consumer/integrasi end-to-end.

## 4. Pekerjaan Blue Team Monitoring dan SIEM (Adam)

- Screenshot menunjukkan Elastic Stack (`elasticsearch`, `logstash`, `kibana`, `freqserver`) berstatus `[ OK ]`.
- Status agent/sensor NSM juga terlihat `[ OK ]`.
- Konfigurasi `IDS_ENGINE_ENABLED="yes"` terlihat.
- Pada screenshot yang sama, pemeriksaan proses Snort tidak memperlihatkan baris proses yang dapat dipastikan dan log menyebut file statistik Snort tidak ditemukan. Oleh karena itu, engine Snort belum dinyatakan terbukti berjalan.
- PCAP dan tampilan Sguil tersedia. Salah satu Sguil memperlihatkan event dari Kali ke Target port 5000, tetapi korelasi dengan scan Nmap perlu dicocokkan berdasarkan timestamp dan port.
- File yang semula bernama `12-m10-snort-local-rules-check.png` ternyata berisi packet capture tambahan, bukan tampilan local rules.
- Screenshot `snort -T` menunjukkan test mode mulai berjalan, tetapi bagian akhir/success summary tidak terlihat.

## 5. Pekerjaan Red Team (Nayla dan Fauzan)

### S1  -  Reconnaissance

Scan Nmap terbatas ke Target `192.168.11.5` tersedia pada `docs/redteam/evidence/minggu10/03-s1-nmap-scan-terbatas.png`. Bukti PCAP dan Sguil diletakkan di `docs/blueteam/evidence/monitoring-seconion/`. Korelasi Sguil terhadap scan perlu dikonfirmasi.

### S2  -  MQTT/API

- API: bukti `202` dan `422` tersedia; bukti `401` tersimpan terpisah.
- MQTT: terdapat percobaan publikasi anonim yang ditolak dan percobaan publish dengan akun lab.
- Screenshot awal bernama baseline justru menunjukkan penolakan anonim; kondisi broker yang menerima anonim tidak dibuktikan oleh screenshot itu.
- Log Mosquitto yang menunjukkan client otomatis melakukan publish tanpa user tercatat sebagai bukti aktivitas baseline, bukan autentikasi final.

### S3  -  SSH Authentication Audit

- Nmap `ssh-auth-methods` menunjukkan metode `publickey`.
- Percobaan password ditolak dengan `Permission denied (publickey)`.
- Log SSH memperlihatkan `Invalid user soc_test` dari Kali.
- Jail Fail2Ban `sshd` terlihat aktif, tetapi `Currently banned: 0` dan `Total banned: 0`.
- Tidak ada klaim bahwa Fail2Ban memblokir Kali.

## 6. Simulasi Aktivitas Non-Malware / Sandbox

Screenshot journal dan auditd memperlihatkan aktivitas file dummy pada direktori simulasi. Kegiatan ini adalah simulasi aktivitas host yang tidak berbahaya, bukan eksekusi malware atau analisis malware di sandbox.

## 7. Alert Rules

Dokumen `docs/blueteam/monitoring_alert_rules.md` dibuat sebagai matriks DET-01 sampai DET-05. Status setiap deteksi dibatasi pada apa yang benar-benar didukung bukti. Custom Snort rule belum boleh ditulis sebagai aktif tanpa bukti bahwa rule dimuat oleh engine dan menghasilkan event.

## 8. Temuan dan Blind Spot

1. Bukti Sguil perlu dicocokkan ulang dengan waktu dan port scan Nmap.
2. Status agent Snort tidak dengan sendirinya membuktikan proses Snort berjalan.
3. Log MQTT yang tersedia belum seluruhnya membedakan hasil autentikasi final di sisi broker.
4. Bukti API 401, 202, dan 422 tersedia secara terpisah; bukti gabungan bersih perlu diperbarui.
5. Screenshot token API yang terlihat harus dikarantina dan token harus dirotasi jika belum.
6. Screenshot baseline Target dan Kali belum sepenuhnya memenuhi seluruh elemen yang diminta.

## 9. Mitigasi yang sudah terlihat pada bukti

- MQTT dikonfigurasi dengan `allow_anonymous false` dan `password_file`.
- MySQL listen pada localhost.
- SSH memperlihatkan `passwordauthentication no` dan `allowusers socadmin`.
- Environment file API terlihat memiliki permission `600` pada screenshot kontrol setelah mitigasi; screenshot yang memperlihatkan token tidak disimpan di repository.

Catatan: keberadaan screenshot kondisi setelah mitigasi bukan otomatis bukti bahwa seluruh pengujian ulang setelah mitigasi berhasil.

## 10. Pekerjaan yang Dibawa ke Minggu 11

- Verifikasi dan dokumentasikan proses Snort aktual serta validasi konfigurasi sampai output akhir.
- Lengkapi korelasi event Sguil dengan sesi scan.
- Ambil log Mosquitto final yang menunjukkan anonim ditolak dan akun lab diterima.
- Ambil screenshot API yang aman dan lengkap setelah token final dirotasi.
- Lengkapi baseline semua node.
- Lengkapi tahap respons dan verifikasi Purple Team yang belum terbukti.
- Susun Vulnerability Assessment Report sesuai rencana Minggu 11.

## 11. Daftar Bukti

Lihat daftar file pada:

- `docs/blueteam/evidence/application/`
- `docs/blueteam/evidence/hardening/`
- `docs/blueteam/evidence/monitoring-seconion/`
- `docs/blueteam/evidence/minggu10/`
- `docs/redteam/evidence/minggu10/`
- `docs/purple-team/evidence/minggu10/`

Rincian ketidaksesuaian nama dan interpretasi screenshot ada di `docs/evidence-review-minggu10.md`.

## 12. Kesimpulan

Minggu 10 memiliki bukti aktivitas Red Team, beberapa kontrol Blue Team, aktivitas benign file, dan monitoring. Namun, kelengkapan monitoring Snort, korelasi event, bukti autentikasi MQTT sisi server, serta verifikasi setelah mitigasi masih perlu dilengkapi. Status akhir Minggu 10 harus mengikuti hasil verifikasi tersebut dan tidak boleh ditingkatkan hanya karena folder bukti sudah tersusun.