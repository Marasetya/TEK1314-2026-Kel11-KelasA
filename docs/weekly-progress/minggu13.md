# Logbook Minggu 13 - SOC BiliCheck

**Mata kuliah:** TEK1314 Keamanan Siber  
**Kelompok/Kelas:** Kelompok 11, Kelas A  
**Proyek:** SOC BiliCheck, replika backend IoT  
**Tanggal kegiatan:** 26 November 2026  
**Fokus:** Remediation, Verification, dan Retest.

## 1. Tujuan Minggu 13

Menyelesaikan temuan keamanan dari Minggu 11 dan Minggu 12, menguji ulang kontrol utama, memastikan sensor Security Onion menerima trafik, dan menyiapkan bukti untuk laporan akhir.

## 2. Kegiatan yang tercatat

### 2.1 Konsolidasi bukti Minggu 12

Folder akuisisi DFIR M12 dipindahkan dari Ubuntu ke Windows. PCAP tambahan M12 berhasil dipindahkan. Output PowerShell yang diberikan menunjukkan SHA-256 aktual sama dengan hash sumber dan `Match: True`. Manifest M12 dibuat.

### 2.2 Baseline tiga node

Ubuntu Target dan Kali mencatat waktu UTC, host, interface, route, dan informasi service/tool. Ubuntu menunjukkan clock synchronized dan NTP active. Security Onion menunjukkan interface sniffing `eth0` UP serta status proses sensor pada screenshot.

### 2.3 SSH dan Fail2Ban

File `/etc/fail2ban/jail.d/bilicheck.local` dibackup dan ditambahkan journalmatch untuk unit `ssh.service` dan proses `sshd`. Validasi konfigurasi berhasil. Lima percobaan key yang tidak terdaftar ditolak oleh SSH. Satu percobaan username tidak valid tercatat pada journal dan menaikkan Fail2Ban counter menjadi satu. Tidak ada IP yang masuk daftar banned pada bukti tersebut.

### 2.4 API

Service `bilicheck-api.service` diidentifikasi. EnvironmentFile berada di `/etc/bilicheck-api/bilicheck-api.env`, owner `root:root`, mode `600`. Token dirotasi dan service aktif kembali. Retest dari Kali dan log service API menunjukkan respons berurutan HTTP 401, 401, 202, dan 422.

Batasan: token lama pada pengujian menggunakan string dummy, bukan token lama aktual. Token baru sempat tampil pada catatan sesi sehingga perlu rotasi ulang. Jangan menyalin nilai token ke laporan.

### 2.5 MQTT

Konfigurasi broker menunjukkan listener `192.168.11.5:1883`, `allow_anonymous false`, dan password file. Uji anonim ditolak dengan CONNACK 5. Akun lab berhasil publish-subscribe dengan CONNACK 0, dan subscriber menerima payload dummy. Log file Mosquitto mencatat kedua kondisi.

### 2.6 Security Onion dan Nmap S1

Status sensor pada screenshot menunjukkan `pcap_agent`, `snort_agent-1`, `snort-1`, dan `barnyard2-1` berstatus `[ OK ]`. Nmap retest dari Kali dilakukan pada 26 November 2026. Capture langsung pada interface `eth0` Security Onion menghasilkan PCAP 42.654 byte. Pemeriksaan file menunjukkan 343 frame IPv4 TCP pada komunikasi dua arah antara Kali dan Ubuntu. SHA-256 PCAP adalah `72d1b237c66706675d775a77861f43df290920e90576afbcb88d5b90283d2383`.

Screenshot Sguil M13 untuk event scan terbaru belum tersedia dalam arsip yang diperiksa. Karena itu, keberhasilan deteksi alert Sguil belum dapat disimpulkan.

### 2.7 Final posture Ubuntu

Snapshot final menunjukkan UFW active dengan default incoming deny. API diizinkan dari subnet lab, MQTT dari Kali, dan MySQL bind ke localhost. SSH diizinkan dari Anywhere pada IPv4 dan IPv6. Snapshot akhir Fail2Ban menunjukkan failed 0 dan banned 0.

## 3. Hasil utama

| Area | Hasil |
|---|---|
| Evidence M12 | Transfer dan hash PCAP cocok |
| SSH | Key tidak terdaftar ditolak |
| Fail2Ban | Satu invalid-user event dihitung, ban tidak terbukti |
| API | HTTP 401, 401, 202, 422 dan log server sesuai |
| MQTT | Anonymous ditolak, authenticated pub-sub berhasil |
| Security Onion | Sensor berstatus OK dan PCAP scan berhasil direkam |
| Sguil | Alert M13 belum terverifikasi |
| Final posture | UFW aktif, MySQL localhost, SSH rule perlu review |

## 4. Kendala dan keterbatasan

1. Clipboard VirtualBox beberapa kali memotong alamat IP saat dipaste. Penggunaan variabel IP membantu menghindari kesalahan input.
2. SCP dari Kali ke Ubuntu gagal karena target hanya menerima public key. Arsip Kali kemudian dikirim menggunakan HTTP sementara pada jaringan lab.
3. Transfer dari Security Onion ke Ubuntu melalui port HTTP yang dicoba mengalami timeout. Arsip yang akhirnya tersedia pada repository berisi PCAP M13, sedangkan folder baseline teks Security Onion tidak ditemukan pada paket yang diperiksa. Screenshot sensor tetap tersedia.
4. Screenshot Sguil M13 tidak ada pada paket bukti yang diperiksa.
5. Fail2Ban belum membuktikan tindakan ban IP.
6. Token baru API terekspos pada catatan sesi dan harus dirotasi ulang.

## 5. Tindak lanjut

- Ambil screenshot Sguil terbaru atau dokumentasikan secara jujur jika tidak ada alert.
- Lakukan retest Fail2Ban yang dapat membuktikan ban atau dokumentasikan keterbatasan action.
- Rotasi ulang token API dan jangan pernah menampilkannya.
- Perkuat password MQTT lab dan evaluasi TLS.
- Tinjau aturan SSH UFW yang mengizinkan Anywhere.
- Selesaikan manifest SHA-256 M13, audit Git, Incident Report Final, video demo, dan slide.

## 6. Catatan screenshot Sguil

Pengguna mengonfirmasi bahwa screenshot `m13-08-sguil-retest-events.png` sudah tersedia pada folder kerja, tetapi file tersebut tidak ikut dalam arsip `docs(3).zip` yang diperiksa saat penyusunan dokumentasi. Skrip finalisasi akan mencari file dengan nama persis tersebut di bawah folder `docs` dan menyalinnya ke `blueteam/evidence/minggu13/screenshots/`. Keberadaan screenshot tidak otomatis membuktikan alert terdeteksi. Isi gambar harus diperiksa dan dicatat berdasarkan timestamp, source, destination, serta detail event yang terlihat.

## 7. Kesimpulan Minggu 13

Minggu 13 menghasilkan perbaikan dan retest untuk API, MQTT, dan konfigurasi Fail2Ban, serta membuktikan bahwa Security Onion menerima trafik scan melalui interface sniffing dan menghasilkan PCAP yang dapat dianalisis. Hasil tersebut belum membuktikan alert Sguil untuk scan M13 atau pemblokiran IP oleh Fail2Ban. Beberapa pengamanan kredensial dan dokumentasi akhir masih harus diselesaikan sebelum proyek dinyatakan siap dikumpulkan.
