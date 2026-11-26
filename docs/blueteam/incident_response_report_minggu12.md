# Incident Response Report Minggu 12

**Mata kuliah:** TEK1314 - Keamanan Siber  
**Kelompok/Kelas:** Kelompok 11, Kelas A  
**Proyek:** SOC BiliCheck, replika backend IoT  
**Periode investigasi:** 26 November 2026  
**Penanggung jawab Blue Team:** Muhammad Adam Ilyasa  
**Case ID:** SOC-BILICHECK-DFIR-M12-20261126

## 1. Ringkasan Eksekutif

Investigasi Minggu 12 memeriksa aktivitas jaringan dan autentikasi pada Ubuntu Target `192.168.11.5`, menggunakan log sistem, journal SSH, status Fail2Ban, event historis Sguil, dokumentasi hasil pengujian API dan MQTT, serta PCAP tambahan yang direkam pada 26 November 2026.

Log SSH tanggal 25 November menunjukkan koneksi dari Kali Linux `192.168.11.100`, termasuk beberapa entri `Invalid user soc_test` dan koneksi yang ditutup pada tahap pre-authentication. Sguil memperlihatkan alert scan historis pada 22 Oktober dan event port 5000 pada 4 November, tetapi bukti yang diberikan belum mengonfirmasi alert yang cocok pada jendela scan 25 November. Snapshot Fail2Ban M12 menunjukkan jail `sshd` tersedia, tetapi `Total failed` dan `Total banned` sama-sama 0.

PCAP tambahan 26 November berhasil merekam 9 paket tanpa packet drop. Capture memperlihatkan TCP handshake dari Kali ke port SSH Ubuntu dan banner OpenSSH dari server, lalu RST dari klien. Ini membuktikan keterjangkauan TCP SSH pada pengujian tambahan, bukan autentikasi atau login berhasil.

Berdasarkan bukti yang tersedia, ditemukan aktivitas reconnaissance dan probing autentikasi. Tidak ada bukti langsung mengenai login tidak sah yang berhasil, kompromi host, eksfiltrasi data, perubahan data tanpa izin, atau gangguan layanan. Temuan monitoring dan Fail2Ban memerlukan tindak lanjut.

## 2. Gambaran Insiden

| Parameter | Keterangan |
|---|---|
| Jenis aktivitas | Network reconnaissance dan SSH authentication probing |
| Sumber utama yang diamati | Kali Linux `192.168.11.100` |
| Target | Ubuntu `target-bilicheck`, `192.168.11.5` |
| Monitoring | Security Onion `192.168.11.200`, Sguil 0.9.0 |
| Rentang kejadian utama | 25 November 2026, termasuk jendela Nmap 03:22:42-03:24:15 UTC |
| Akuisisi awal | 26 November 2026, sekitar 01:40:32 UTC |
| Capture tambahan | 26 November 2026, filename timestamp 02:44:24 UTC, paket terlihat sekitar 02:45:11 UTC |
| Status kompromi | Tidak terbukti dari bukti yang diberikan |

## 3. Ruang Lingkup Investigasi

Investigasi dibatasi pada replika laboratorium VirtualBox `bilicheck-lab` dengan jaringan internal `192.168.11.0/24`. Pemeriksaan mencakup Ubuntu Target, Kali Linux sebagai sumber pengujian, dan Security Onion sebagai monitoring node. Seluruh pengujian aplikasi yang dirujuk menggunakan data dummy.

Tidak ada pemeriksaan terhadap VPS produksi, Firebase asli, perangkat sensor fisik, jaringan kampus, atau sistem milik mitra. Tidak dilakukan perubahan firewall, konfigurasi SSH, MQTT, API, atau service selama akuisisi bukti.

## 4. Metodologi dan Akuisisi Bukti

Akuisisi Ubuntu dilakukan menggunakan journalctl, status Fail2Ban, informasi host, interface, port, service, dan salinan log file yang tersedia. Folder kasus sumber adalah `/home/socadmin/DFIR_M12_20261126T014140Z`.

Bukti monitoring diperiksa melalui Sguil 0.9.0 yang terhubung ke localhost. Event historis dan detail header paket Alert ID `5.1183` dicatat. PCAP historis untuk alert tersebut belum terbukti tersedia. Sebagai bukti tambahan yang terpisah, tcpdump dijalankan pada interface `enp0s8` dengan filter host Kali dan TCP port 22. PCAP tambahan tersebut kemudian dibaca menggunakan tcpdump dan dihitung SHA-256.

