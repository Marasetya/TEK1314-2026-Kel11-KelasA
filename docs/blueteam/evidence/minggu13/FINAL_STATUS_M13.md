# Penutupan Final Minggu 13 - SOC BiliCheck

**Kelompok:** 11, Kelas A  
**Proyek:** Replika backend IoT BiliCheck  
**Tanggal finalisasi dokumen:** 2026-11-27 00:42:53 +07:00  
**Root dokumentasi:** `D:\Kuliah Semester 5\SIBER - Keamanan Siber\PJBL\Output\docs`

## 1. Keputusan penutupan

Seluruh pekerjaan dokumentasi dan evaluasi Minggu 13 ditetapkan **DITUTUP SECARA ADMINISTRATIF**. Penutupan ini berarti setiap skenario telah memiliki hasil, bukti yang tersedia telah dipetakan, dan keterbatasan telah diberi keputusan pelaporan. Status penutupan administratif **tidak berarti seluruh kontrol keamanan berhasil atau seluruh risiko teknis hilang**.

## 2. Hasil teknis final berdasarkan artefak yang tersedia

| Area | Hasil yang didukung bukti | Disposisi final |
|---|---|---|
| Evidence M12 | Transfer artefak dan kecocokan hash PCAP M12 tercatat pada log sebelumnya. | Ditutup - integritas transfer tercatat. |
| S1 Nmap dan sensor | PCAP Security Onion M13 tercatat berisi 343 frame IPv4 TCP, 42.654 byte, komunikasi dua arah Kali dan Ubuntu. | Ditutup - packet visibility terbukti. |
| S1 alert Sguil | Screenshot Sguil harus diperiksa berdasarkan waktu, source, destination, dan detail event. Kehadiran screenshot saja tidak membuktikan alert cocok. | Ditutup - hasil alert dibatasi pada bukti yang benar-benar terlihat. |
| S2A API | Hasil retest 401 tanpa token, 401 token dummy, 202 ADC valid, dan 422 ADC di luar rentang; log service sesuai. | Ditutup - perilaku endpoint teruji; token lama aktual tidak terbukti diuji. |
| S2B MQTT | Koneksi anonim ditolak (CONNACK 5); akun lab berhasil publish-subscribe (CONNACK 0). | Ditutup - autentikasi dan pertukaran pesan pada topic uji terbukti. |
| S3 SSH | Key tidak terdaftar ditolak; satu invalid-user event dihitung Fail2Ban. | Ditutup - penolakan dan pencatatan event terbukti; IP ban tidak terbukti. |
| Final posture | UFW aktif, API dan MQTT pada interface lab, MySQL localhost; SSH masih diizinkan dari Anywhere. | Ditutup - snapshot didokumentasikan; cakupan SSH menjadi rekomendasi hardening. |

## 3. Verifikasi integritas PCAP M13

- SHA-256 aktual: `72d1b237c66706675d775a77861f43df290920e90576afbcb88d5b90283d2383`
- SHA-256 sidecar: `72d1b237c66706675d775a77861f43df290920e90576afbcb88d5b90283d2383`
- Hasil perbandingan: **COCOK**

## 4. Audit kelengkapan artefak

- Item ditemukan: **25**
- Item tidak ditemukan: **1**
- Screenshot Sguil M13 ditemukan di folder final: **True**

Daftar per item tersedia pada `FINALIZATION_AUDIT_M13.csv`. Artefak yang tidak ditemukan tidak dibuat secara otomatis dan tidak boleh diganti dengan screenshot atau log rekayasa.

## 5. Catatan diagnosis jaringan terbaru

Pemeriksaan jaringan Kali yang dilakukan setelah sesi capture terdahulu menunjukkan neighbor ARP `192.168.11.5` berstatus `FAILED` pada interface `eth0`. Ini menunjukkan konektivitas layer-2 saat pemeriksaan terbaru belum berfungsi. Hasil ini **tidak menghapus PCAP historis** yang sudah tersimpan, tetapi pengujian live tambahan tidak boleh diklaim berhasil sampai konektivitas diperbaiki dan diuji ulang.

## 6. Risiko dan rekomendasi yang dicatat sebagai tindak lanjut

1. Periksa isi screenshot Sguil M13. Jika tidak ada event yang cocok, tulis sebagai hasil deteksi negatif atau tidak terverifikasi.
2. Rotasi ulang token API yang sempat terekspos. Jangan mencetak token ke terminal, screenshot, atau repository.
3. Pengujian token lama aktual tidak terbukti; hasil 401 pada string dummy hanya menunjukkan token dummy ditolak.
4. Fail2Ban menghitung invalid-user event, tetapi tidak ada bukti IP masuk daftar banned.
5. Kredensial MQTT adalah kredensial lab; perkuat sebelum penggunaan non-lab dan evaluasi TLS.
6. Tinjau rule SSH UFW yang mengizinkan Anywhere tanpa memutus jalur administrasi VirtualBox.
7. Perbaiki Internal Network VirtualBox bila pengujian live lanjutan diperlukan.

## 7. Kesimpulan

Sprint Minggu 13 ditutup secara administratif dengan hasil teknis yang terdokumentasi. API, MQTT, penolakan SSH key, pencatatan invalid-user oleh Fail2Ban, dan penerimaan paket oleh sensor memiliki bukti pada arsip. Keterbatasan seperti token lama aktual, pemblokiran IP Fail2Ban, korelasi alert Sguil, dan cakupan SSH dicatat secara eksplisit sebagai batas hasil dan rekomendasi. Laporan akhir harus mempertahankan pembedaan antara **pekerjaan ditutup**, **kontrol terbukti bekerja**, dan **risiko yang masih perlu diperkuat**.

---
Dibuat otomatis oleh Finalize-SOC-BiliCheck-M13.ps1. Dokumen sumber dan bukti mentah tidak diubah.