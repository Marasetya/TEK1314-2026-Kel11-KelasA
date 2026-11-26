# Remediation Log Minggu 13 - SOC BiliCheck

**Tanggal utama:** 26 November 2026  
**Target:** `target-bilicheck`, `192.168.11.5`  
**Attacker:** `attacker-kali`, `192.168.11.100`  
**Monitoring:** Security Onion Classic, interface sniffing `eth0`

## 1. Ringkasan tindakan yang tercatat

| Waktu UTC | Area | Tindakan | Hasil yang didukung bukti |
|---|---|---|---|
| 06:31-06:32 | Baseline | Mengambil waktu, host, interface, route, port, UFW, service, dan status Fail2Ban pada Ubuntu serta identitas dan tool pada Kali. | Baseline Ubuntu dan Kali tersedia. Screenshot `m13-01` tersedia. |
| 08:34 | Fail2Ban | Backup `bilicheck.local`, menambahkan `journalmatch = _SYSTEMD_UNIT=ssh.service + _COMM=sshd`, validasi konfigurasi, dan restart service. | `fail2ban-client -t` menghasilkan `OK: configuration test is successful`. Restart berhasil setelah socket tersedia. |
| 08:35 | SSH | Lima percobaan dengan key tidak terdaftar dari Kali dan satu pengujian username tidak valid. | Key tidak terdaftar ditolak. Log menunjukkan lima koneksi pre-authentication ditutup dan satu event `Invalid user`. |
| 08:35 | Fail2Ban | Memeriksa journal dan status jail setelah invalid-user. | `Currently failed: 1`, `Total failed: 1`, `Currently banned: 0`, `Total banned: 0`. |
| 08:37 | API | Mengidentifikasi `bilicheck-api.service`, EnvironmentFile, dan permission. Merotasi variabel `BILICHECK_API_TOKEN`. | EnvironmentFile `root:root` mode `600`; service API kembali active. |
| 10:14 | API | Retest empat kondisi dari Kali. | HTTP 401, 401, 202, 422. Log service API mencatat empat request dengan respons yang sama. |
| 10:29 | MQTT | Menguji koneksi anonim dan authenticated pub-sub. | Anonymous ditolak dengan CONNACK 5. Publisher dan subscriber terautentikasi berhasil. |
| 10:29 | MQTT | Mengambil log broker Mosquitto. | File log server mencatat penolakan anonim, koneksi user lab, SUBSCRIBE, PUBLISH, dan delivery ke subscriber. |
| 10:37 | Security Onion | Memeriksa interface dan status proses sensor. | `eth0` UP; pcap_agent, snort_agent-1, snort-1, barnyard2-1 berstatus `[ OK ]` pada screenshot. |
| 11:57-11:59 | S1 | Menjalankan Nmap dari Kali sambil tcpdump merekam pada `eth0` Security Onion. | PCAP M13 tersedia, 42.654 byte, 343 frame IPv4 TCP pada pemeriksaan file, hash tersedia. |
| 12:09 | Final posture | Mengambil status UFW, socket listening, dan jail Fail2Ban. | UFW active; port API dan MQTT terbatas sesuai aturan yang terlihat; MySQL bind localhost; snapshot Fail2Ban menunjukkan counter nol dan tidak ada IP banned. |

Waktu sesi di atas disalin dari timestamp output yang tersedia. Perbedaan zona waktu tampilan Nmap dan timestamp log diselaraskan menggunakan waktu UTC yang dicatat oleh Kali dan PCAP.

## 2. Detail perubahan Fail2Ban

File konfigurasi yang diubah:

`/etc/fail2ban/jail.d/bilicheck.local`

Diff yang tersimpan menunjukkan penambahan:

```ini
journalmatch = _SYSTEMD_UNIT=ssh.service + _COMM=sshd
```

Parameter jail yang terlihat pada diagnosis:

| Parameter | Nilai |
|---|---:|
| backend | `systemd` |
| maxretry | 5 |
| findtime | 600 detik |
| bantime | 3600 detik |

Catatan interpretasi: setelah perubahan, event `Invalid user invalid_m13_test` terlihat pada journal dan status jail menunjukkan satu kegagalan terhitung. Lima percobaan key tidak terdaftar menghasilkan pesan `Connection closed ... [preauth]` dan tidak menaikkan counter pada output yang tersedia. Ini menunjukkan perbedaan pola log antara key yang tidak cocok dan username tidak valid. Bukti yang tersedia belum menunjukkan IP masuk daftar banned.

## 3. Detail rotasi API

Service yang berjalan:

- Unit: `bilicheck-api.service`
- Program: `/opt/bilicheck-api/app.py`
- Port: `192.168.11.5:5000`
- Environment variable: `BILICHECK_API_TOKEN`
- EnvironmentFile: `/etc/bilicheck-api/bilicheck-api.env`
- Permission saat pemeriksaan: `root:root 600`

