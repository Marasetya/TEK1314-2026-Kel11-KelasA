# Dokumentasi Minggu 13 - SOC BiliCheck

**Mata kuliah:** TEK1314 Keamanan Siber  
**Kelompok/Kelas:** Kelompok 11, Kelas A  
**Proyek:** SOC BiliCheck, replika keamanan backend IoT  
**Tanggal kegiatan:** 26 November 2026  
**Fokus:** Remediation, verification, retest, dan konsolidasi bukti.

## 1. Tujuan

Folder ini menyimpan bukti dan dokumentasi final sprint Minggu 13. Pekerjaan diarahkan pada perbaikan kontrol yang sudah ditemukan pada Minggu 11 dan Minggu 12, kemudian menguji ulang perilaku SSH, API, MQTT, dan monitoring jaringan.

Seluruh pengujian dilakukan pada replika laboratorium VirtualBox dan menggunakan data dummy. Dokumen ini tidak menyatakan bahwa sistem produksi BiliCheck atau perangkat sensor milik mitra telah diuji.

## 2. Struktur folder

| Folder | Isi |
|---|---|
| `baseline/` | Baseline Ubuntu Target dan materi baseline yang diekspor dari VM lain. |
| `blueteam/` | Diagnosis dan hasil remediation SSH, Fail2Ban, API, MQTT, serta posture akhir. |
| `redteam/` | Output Nmap, API retest, MQTT retest, SSH retest, dan identitas Kali. |
| `packet-capture/` | PCAP M13 dari interface sniffing Security Onion dan sidecar checksum. |
| `screenshots/` | Screenshot bukti M13. Jalankan `Finalize-SOC-BiliCheck-M13.ps1` dari root folder `docs` untuk merapikan screenshot dan membuat manifest. |
| `hashes/` | Manifest SHA-256 akhir. Dibuat oleh skrip finalisasi. |
| `purpleteam/` | Ruang untuk salinan atau lampiran evaluasi lintas tim. |

## 3. Bukti screenshot yang diketahui tersedia

| Nama | Isi | Status berdasarkan arsip yang diperiksa |
|---|---|---|
| `m13-01-baseline-before.png` | Baseline Ubuntu Target | Tersedia |
| `m13-02-fail2ban-diagnosis.png` | Diagnosis konfigurasi Fail2Ban | Tersedia |
| `m13-03-fail2ban-retest.png` | Invalid user tercatat, counter failed 1, banned 0 | Tersedia |
| `m13-04-api-token-retest.png` | Hasil API 401, 401, 202, 422 | Tersedia |
| `m13-05-mqtt-auth-retest.png` | Anonymous ditolak, authenticated pub-sub diterima | Tersedia |
| `m13-06-security-onion-sensor.png` | Status sensor Security Onion | Tersedia |
| `m13-07-pcap-nmap-correlation.png` | Capture paket Nmap dari interface sensor | Tersedia |
| `m13-08-sguil-retest-events.png` | Daftar RealTime Events Sguil terbaru | Pengguna mengonfirmasi screenshot tersedia di folder kerja, tetapi file tidak ikut dalam arsip yang dipakai saat penyusunan. Skrip finalisasi akan mencarinya di bawah `docs` dan menyalinnya ke folder `screenshots/`. |
| `m13-09-remediation-after.png` | Posture UFW, port, dan Fail2Ban setelah perubahan | Tersedia |
| `m13-10-sha256-manifest.png` | Manifest akhir M13 | Ambil setelah skrip finalisasi berhasil |

Jangan membuat screenshot pengganti untuk `m13-08`. Pengguna menyatakan screenshot tersebut sudah ada. Pastikan file dengan nama persis `m13-08-sguil-retest-events.png` dapat ditemukan oleh skrip. Isi screenshot tetap harus dibaca untuk menentukan apakah alert scan M13 benar-benar muncul. Jika tidak ada alert yang cocok, laporkan hasil pemeriksaan negatif secara jujur.

## 4. Catatan batas bukti

1. PCAP M13 membuktikan bahwa interface sniffing Security Onion menerima paket dari Kali ke Ubuntu pada saat Nmap dijalankan. PCAP tidak dengan sendirinya membuktikan alert Snort atau event Sguil berhasil dibuat.
2. Screenshot `m13-06` menunjukkan interface `eth0` UP serta `pcap_agent`, `snort_agent-1`, `snort-1`, dan `barnyard2-1` berstatus `[ OK ]`. Screenshot tersebut adalah bukti status sensor pada saat pengambilan, bukan bukti alert spesifik S1.
3. Hasil Fail2Ban retest memperlihatkan satu kegagalan terhitung. Tidak ada bukti IP Kali berhasil diblokir. Posture akhir juga menunjukkan `Total banned: 0`.
4. API retest memakai string dummy untuk uji token lama. Bukti ini tidak membuktikan bahwa token lama yang sebenarnya telah diuji ulang. Token pengganti juga sempat tampil dalam catatan sesi dan harus dirotasi lagi sebelum dinyatakan sebagai kredensial final.
5. MQTT anonymous ditolak dan authenticated pub-sub berhasil. Pengujian memakai kredensial lab. Kekuatan password dan penggunaan TLS tetap menjadi catatan penguatan untuk lingkungan non-lab.
6. Folder teks baseline terpisah dari Security Onion tidak ditemukan pada arsip M13 yang diperiksa. Screenshot `m13-06` tetap tersedia sebagai bukti visual status sensor, dan PCAP tersedia di `packet-capture/`.

## 5. Dokumen Minggu 13

- `remediation_plan_minggu13.md`
- `remediation_log_minggu13.md`
- `retest_report_minggu13.md`
- `purple-team-validation_minggu13.md`
- `final_security_posture_minggu13.md`
- `finalization_checklist_minggu13.md`
- `incident_report_final_content_minggu13.md`

## 6. Cara membuat manifest

Dari PowerShell Windows, jalankan:

```powershell
pwsh -NoProfile -ExecutionPolicy Bypass -File "<docs-root>\Finalize-SOC-BiliCheck-M13.ps1" -DocsRoot "<docs-root>"
```

Skrip tunggal membuat dokumen Markdown M13 dari konten yang tertanam, merapikan screenshot, mencari screenshot Sguil, memeriksa SHA-256 PCAP, dan membuat manifest CSV dan TXT. Screenshot manifest `m13-10-sha256-manifest.png` diambil setelah skrip selesai dan dikecualikan dari manifest untuk menghindari hash yang berubah karena manifest memuat hash dirinya sendiri.
