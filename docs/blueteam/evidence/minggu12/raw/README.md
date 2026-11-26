# Bukti Mentah Ubuntu Target

Folder ini menyimpan salinan file mentah yang dikumpulkan dari Ubuntu Target `target-bilicheck` (`192.168.11.5`). File di dalam folder ini dipertahankan sebagai salinan bukti. Jangan mengedit file mentah secara langsung.

## Isi utama yang diharapkan

- `system_journal.txt`: journal sistem untuk periode 18 sampai 25 November 2026.
- `ssh_events.txt`: journal service SSH dan sshd.
- `_var_log_auth.log`: salinan `/var/log/auth.log` jika tersedia.
- `fail2ban_events.txt`: journal Fail2Ban.
- `mqtt_events.txt`: journal Mosquitto.
- `apache_events.txt`: journal Apache. File ini tidak otomatis mewakili log aplikasi API.
- `metadata/`: identitas host, konfigurasi waktu, interface jaringan, port, service, dan status Fail2Ban.

Periksa ukuran dan isi setiap file sebelum menyatakan sumber log lengkap. File kosong atau sangat kecil harus dicatat sebagai keterbatasan, bukan diisi dengan perkiraan.