Service aktif kembali setelah rotasi. Output Kali dan log service API memperlihatkan urutan respons yang konsisten: 401, 401, 202, 422.

Batas bukti: permintaan dengan label `Old token (revoked)` pada retest menggunakan string dummy, bukan nilai token lama M11 yang sebenarnya. Karena itu, dokumen ini tidak menyatakan bahwa token lama aktual sudah diuji langsung. Selain itu, token baru sempat tampil dalam catatan sesi. Token tersebut tidak boleh dipakai sebagai kredensial final dan perlu dirotasi lagi. Jangan menaruh nilai token pada dokumen atau screenshot.

## 4. Detail MQTT

Konfigurasi yang diperiksa pada `/etc/mosquitto/conf.d/bilicheck-lab.conf`:

```ini
listener 1883 192.168.11.5
allow_anonymous false
password_file /etc/mosquitto/passwd
```

Log broker menunjukkan:

- Koneksi tanpa kredensial menerima CONNACK `(0, 5)` dan ditutup karena tidak berwenang.
- Koneksi dengan user `esp32_bilicheck` menerima CONNACK `(0, 0)`.
- Broker menerima SUBSCRIBE dan PUBLISH pada topic uji.
- Pesan diteruskan kepada subscriber.

Bukti hanya mendukung autentikasi broker dan pertukaran pesan pada topic uji. Integrasi end-to-end ke API atau MySQL belum dibuktikan. Password lab yang digunakan pada pengujian tidak boleh dianggap sesuai untuk produksi.

## 5. Detail S1 dan PCAP Security Onion

| Parameter | Nilai |
|---|---|
| File | `packet-capture/m13-s1-nmap-retest.pcap` |
| Interface capture | `eth0` Security Onion |
| Source | `192.168.11.100` |
| Destination | `192.168.11.5` |
| Waktu frame pertama | 2026-11-26 11:57:47.493893 UTC |
| Waktu frame terakhir | 2026-11-26 11:59:21.321134 UTC |
| Ukuran file | 42.654 byte |
| Jumlah frame pada pemeriksaan file | 343 |
| Protokol IPv4 pada file | TCP |
| SHA-256 | `72d1b237c66706675d775a77861f43df290920e90576afbcb88d5b90283d2383` |

Nmap dijalankan pada `2026-11-26T11:57:45+00:00` sampai `11:59:20+00:00`. Rentang ini beririsan dengan timestamp paket. PCAP menunjukkan komunikasi dua arah antara Kali dan Ubuntu pada port yang diuji.

Hasil Nmap:

| Port | Hasil | Identifikasi yang terlihat |
|---|---|---|
| 22/tcp | open | OpenSSH 9.6p1 |
| 80/tcp | filtered | Tidak ada respons pada scan |
| 1883/tcp | open | MQTT |
| 3306/tcp | filtered | Tidak dapat dijangkau dari Kali pada scan |
| 5000/tcp | open | Fingerprint HTTP `BiliCheckLabAPI/1.0` |
| 8080/tcp | filtered | Tidak ada respons pada scan |
| 9099/tcp | filtered | Tidak ada respons pada scan |

PCAP membuktikan sensor menerima trafik scan. Screenshot Sguil `m13-08-sguil-retest-events.png` tidak ada dalam arsip yang diperiksa, sehingga event alert Sguil yang cocok dengan scan ini belum terverifikasi.

## 6. Final posture yang tercatat

- UFW: active, default incoming deny, outgoing allow.
- SSH: port 22 diizinkan dari Anywhere pada IPv4 dan IPv6.
- API: port 5000 diizinkan dari `192.168.11.0/24`.
- MQTT: port 1883 diizinkan dari `192.168.11.100`.
- MySQL: listener `127.0.0.1:3306`.
- API listener: `192.168.11.5:5000`.
- MQTT listener: `192.168.11.5:1883`.
- Fail2Ban pada snapshot akhir: `Currently failed: 0`, `Total failed: 0`, `Currently banned: 0`, `Total banned: 0`.

Snapshot akhir tidak membuktikan adanya IP yang diblokir. Aturan SSH Anywhere perlu ditinjau pada tahap hardening lanjutan dengan mempertahankan jalur administrasi VirtualBox.

## 7. Artefak pendukung

- `baseline/final_posture_after.txt`
- `blueteam/f2b_diagnosis_before.txt`
- `blueteam/f2b_status_retest.txt`
- `blueteam/ssh_journal_retest.txt`
- `blueteam/api_journal_retest.txt`
- `blueteam/mosquitto_filelog_retest.txt`
- `redteam/s3_ssh_retest.txt`
- `redteam/s2a_summary.txt`
- `redteam/mqtt_retest/s2b_mqtt_retest.txt`
- `redteam/m13-s1-nmap-retest.txt`
- `packet-capture/m13-s1-nmap-retest.pcap`
- `packet-capture/m13-s1-nmap-retest.pcap.sha256`