File mentah disimpan terpisah dari analisis. Manifest SHA-256 final dibuat oleh skrip PowerShell setelah file bukti dipindahkan ke direktori proyek.

## 5. Daftar Bukti

Daftar dinamis seluruh file tersedia pada `evidence/minggu12/evidence_register.md`. Bukti utama meliputi:

| ID kategori | Bukti | Sumber | Keterangan |
|---|---|---|---|
| E-01 | Journal sistem, SSH, Fail2Ban, MQTT, Apache | Ubuntu Target | Log hasil akuisisi. Isi tiap file harus ditinjau. |
| E-02 | `auth.log`, syslog, dan log Mosquitto | Ubuntu Target | Salinan file log yang tersedia saat akuisisi. |
| E-03 | Status Fail2Ban dan jail `sshd` | Ubuntu Target | Snapshot M12 menunjukkan 0 failed dan 0 banned. |
| E-04 | Event historis dan detail paket Sguil | Security Onion | Alert SSH scan 22 Oktober dan event 4 November. |
| E-05 | PCAP tambahan `M12_SUPPLEMENTAL_20261126T024424Z.pcap` | Ubuntu Target | 9 paket, 0 dropped, TCP SSH test 26 November. |
| E-06 | SHA-256 PCAP dan manifest proyek | Ubuntu Target dan PowerShell | Hash PCAP awal dan manifest penghitungan ulang. |
| E-07 | Bukti API dan MQTT Minggu 11 | Dokumentasi Red Team | Bukti sisi klien, bukan korelasi server-side independen. |

## 6. Analisis Timeline

Timeline rinci tersedia pada `evidence/minggu12/incident_timeline.md`. Ringkasan penting:

1. Pada 22 Oktober 2026 pukul 07:35:07 UTC, Sguil menampilkan alert `ET SCAN Potential SSH Scan` dari `192.168.11.100` ke `192.168.11.5:22`.
2. Pada 4 November 2026, Sguil menampilkan event historis pada port 5000.
3. Pada 25 November 2026 pukul 03:22:42-03:24:15 UTC, pengujian Nmap Minggu 11 berlangsung. Log SSH mencatat koneksi dari Kali ditutup pada 03:22:48.864887 UTC.
4. Pada 25 November pukul 03:45:35 dan 03:45:57 UTC, log SSH mencatat beberapa `Invalid user soc_test` dari Kali.
5. Pengujian API yang dicatat Red Team menghasilkan HTTP 401 pada 03:33:35, HTTP 202 pada 03:35:04, dan HTTP 422 pada 03:35:23 UTC.
6. Pada 26 November, PCAP tambahan merekam sesi uji TCP SSH yang baru dilakukan.

Event Sguil tanggal 22 Oktober dan 4 November tidak digunakan untuk menyatakan scan 25 November telah terdeteksi.

## 7. Temuan Forensik

### 7.1 SSH

Log mencatat koneksi dari `192.168.11.100` dan percobaan nama pengguna `soc_test` yang tidak valid. Tidak ada baris `Accepted` dari IP Kali pada potongan log yang dibagikan. Login publickey yang terlihat berasal dari `10.0.2.2`, bukan IP Kali. Baris `Connection closed` tidak otomatis berarti autentikasi berhasil.

### 7.2 Fail2Ban

Jail `sshd` tersedia. Snapshot akuisisi M12 menunjukkan `Currently failed: 0`, `Total failed: 0`, `Currently banned: 0`, dan `Total banned: 0`. Snapshot Blue Team M11 sebelumnya menunjukkan satu kegagalan dan nol IP diblokir. Perbedaan antar-snapshot dan entri invalid user pada log SSH perlu diteliti. Belum ada bukti bahwa Fail2Ban memblokir Kali.

### 7.3 Security Onion

Sguil memiliki event historis dari IP Kali ke Ubuntu, termasuk SSH scan 22 Oktober. Detail Alert ID `5.1183` menunjukkan paket TCP SYN ke port 22 tanpa payload. Korelasi dengan jendela scan 25 November belum terverifikasi dari event yang diperlihatkan.

### 7.4 PCAP tambahan

PCAP tambahan menunjukkan SYN, SYN-ACK, ACK, penutupan sesi, banner OpenSSH, lalu RST. Port TCP 22 dapat dijangkau. Tidak ada bukti autentikasi SSH atau login pada capture tersebut. Hash dari Ubuntu yang harus dicocokkan pada salinan Windows adalah `3cf22256fbf085acce4d5a23482d5c5965b9e96ca6df0dc01f7a5bd1cf0c9fbd`.

### 7.5 API dan MQTT

