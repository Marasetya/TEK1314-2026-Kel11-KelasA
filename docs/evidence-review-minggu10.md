# Catatan Penataan dan Review Bukti Screenshot Minggu 10  -  BiliCheck

> **Keterangan file:** Daftar pemeriksaan isi, nama, lokasi, dan batas klaim seluruh bukti Minggu 10.
> **Isi utama:** Pemetaan screenshot, bukti lintas-folder, kekurangan bukti, dan catatan keamanan.
> **Penanggung jawab:** Lead Moh Rifky untuk integrasi. Pemilik bukti mengikuti pembagian Blue Team dan Red Team.
> **Status dokumen:** Dokumen review aktif.

Dokumen ini menjelaskan pemindahan dan batas interpretasi gambar yang namanya tidak sesuai isi atau bukti yang tersedia belum lengkap. Screenshot asli sebelum penataan tersimpan pada backup di luar folder `docs`.

## 1. Hasil pemetaan screenshot

| Nama awal di `docs/minggu10/` | Lokasi/nama baru | Catatan isi |
|---|---|---|
| `01-target-baseline-dan-service.png` | `blueteam/evidence/minggu10/01-target-baseline-service-ufw-api-mysql-sebagian.png` | Menunjukkan UFW aktif, API aktif, MySQL aktif. Belum menampilkan keseluruhan hostname/IP/route/time. |
| `01a-target-baseline-lanjutan.png` | `redteam/evidence/minggu10/01-kali-versi-tools-minggu10.png` | Isinya versi Nmap, cURL, Python, dan MQTT client pada Kali; bukan baseline lanjutan Target. |
| `02-hasil-uji-endpoint-api.png` | `redteam/evidence/minggu10/02-hasil-uji-endpoint-api-202-dan-422.png` | Memperlihatkan API `202` untuk ADC 2048 dan `422` untuk ADC 4096. Terdapat percobaan awal read prompt yang gagal; token tidak terlihat. |
| `07-uji-ulang-api-401-202-422.png` | `blueteam/evidence/application/07-uji-api-tanpa-token-401-dan-permintaan-lanjutan-belum-lengkap.png` | Menunjukkan health check dan HTTP 401. Bagian akhir pengujian lanjutan tidak terlihat lengkap. |
| Screenshot 06 rotasi token | Karantina di luar `docs` | Nilai token tampak pada gambar. Jangan di-commit. Rotasi token jika masih aktif. |
| `04-s2-publikasi-mqtt-baseline.png` | `redteam/evidence/minggu10/04-s2-publikasi-anonim-hasil-awal-ditolak.png` | Perintah subscriber/publisher berakhir dengan `Connection Refused: not authorised`; bukan bukti baseline menerima anonim. |
| `05-s2-publikasi-mqtt-anonim-ditolak.png` | `redteam/evidence/minggu10/05-s2-publikasi-anonim-ditolak-sebagian.png` | Menunjukkan penolakan anonim. Ada percobaan read prompt yang menghasilkan `read: -p: no coprocess`; gunakan sebagai bukti parsial. |
| `06-s2-publikasi-mqtt-akun-lab.png` | `redteam/evidence/minggu10/06-s2-publikasi-akun-lab.png` | Command publish menggunakan user `esp32_bilicheck`; tidak tampak error pada akhir screenshot. Bukti penerimaan server/subscriber masih perlu diperjelas. |
| `07-s3-ssh-auth-methods dan 08-s3-ssh-password-ditolak.png` | `redteam/evidence/minggu10/07-08-s3-ssh-auth-methods-dan-password-ditolak-gabungan.png` | Satu screenshot gabungan menampilkan metode publickey dan penolakan password; tidak dipisah menjadi dua gambar agar tidak menggandakan bukti. |
| `09-m10-elastic-sensor-dan-snort-status.png` | `blueteam/evidence/monitoring-seconion/09-m10-elastic-sensor-dan-snort-engine-perlu-verifikasi-proses.png` | Elastic dan status agent terlihat OK; `IDS_ENGINE_ENABLED=yes` tampil, tetapi proses Snort tidak terlihat jelas dan log menyebut file statistik tidak ditemukan. |
| `12-m10-snort-local-rules-check.png` | `blueteam/evidence/monitoring-seconion/12-m10-packet-capture-tambahan-bukan-local-rules.png` | Isi screenshot adalah packet capture, bukan local rules. |
| `13-m10-snort-config-validation.png` | `blueteam/evidence/monitoring-seconion/13-m10-snort-config-test-output-parsial.png` | Snort test mode mulai berjalan; output akhir/success summary tidak terlihat. Ada typo command awal `snor` sebelum `snort`. |
| `01-s1-purple-loop-baseline-detection.png` | `purple-team/evidence/minggu10/01-s2-purple-loop-api-mqtt.png` | Isi screenshot menunjukkan API 401/202/422 dan aktivitas MQTT; bukan S1 Nmap. |
| `02-s2-purple-loop-mqtt-api.png` | `purple-team/evidence/minggu10/02-s3-purple-loop-ssh-audit-awal.png` | Isi screenshot menunjukkan Nmap SSH auth methods, password ditolak, log SSH, dan Fail2Ban; bukan S2 MQTT/API. |
| `03-s3-purple-loop-ssh-audit.png` | `purple-team/evidence/minggu10/03-s3-purple-loop-ssh-audit-lanjutan.png` | Screenshot tindak lanjut SSH/Fail2Ban; daftar IP banned tetap 0. |
| `04-mitigasi-dan-verifikasi-ulang.png` | `purple-team/evidence/minggu10/04-kondisi-kontrol-setelah-mitigasi.png` | Memperlihatkan kondisi kontrol (UFW, MySQL bind, MQTT auth, SSH, permission env); tidak sendirinya membuktikan retest Red Team setelah perubahan. |
| `11-hasil-uji-mqtt-anonim-ditolak dan 12-hasil-uji-mqtt-akun-lab.png` | `blueteam/evidence/minggu10/06-mqtt-log-publish-client-anonymous-baseline.png` | Log menunjukkan client otomatis tanpa username yang jelas melakukan subscribe/publish; ini bukti aktivitas anonim baseline, bukan penolakan anonim atau authenticated user. |
| `13-log-mosquitto-minggu10, 06-mqtt-log-auth-failure, dan 07-mqtt-authenticated-publish.png` | `blueteam/evidence/minggu10/06a-mqtt-log-client-anonymous-baseline-tambahan.png` | Log memperlihatkan client `auto-*` terhubung dan melakukan publish. Tidak membuktikan log auth failure atau user terautentikasi. |
| `13-log-mosquitto-minggu10.png` | `blueteam/evidence/application/13-log-mosquitto-service-startup-saja.png` | Hanya menunjukkan service start/stop/restart dan load konfigurasi. |

