# Checklist Finalisasi Minggu 13 - SOC BiliCheck

Gunakan checklist ini setelah seluruh bukti tersedia. Centang hanya item yang sudah diverifikasi dari file aktual.

## A. Evidence M12

- [x] Raw log M12 dipindahkan ke repository Windows menurut arsip M12.
- [x] PCAP M12 dipindahkan dan hash sumber cocok pada output PowerShell.
- [x] Manifest M12 dibuat.
- [ ] Screenshot manifest M12 diverifikasi berada di folder `screenshots/`.
- [ ] `PCAP_SHA256_VERIFICATION.md` diperbarui dan manifest dihitung ulang setelah perubahan dokumentasi.

## B. Remediation dan retest M13

- [x] Baseline Ubuntu dan Kali tersimpan.
- [x] Screenshot baseline Security Onion tersedia.
- [x] Diagnosis Fail2Ban tersimpan.
- [x] Konfigurasi journalmatch diperbarui dan validasi konfigurasi sukses.
- [x] Log SSH retest menunjukkan key tidak terdaftar ditolak.
- [x] Event invalid-user M13 terlihat dan satu event dihitung Fail2Ban.
- [ ] Tindakan ban IP dibuktikan atau batas deteksi/action dijelaskan final.
- [x] API retest menghasilkan 401, 401, 202, 422.
- [ ] Rotasi ulang token API yang terekspos.
- [ ] Pengujian token lama aktual dilakukan atau keterbatasannya disetujui secara eksplisit.
- [x] MQTT anonymous ditolak.
- [x] MQTT authenticated pub-sub berhasil dan log broker tersedia.
- [ ] Password MQTT lab diganti dengan rahasia kuat jika lingkungan masih akan digunakan.
- [x] Sensor Security Onion berstatus OK pada screenshot.
- [x] PCAP Nmap M13 direkam langsung pada sensor.
- [x] Pengguna mengonfirmasi screenshot Sguil M13 tersedia; file perlu ditemukan dan ditempatkan oleh skrip finalisasi.
- [ ] Isi screenshot Sguil diperiksa untuk menentukan apakah alert scan M13 benar-benar muncul.
- [x] Final posture UFW, port, dan Fail2Ban tersimpan.

## C. Dokumentasi

- [x] Remediation plan M13.
- [x] Remediation log M13.
- [x] Retest report M13.
- [x] Purple Team validation M13.
- [x] Final security posture M13.
- [x] Mapping pengisian Incident Report Final.
- [ ] Weekly progress M13 direview oleh Lead.
- [ ] Root README diperbarui dengan tautan dokumen M13.
- [ ] Hasil review Lead, Red Team, dan Blue Team disepakati.

## D. Repository dan pengumpulan

- [ ] Screenshot dipindahkan ke `blueteam/evidence/minggu13/screenshots/`.
- [x] Screenshot `m13-08-sguil-retest-events.png` dikonfirmasi tersedia oleh pengguna.
- [ ] Pastikan skrip menemukan file tersebut dan menempatkannya di `screenshots/`.
- [ ] Baca isi screenshot dan catat apakah alert S1 M13 terlihat.
- [ ] Manifest SHA-256 M13 dibuat dan PCAP cocok dengan sidecar.
- [ ] Screenshot `m13-10-sha256-manifest.png` diambil setelah manifest selesai.
- [ ] Manifest dibuat ulang jika ada file yang berubah setelah perhitungan.
- [ ] `git status --short` diperiksa.
- [ ] `git diff --check` bersih.
- [ ] Tidak ada token, password, private key, atau EnvironmentFile di repository.
- [ ] Incident Report Final diisi dan ditinjau.
- [ ] Video demo 5 sampai 8 menit siap.
- [ ] Slide final PDF siap.
- [ ] Repository dapat diakses oleh dosen sesuai instruksi mata kuliah.

## Status penutupan

Jangan menyatakan proyek selesai hanya berdasarkan jumlah item yang dicentang. Proyek dapat ditutup jika setiap risiko terbuka sudah ditangani atau secara eksplisit dicatat dengan dampak dan rekomendasi, dan seluruh klaim laporan memiliki bukti pendukung.
