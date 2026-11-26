# Monitoring Alert Rules - SOC BiliCheck

**Mata kuliah:** TEK1314 - Keamanan Siber  
**Kelompok/Kelas:** Kelompok 11, Kelas A  
**Proyek:** BiliCheck, replika backend IoT  
**Periode pembaruan:** 25 November 2026  
**Penanggung jawab monitoring:** Muhammad Adam Ilyasa, Blue Team

## 1. Tujuan dan ruang lingkup

Dokumen ini menjadi matriks pemantauan untuk lima area DET-01 sampai DET-05. Status di bawah membedakan aktivitas yang diuji, telemetry yang tersedia, alert yang benar-benar terlihat, dan validasi rule pada engine. Pengujian hanya mengacu pada Target replika `192.168.11.5` di jaringan `bilicheck-lab`.

**Catatan status:** sebuah paket yang tertangkap PCAP tidak otomatis berarti alert Snort muncul. Status agent OK tidak otomatis membuktikan proses Snort aktif. Koneksi TCP tidak otomatis membuktikan autentikasi aplikasi berhasil.

## 2. Definisi status

- **Rencana:** kriteria deteksi telah ditentukan, tetapi pengujian belum tersedia.
- **Diuji:** aktivitas atau sumber telemetry telah diperiksa.
- **Terbukti:** bukti menunjukkan perilaku atau alert yang sesuai dengan kriteria.
- **Sebagian:** bukti hanya mendukung sebagian kriteria.
- **Belum terverifikasi:** sumber bukti yang diperlukan belum tersedia atau belum dapat dikorelasikan.
- **Blind spot:** aktivitas yang diuji tidak menghasilkan alert yang dapat ditemukan pada sumber monitoring yang diperiksa.
- **Aktif:** hanya digunakan jika rule terbukti dimuat dan dijalankan oleh engine.

## 3. Matriks DET-01 sampai DET-05

| ID | Tujuan monitoring | Sumber telemetry | Bukti yang tersedia | Status hasil | Batas interpretasi |
|---|---|---|---|---|---|
| DET-01 | Pola reconnaissance atau koneksi scan dari Kali ke Target | PCAP, Snort, Sguil | Scan Nmap 25 November tercatat pada dokumentasi VA. Screenshot Sguil memperlihatkan event historis 22 Oktober dan 4 November dari pasangan IP yang sama. | Sebagian, korelasi scan terbaru belum terverifikasi | Event historis tidak boleh disebut sebagai alert dari scan 25 November. Cocokkan timestamp UTC, source, destination, dan port. |
| DET-02 | Koneksi dan autentikasi MQTT | Log Mosquitto, output `mosquitto_pub` dan `mosquitto_sub`, PCAP | Konfigurasi `allow_anonymous false` dan password file tercatat. Red Team membuktikan penolakan anonim, `CONNACK (0)`, `SUBACK`, dan subscriber menerima payload. | Terbukti pada sisi klien; log server pendukung terbatas | Output klien membuktikan perilaku yang terlihat klien. Jangan menyimpulkan alur ke API atau MySQL tanpa bukti consumer. |
| DET-03 | Kegagalan autentikasi SSH | SSH journal, auth log, Fail2Ban | Nmap NSE hanya menunjukkan `publickey`; screenshot Blue Team menunjukkan jail `sshd` aktif, `Currently failed: 1`, `Total failed: 1`, `Currently banned: 0`, `Total banned: 0`. | Diuji; tidak ada ban pada snapshot | Jail aktif bukan bukti IP diblokir. Angka statistik hanya berlaku pada waktu screenshot. |
| DET-04 | Autentikasi dan validasi API telemetry | Respons HTTP dan log aplikasi | Red Team menunjukkan HTTP 401 tanpa token, 202 untuk token valid dan ADC 2048, serta 422 untuk ADC 4096. | Terbukti pada sesi uji; retest token final belum terverifikasi | Respons HTTP berasal dari sisi klien. Snort tidak otomatis membuktikan logika autentikasi aplikasi. Token yang pernah terekspos perlu dirotasi. |
| DET-05 | Aktivitas file pada direktori simulasi | auditd dan journald | Screenshot Minggu 10 menunjukkan aktivitas file dummy pada host dan audit log. | Diuji | Ini simulasi aktivitas file benign, bukan eksekusi atau deteksi malware sungguhan. |

