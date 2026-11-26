# PCAP Tambahan Minggu 12

Folder ini menyimpan capture tambahan yang dibuat pada 26 November 2026 dari interface `enp0s8` pada Ubuntu Target. Filter capture adalah host `192.168.11.100` dan TCP port `22`.

File yang diharapkan:

- `M12_SUPPLEMENTAL_20261126T024424Z.pcap`
- `pcap_analysis.txt`
- `pcap.sha256`

Capture ini merekam sesi uji konektivitas baru menggunakan `nc -vz -w 3 192.168.11.5 22`. Capture bukan bukti historis untuk event Sguil 22 Oktober 2026 atau scan 25 November 2026.

Hasil terminal yang tersedia menyatakan 9 paket ditangkap, 9 paket cocok dengan filter, dan 0 paket dropped oleh kernel. Timestamp paket pada output tcpdump tampil sebagai waktu lokal WIB sekitar `2026-11-26 09:45:11`, setara dengan `2026-11-26 02:45:11 UTC`.