## 2. Salinan lintas-folder

Beberapa gambar disalin ke lebih dari satu folder karena dipakai sebagai bukti dari perspektif tim yang berbeda:

- Konfigurasi autentikasi MQTT disalin dari `application/10` ke `blueteam/evidence/minggu10/05-mqtt-before-after-auth-config.png`.
- Hasil penolakan anonim dari Red Team disalin ke `application/11`.
- Hasil publish akun lab dari Red Team disalin ke `application/12` dan bukti Blue Team `07`.
- Hasil API 202/422 dari Red Team disalin ke `application/07b` sebagai pelengkap bukti 401.

## 3. Kekurangan bukti yang masih perlu dilengkapi

- [ ] Screenshot baseline Target lengkap: hostname, date, IP, route, UFW, port, API, MySQL, MQTT, Fail2Ban, disk, waktu.
- [ ] Screenshot IP Kali dan versi tool dalam satu baseline atau dua screenshot yang jelas.
- [ ] Screenshot rotasi token API yang aman, tanpa menampilkan nilai token.
- [ ] Uji API baru setelah token terakhir dirotasi: 401, 202, 422.
- [ ] Bukti log Mosquitto final yang jelas untuk anonymous rejected dan user `esp32_bilicheck` accepted.
- [ ] Bukti subscriber atau mekanisme lain yang mengonfirmasi payload publish diterima, jika requirement mengharuskannya.
- [ ] Verifikasi proses Snort yang benar-benar aktif dan output akhir validasi konfigurasi.
- [ ] Korelasi timestamp/port scan Nmap dengan event Sguil.
- [ ] Screenshot Purple Team khusus S1 dan bukti verifikasi ulang setelah mitigasi.

## 4. Catatan keamanan

Screenshot token yang dikarantina berada di folder `_PRIVATE_REVIEW_DO_NOT_COMMIT_BILICHECK` di luar `docs`. Jangan memindahkannya kembali ke repository. Bila screenshot tersebut menunjukkan token yang masih berlaku, rotasi token dan pastikan token lama tidak lagi diterima.

## 5. Revisi Peran dan Dokumen Historis

Pembagian peran yang berlaku mulai Minggu 10 adalah:
- Lead: Moh Rifky Marasetya.
- Red Team: Nayla Januarisa Prihadi dan Fauzan Nafis Jiyad Akbar.
- Blue Team: Muhammad Adam Ilyasa sebagai satu-satunya anggota Blue Team.

Seluruh pekerjaan Blue Team, termasuk Target Ubuntu dan Monitoring Security Onion, menjadi tanggung jawab Adam. Nayla tidak lagi dicatat sebagai Blue Team pada dokumen aktif Minggu 10.

### Catatan untuk presentasi Minggu 9

`docs/presentation-minggu9/slides.pdf` merupakan arsip presentasi tanggal 23 Oktober 2026. Slide OUR TEAM pada halaman 2 sudah menampilkan Moh Rifky sebagai Lead, Adam sebagai Blue Team, serta Nayla dan Fauzan sebagai Red Team. Slide Attack Plan pada halaman 9 adalah rencana historis Minggu 9. Hydra tercantum sebagai rencana pada slide tersebut, tetapi bukti Minggu 10 tidak menunjukkan Hydra dijalankan pada SSH utama. Jangan membaca isi slide rencana sebagai hasil aktual Minggu 10.

### Batas bukti

Nama berkas yang menyebut `sebagian`, `belum lengkap`, `perlu verifikasi`, atau `client-side` harus dipertahankan sesuai kondisi. Jangan mengganti nama atau statusnya menjadi `berhasil`, `terbukti`, atau `selesai` tanpa bukti tambahan.