## 4. Status komponen Security Onion dan Snort

Bukti historis menunjukkan Elastic Stack dan sejumlah komponen sensor NSM pernah berstatus OK. Namun, paket bukti yang diperiksa belum cukup untuk menyatakan proses Snort berjalan normal pada interface sniffing selama sesi scan 25 November.

Pemeriksaan berikut harus dibedakan:

1. Status Elastic Stack.
2. Status sensor dan agent NSM.
3. Keberadaan proses Snort pada saat pemeriksaan.
4. Hasil validasi konfigurasi Snort sampai baris akhir dan exit status.
5. Keberhasilan menangkap paket pada PCAP.
6. Keberadaan alert yang sesuai di Sguil.

Validasi konfigurasi dengan mode test atau dump tidak dengan sendirinya membuktikan bahwa engine sensor sedang memproses trafik pada interface monitoring.

## 5. Status custom rule

Custom rule DET-01 sampai DET-03 tidak dinyatakan aktif berdasarkan paket bukti yang ditinjau. Status aktif baru dapat digunakan apabila file rule, include pada konfigurasi yang benar, hasil validasi, dan pemuatan oleh engine telah dibuktikan.

| Area | Makna telemetry | Hal yang tidak boleh disimpulkan tanpa bukti tambahan |
|---|---|---|
| DET-01 | Pola koneksi/scan dapat terlihat pada PCAP atau alert | Paket tertangkap tidak otomatis berarti alert Sguil cocok dengan scan |
| DET-02 | Koneksi dan hasil autentikasi dapat diperiksa dari broker/client | TCP port 1883 terbuka tidak membuktikan autentikasi atau delivery |
| DET-03 | Kegagalan login dapat terlihat pada log SSH | Koneksi ke port 22 tidak membuktikan kegagalan login atau ban |
| DET-04 | Status HTTP dan validasi payload terlihat pada aplikasi | Snort tidak otomatis membaca hasil logika API |
| DET-05 | auditd/journald mencatat aktivitas file host | Aktivitas file dummy bukan malware |

## 6. Tindak lanjut monitoring

- [ ] Cocokkan waktu scan Nmap 25 November dengan PCAP dan Sguil berdasarkan UTC, source, destination, serta port.
- [ ] Jika event tidak ditemukan, catat hasil pencarian sebagai blind spot untuk sesi yang diperiksa.
- [ ] Simpan output status proses Snort dan validasi konfigurasi lengkap secara terpisah.
- [ ] Verifikasi file rule dan include sebelum menyatakan custom rule aktif.
- [ ] Simpan log broker jika dibutuhkan untuk menguatkan hasil MQTT dari sisi server.
- [ ] Pertahankan angka Fail2Ban sesuai snapshot dan waktu pengambilan.
- [ ] Gunakan hasil audit aplikasi untuk DET-04 dan audit host untuk DET-05.

## 7. Referensi bukti

- `docs/blueteam/evidence/monitoring-seconion/`
- `docs/blueteam/evidence/minggu11/01-m11-s1-sguil-korelasi.png`
- `docs/blueteam/evidence/minggu11/04-m11-s3-ssh-fail2ban-final.png`
- `docs/redteam/evidence/minggu11/02-m11-s2-api-401-202-422-final.png`
- `docs/redteam/evidence/minggu11/03-m11-s2-mqtt-authenticated-pub-sub.png`
- `docs/blueteam/evidence/minggu10/03-journal-benign-infiltration-simulation.png`
- `docs/blueteam/evidence/minggu10/04-auditd-benign-file-activity.png`

## 8. Kesimpulan

Matriks DET-01 sampai DET-05 telah diselaraskan dengan bukti Minggu 11 yang tersedia. Hasil API dan MQTT dicatat berdasarkan output Red Team, sedangkan event Sguil dan Fail2Ban dicatat berdasarkan bukti Blue Team. Event historis tidak digunakan untuk mengklaim deteksi pada scan terbaru. Status proses Snort dan pemuatan custom rule tetap belum terverifikasi sampai bukti proses dan validasi yang lengkap tersedia.
