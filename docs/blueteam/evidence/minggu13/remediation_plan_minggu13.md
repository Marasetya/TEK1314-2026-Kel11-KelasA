# Remediation Plan Minggu 13 - SOC BiliCheck

## 1. Tujuan

Minggu 13 merupakan final technical sprint. Tujuannya bukan menambah fitur baru, melainkan memperbaiki atau mengklarifikasi temuan terdahulu, mengulang skenario utama, dan menghasilkan bukti before-after yang dapat diaudit.

## 2. Ruang lingkup

Target tetap berada pada jaringan internal VirtualBox `bilicheck-lab`.

| Node | Hostname | IP lab | Fungsi |
|---|---|---:|---|
| Ubuntu Target | `target-bilicheck` | `192.168.11.5` | Backend replika, SSH, API, MySQL, Mosquitto |
| Kali Linux | `attacker-kali` | `192.168.11.100` | Pengujian Red Team |
| Security Onion Classic | `SecOnion` | `192.168.11.200` pada interface manajemen | Monitoring dan packet capture |

Pengujian dibatasi pada host replika dan data dummy. Tidak ada pengujian terhadap VPS produksi, Firebase asli, perangkat sensor mitra, atau jaringan kampus.

## 3. Temuan yang menjadi dasar remediation

| ID | Area | Temuan sebelum M13 | Prioritas |
|---|---|---|---|
| M13-F01 | SSH dan Fail2Ban | Log SSH M12 memiliki event invalid-user, sementara snapshot Fail2Ban M12 menunjukkan counter nol. Backend dan journal match perlu diperiksa. | Tinggi |
| M13-F02 | API credential | Token API pernah muncul pada catatan terminal. Service membaca token dari EnvironmentFile. | Tinggi |
| M13-F03 | MQTT | Autentikasi broker sudah diterapkan, tetapi bukti log sisi server dan validasi ulang diperlukan. | Sedang |
| M13-F04 | Security Onion | Snort dan pipeline sensor perlu diverifikasi, serta scan terbaru perlu direkam pada interface sensor. | Tinggi |
| M13-F05 | Korelasi alert | Event Sguil historis belum membuktikan deteksi scan terbaru. | Tinggi |
| M13-F06 | Firewall | UFW mengizinkan SSH dari Anywhere pada IPv4 dan IPv6. Aturan ini perlu ditinjau tanpa memutus akses administrasi NAT. | Sedang |

## 4. Rencana tindakan dan kriteria selesai

### 4.1 Evidence closure

- Salin raw log dan PCAP M12 dari Ubuntu ke repository Windows.
- Cocokkan hash PCAP M12 dengan nilai sumber.
- Buat manifest M12 dan simpan screenshot.
- Jangan mengubah bukti mentah setelah hash awal dibuat. Jika file dokumentasi berubah, buat manifest baru dan catat versi akhirnya.

**Kriteria:** PCAP sumber dan salinan memiliki SHA-256 yang sama, artefak mentah berada di repository, dan manifest final tersedia.

### 4.2 SSH dan Fail2Ban

- Periksa unit SSH aktual, backend Fail2Ban, filter, journal match, dan parameter jail.
- Simpan konfigurasi sebelum perubahan.
- Periksa hasil konfigurasi sesudah perubahan dan validasi sintaks.
- Uji key yang tidak terdaftar dan username tidak valid secara terbatas.
- Bedakan kegagalan autentikasi, event yang dihitung Fail2Ban, dan tindakan ban.

**Kriteria:** log SSH menunjukkan aktivitas uji; Fail2Ban menghitung event yang cocok; klaim ban hanya dibuat jika counter banned dan daftar IP banned membuktikannya.

### 4.3 API

- Identifikasi service API dan EnvironmentFile tanpa mencetak rahasia.
- Rotasi token yang terekspos.
- Uji request tanpa token, token yang tidak berlaku, token baru dengan ADC valid, dan token baru dengan ADC di luar rentang.
- Catat log service API untuk jendela uji.

**Kriteria:** respons HTTP dan log server cocok. Token tidak dimasukkan ke screenshot, Markdown, repository, atau riwayat terminal. Uji token lama harus memakai token lama yang sebenarnya jika masih tersedia secara aman. String dummy hanya membuktikan token acak ditolak.

### 4.4 MQTT

- Pastikan `allow_anonymous false` dan `password_file` diterapkan pada konfigurasi broker.
- Uji koneksi anonim.
- Uji publish dan subscribe dengan akun lab.
- Ambil log broker sisi server.

**Kriteria:** koneksi anonim ditolak dan publish-subscribe terautentikasi berhasil. Jangan menyimpulkan integrasi MQTT ke API atau MySQL tanpa bukti consumer.

### 4.5 Security Onion dan S1

- Pastikan interface sniffing `eth0` UP.
- Periksa status NSM, pcap agent, Snort, dan barnyard2.
- Jalankan tcpdump pada `eth0` saat Kali mengulang Nmap ke Target.
- Simpan PCAP, ringkasan paket, dan checksum.
- Periksa Sguil pada rentang waktu UTC yang sama.

**Kriteria:** penerimaan paket dibuktikan oleh PCAP. Deteksi Sguil hanya dinyatakan berhasil bila event yang sesuai timestamp, source, destination, dan port terlihat.

### 4.6 Final review

- Periksa UFW, listener port, service, Fail2Ban, dan bind MySQL.
- Catat aturan yang masih memiliki cakupan luas.
- Transfer bukti dari seluruh VM.
- Buat manifest M13 dan audit repository.

## 5. Risiko dan keputusan yang masih terbuka

1. Belum ada bukti pemblokiran IP Kali oleh Fail2Ban pada hasil yang tersedia.
2. Screenshot atau export Sguil untuk scan S1 tanggal 26 November belum ditemukan pada arsip yang diperiksa.
3. Pengujian token lama yang sebenarnya belum dibuktikan karena retest menggunakan nilai dummy.
4. Token pengganti sempat terekspos pada catatan sesi sehingga perlu rotasi ulang.
5. Aturan UFW SSH masih mengizinkan Anywhere. Pembatasan harus direncanakan hati-hati karena administrasi dilakukan melalui NAT port forwarding.
6. Password MQTT lab perlu diganti dengan kredensial kuat dan TLS sebelum penggunaan di luar simulasi.

## 6. Aturan pelaporan

Gunakan `Terbukti`, `Sebagian`, `Belum terverifikasi`, atau `Keterbatasan` sesuai bukti. Jangan mengubah hasil yang belum berhasil menjadi hasil sukses. Catat sumber, timestamp UTC, lokasi file, dan checksum untuk setiap bukti utama.
