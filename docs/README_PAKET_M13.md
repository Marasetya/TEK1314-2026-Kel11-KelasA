# Paket Dokumentasi Minggu 13 SOC BiliCheck

Paket ini memuat dokumen final sprint Minggu 13 dan skrip PowerShell untuk merapikan screenshot serta membuat manifest SHA-256. Semua dokumen disusun dari bukti yang ada pada arsip `docs(3).zip` dan catatan terminal yang menyertainya.

## Isi paket

- `blueteam/evidence/minggu13/README_MINGGU13.md`
- `blueteam/evidence/minggu13/remediation_plan_minggu13.md`
- `blueteam/evidence/minggu13/remediation_log_minggu13.md`
- `blueteam/evidence/minggu13/retest_report_minggu13.md`
- `blueteam/evidence/minggu13/purple-team-validation_minggu13.md`
- `blueteam/evidence/minggu13/final_security_posture_minggu13.md`
- `blueteam/evidence/minggu13/finalization_checklist_minggu13.md`
- `blueteam/evidence/minggu13/incident_report_final_content_minggu13.md`
- `weekly-progress/minggu13.md`
- `Finalize-SOC-BiliCheck-M13.ps1` (skrip tunggal yang tertanam dengan seluruh dokumen Markdown)

## Cara memasukkan ke proyek

1. Simpan file `Finalize-SOC-BiliCheck-M13.ps1` di folder `D:\Kuliah Semester 5\SIBER - Keamanan Siber\PJBL\Output\docs`.
2. Buka PowerShell 7.
3. Jalankan skrip:

```powershell
pwsh -NoProfile -ExecutionPolicy Bypass -File "D:\Kuliah Semester 5\SIBER - Keamanan Siber\PJBL\Output\docs\Finalize-SOC-BiliCheck-M13.ps1" -DocsRoot "D:\Kuliah Semester 5\SIBER - Keamanan Siber\PJBL\Output\docs"
```

4. Skrip membuat semua dokumen Markdown yang tercantum di atas, memindahkan screenshot `m13-*.png` ke subfolder `screenshots`, mencari screenshot Sguil yang pengguna nyatakan sudah tersedia, memverifikasi hash PCAP, lalu membuat manifest CSV dan TXT.
5. Setelah skrip selesai, ambil screenshot terminal PowerShell dan simpan sebagai `m13-10-sha256-manifest.png` di folder `blueteam\evidence\minggu13\screenshots`.
6. Pastikan skrip menemukan `m13-08-sguil-retest-events.png`. Keberadaan file saja tidak menyatakan alert berhasil. Periksa gambar untuk memastikan apakah event S1 M13 terlihat.

## Batas klaim

- PCAP M13 membuktikan paket scan diterima sensor. Ini bukan bukti alert Sguil.
- Fail2Ban menghitung satu invalid-user event pada retest, tetapi tidak ada IP yang terbukti diblokir.
- API retest memakai token dummy pada pengujian yang diberi label token lama. Token lama aktual belum terbukti diuji.
- Token baru sempat terekspos pada catatan sesi dan perlu dirotasi ulang.
- MQTT authenticated pub-sub berhasil pada topic uji. Pengujian tidak membuktikan integrasi ke API atau MySQL.
