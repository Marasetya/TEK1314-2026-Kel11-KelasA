# Indeks Bukti Digital Forensics and Incident Response Minggu 12

**Mata kuliah:** TEK1314 - Keamanan Siber  
**Kelompok/Kelas:** Kelompok 11, Kelas A  
**Proyek:** SOC BiliCheck, replika backend IoT  
**Periode investigasi:** 26 November 2026  
**Penanggung jawab Blue Team:** Muhammad Adam Ilyasa

## Tujuan

Folder ini menyimpan bukti mentah, ekspor monitoring, PCAP tambahan, screenshot, analisis, dan manifest integritas untuk kegiatan Digital Forensics and Incident Response Minggu 12.

## Struktur

- `raw/`: journal dan log asli hasil akuisisi Ubuntu Target.
- `raw/metadata/`: identitas sistem, waktu, interface, port, service, dan status Fail2Ban.
- `exports/historical/`: tempat PCAP historis dari Sguil jika berhasil ditemukan. Saat ini ketersediaannya belum terverifikasi.
- `exports/supplemental/`: PCAP tambahan tanggal 26 November 2026 dan hasil analisisnya.
- `screenshots/`: screenshot 01 sampai 07.
- `analysis/`: analisis forensik, dampak, dan tindakan respons.
- `hashes/`: manifest SHA-256 dan catatan verifikasi integritas.

## Batas interpretasi

Event Sguil tanggal 22 Oktober dan 4 November 2026 merupakan event historis. Belum ada bukti yang mengonfirmasi event Sguil pada rentang scan 25 November 2026. PCAP tambahan tanggal 26 November hanya membuktikan sesi uji konektivitas TCP SSH yang baru dibuat, bukan aktivitas historis tersebut.