Hasil API 401, 202, dan 422 tersedia dari dokumentasi pengujian sisi klien. Bukti log server API belum lengkap. Hasil MQTT menunjukkan koneksi anonim ditolak dan subscriber menerima payload dummy setelah autentikasi, tetapi korelasi log broker dan integrasi ke API atau MySQL belum dibuktikan.

## 8. Penilaian Dampak

Tidak ada bukti langsung tentang kebocoran data, perubahan data tanpa izin, login tidak sah yang berhasil, atau gangguan layanan. Dampak CIA yang terkonfirmasi belum ditemukan. Risiko yang perlu ditindaklanjuti berhubungan dengan permukaan layanan yang dapat dijangkau, ketidakselarasan statistik Fail2Ban, korelasi alert Security Onion, serta kelengkapan log aplikasi.

Penilaian rinci tersedia pada `evidence/minggu12/analysis/impact_assessment.md`.

## 9. Respons Insiden

Pada tahap akuisisi tidak dilakukan pemblokiran IP, perubahan firewall, penghentian service, penghapusan file, atau reset VM. Log dikumpulkan, PCAP tambahan direkam, dan checksum dihitung. Keputusan ini mempertahankan kondisi bukti sebelum remediation.

Rincian tindakan dan rekomendasi tersedia pada `evidence/minggu12/analysis/response_actions.md`.

## 10. Keterbatasan Investigasi

1. PCAP historis untuk alert Sguil 22 Oktober belum terbukti tersedia.
2. Event Sguil yang cocok dengan scan 25 November belum diperlihatkan.
3. Isi lengkap log MQTT dan Apache perlu diperiksa sebelum digunakan untuk klaim rinci.
4. `apache_events.txt` yang terlihat hanya berukuran 17 byte. Log API dapat berada pada service atau file lain.
5. Statistik Fail2Ban M12 menunjukkan nol kegagalan, sedangkan log SSH memiliki entri invalid user. Penyebab perbedaan belum dipastikan.
6. Bukti API dan MQTT dari Red Team terutama merupakan bukti sisi klien.
7. PCAP tambahan dibuat pada 26 November dan tidak menggantikan capture historis.
8. Transkripsi Sguil memiliki perbedaan dengan dokumentasi M11 untuk port VNC dan satu timestamp ICMP. Nilai akhir perlu dicocokkan dengan screenshot asli.

## 11. Rekomendasi

1. Periksa filter dan backend Fail2Ban untuk memastikan pola `Invalid user` dan unit SSH dipantau.
2. Verifikasi sensor dan retensi PCAP Security Onion, kemudian lakukan pencarian event 25 November berdasarkan IP, port, dan timestamp UTC.
3. Temukan log service API yang benar dan korelasikan hasil 401, 202, dan 422.
4. Periksa log Mosquitto untuk mencocokkan autentikasi anonim dan sesi akun lab.
5. Rotasi token API yang pernah tercatat utuh pada catatan pengujian sebelum penggunaan lanjutan.
6. Pertahankan bukti mentah, checksum, dan dokumentasi transfer. Lakukan remediation dan retest pada tahap terpisah.

## 12. Kesimpulan

Investigasi Minggu 12 menemukan bukti aktivitas reconnaissance dan probing autentikasi pada lingkungan laboratorium SOC BiliCheck. Log SSH menunjukkan beberapa percobaan menggunakan akun tidak valid dari Kali, sedangkan Sguil menyimpan event historis scan dari pasangan IP yang sama. PCAP tambahan membuktikan port TCP 22 merespons pada uji konektivitas 26 November.

Bukti yang tersedia belum membuktikan kompromi, login tidak sah yang berhasil, eksfiltrasi data, perubahan data tanpa izin, atau gangguan layanan. Kualitas monitoring dan pencatatan Fail2Ban masih perlu ditingkatkan dan diverifikasi. Kesimpulan akhir dibatasi pada bukti yang berhasil dikumpulkan dan tidak mengubah event historis menjadi klaim deteksi baru.

## 13. Lampiran dan lokasi dokumen

- `evidence/minggu12/evidence_register.md`
- `evidence/minggu12/chain_of_custody.md`
- `evidence/minggu12/incident_timeline.md`
- `evidence/minggu12/analysis/forensic_analysis.md`
- `evidence/minggu12/analysis/impact_assessment.md`
- `evidence/minggu12/analysis/response_actions.md`
- `evidence/minggu12/hashes/SHA256SUMS_M12.csv`
- `evidence/minggu12/hashes/SHA256SUMS_M12.txt`
