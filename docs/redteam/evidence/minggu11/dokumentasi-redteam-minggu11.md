# Dokumentasi Red Team Minggu 11

**Mata kuliah:** TEK1314 - Keamanan Siber  
**Kelompok/Kelas:** Kelompok 11, Kelas A  
**Proyek:** BiliCheck, replika backend IoT  
**Periode pengujian:** 25 November 2026  
**Penanggung jawab Red Team:** Nayla Januarisa Prihadi dan Fauzan Nafis Jiyad Akbar

## 1. Tujuan dan ruang lingkup

Kegiatan Red Team Minggu 11 memvalidasi perilaku API telemetry dan komunikasi MQTT pada Target replika BiliCheck. Pengujian dilakukan dari Kali Linux menuju Ubuntu Target di jaringan internal VirtualBox `bilicheck-lab`.

Seluruh pengujian menggunakan data dummy. Tidak ada pengujian terhadap backend produksi, Firebase asli, perangkat sensor fisik, jaringan kampus, atau sistem milik mitra. Token API dan password MQTT tidak dicantumkan dalam dokumentasi.

## 2. Lingkungan pengujian

| Komponen | Alamat atau identitas | Peran |
|---|---|---|
| Kali Linux | `192.168.11.100` | Attacker Node |
| Ubuntu Target | `192.168.11.5` | API BiliCheck dan MQTT Broker |
| API | `http://192.168.11.5:5000/api/telemetry` | Endpoint telemetry |
| MQTT Broker | `192.168.11.5:1883` | Broker publish-subscribe |

Konektivitas Kali ke Ubuntu diperiksa menggunakan ping dengan hasil 4 paket terkirim, 4 paket diterima, dan 0% packet loss.

## 3. Skenario S2A: Pengujian API BiliCheck

### 3.1 Tujuan dan parameter

Pengujian memeriksa tiga perilaku API:

1. Request tanpa token ditolak.
2. Request dengan token valid dan data sesuai diterima.
3. Nilai `rawAdc` di luar rentang ditolak.

Endpoint yang diuji:

```text
POST http://192.168.11.5:5000/api/telemetry
```

Payload valid menggunakan `deviceId=dev01` dan `rawAdc=2048`. Rentang `rawAdc` yang diuji adalah 0 sampai 4095.

### 3.2 Request tanpa token

```bash
curl -i -X POST http://192.168.11.5:5000/api/telemetry \
  -H 'Content-Type: application/json' \
  -d '{"deviceId":"dev01","rawAdc":2048}'
```

Hasil yang tercatat:

```text
HTTP/1.0 401 Unauthorized
{"error":"unauthorized"}
```

Interpretasi: API menolak request tanpa token autentikasi.

### 3.3 Request dengan token valid

Token dimasukkan secara lokal melalui input tersembunyi dan tidak dicantumkan dalam dokumen.

```zsh
read -rs 'API_TOKEN?Masukkan API Token: '
echo
```

Request:

```bash
curl -i -X POST http://192.168.11.5:5000/api/telemetry \
  -H "Authorization: Bearer ${API_TOKEN}" \
  -H 'Content-Type: application/json' \
  -d '{"deviceId":"dev01","rawAdc":2048}'
```

Hasil yang tercatat:

```text
HTTP/1.0 202 Accepted
```

Respons JSON menyatakan `accepted=true`, `validation=passed`, dan `stored=true`. Pernyataan `stored=true` merupakan isi respons aplikasi. Bukti pada bagian ini tidak digunakan sebagai pemeriksaan independen terhadap isi tabel MySQL.

### 3.4 Request dengan nilai ADC di luar rentang

```bash
curl -i -X POST http://192.168.11.5:5000/api/telemetry \
  -H "Authorization: Bearer ${API_TOKEN}" \
  -H 'Content-Type: application/json' \
  -d '{"deviceId":"dev01","rawAdc":4096}'
```

Hasil yang tercatat:

```text
HTTP/1.0 422 Unprocessable Entity
```

Respons JSON menunjukkan `rawAdc_out_of_range`, `allowed_min=0`, dan `allowed_max=4095`.

Setelah pengujian, hapus variabel sesi:

```bash
unset API_TOKEN
```

### 3.5 Ringkasan hasil API

| Waktu UTC | Pengujian | Hasil |
|---|---|---|
| `2026-11-25 03:33:35` | Tanpa token | HTTP `401` |
| `2026-11-25 03:35:04` | Token valid, `rawAdc=2048` | HTTP `202` |
| `2026-11-25 03:35:23` | Token valid, `rawAdc=4096` | HTTP `422` |

Tiga hasil tersebut mendukung perilaku autentikasi dan validasi aplikasi pada sesi pengujian. Catatan keamanan menyebut token pernah tercatat utuh dalam catatan terminal yang disampaikan saat dokumentasi. Perlakukan token tersebut sebagai rahasia yang berpotensi terekspos dan lakukan rotasi sebelum penggunaan lanjutan. Bukti yang ditinjau belum memastikan pengujian dilakukan setelah rotasi token final.

### 3.6 Bukti API

`docs/redteam/evidence/minggu11/02-m11-s2-api-401-202-422-final.png`

Screenshot memperlihatkan tiga respons HTTP dalam satu sesi. Nilai token tidak ditampilkan.

## 4. Skenario S2B: MQTT Authenticated Publish-Subscribe

### 4.1 Tujuan dan parameter

Pengujian memeriksa penolakan koneksi anonim, penerimaan autentikasi akun lab, subscription, serta penerimaan payload oleh subscriber.

| Parameter | Nilai |
|---|---|
| Broker | `192.168.11.5` |
| Port | `1883` |
| Username lab | `esp32_bilicheck` |
| Topic | `bilicheck/puskesmas01/dev-test/telemetry` |
| Device ID dummy | `BC-TEST-001` |
| `rawAdc` | `2048` |

Password digunakan secara lokal dan tidak disimpan dalam Markdown atau screenshot.

### 4.2 Hasil percobaan autentikasi awal

Pada percobaan tanpa kredensial, broker mengembalikan:

```text
CONNACK (5)
Connection Refused: not authorised
```

Konfigurasi broker yang diperiksa pada Target menunjukkan:

```text
listener 1883 192.168.11.5
allow_anonymous false
password_file /etc/mosquitto/passwd
```

Catatan pengujian menyebut akun `esp32_bilicheck` ditemukan pada password file. Percobaan awal yang ditolak tetap dicatat sebagai bagian dari hasil pengujian.

### 4.3 Subscription terautentikasi

Subscriber dijalankan lebih dahulu. Password dimasukkan secara tersembunyi pada sesi terminal.

```zsh
read -rs 'MQTT_PASS?Masukkan password MQTT: '
echo
timeout 120s mosquitto_sub -d \
  -h 192.168.11.5 -p 1883 \
  -t 'bilicheck/puskesmas01/dev-test/telemetry' \
  -u 'esp32_bilicheck' -P "$MQTT_PASS" \
  -C 1 -v
unset MQTT_PASS
```

Hasil subscriber menunjukkan:

```text
Client null received CONNACK (0)
Client null received SUBACK
Subscribed (mid: 1): 0
```

Interpretasi: broker menerima koneksi terautentikasi dan permintaan subscription diterima.

### 4.4 Publish payload dummy

Publisher dijalankan pada terminal Kali yang terpisah dengan password yang dimasukkan secara tersembunyi.

```zsh
read -rs 'MQTT_PASS?Masukkan password MQTT: '
echo
mosquitto_pub -d \
  -h 192.168.11.5 -p 1883 \
  -u 'esp32_bilicheck' -P "$MQTT_PASS" \
  -t 'bilicheck/puskesmas01/dev-test/telemetry' \
  -m '{"deviceId":"BC-TEST-001","rawAdc":2048}'
unset MQTT_PASS
```

Payload yang digunakan:

```json
{"deviceId":"BC-TEST-001","rawAdc":2048}
```

### 4.5 Hasil penerimaan subscriber

Output subscriber menunjukkan:

```text
Client null received CONNACK (0)
Client null received SUBACK
Subscribed (mid: 1): 0
Client null received PUBLISH (d0, q0, r0, m0, 'bilicheck/puskesmas01/dev-test/telemetry', ... (40 bytes))
bilicheck/puskesmas01/dev-test/telemetry {"deviceId":"BC-TEST-001","rawAdc":2048}
Client null sending DISCONNECT
```

Hasil tersebut membuktikan subscriber menerima payload dummy pada topic yang sama. Bukti ini terbatas pada komunikasi publisher ke subscriber di broker MQTT. Tidak ada klaim bahwa payload diteruskan ke API atau disimpan di MySQL karena integrasi consumer tidak dibuktikan dalam sesi ini.

### 4.6 Bukti MQTT

`docs/redteam/evidence/minggu11/03-m11-s2-mqtt-authenticated-pub-sub.png`

Screenshot utama memperlihatkan `CONNACK (0)`, `SUBACK`, `Received PUBLISH`, topic, dan payload. Kredensial tidak ditampilkan.

## 5. Ringkasan Hasil Red Team Minggu 11

| Area | Hasil aktual | Status |
|---|---|---|
| Konektivitas Kali ke Target | Ping 4/4, 0% packet loss | Terbukti |
| API tanpa token | HTTP 401 | Terbukti |
| API token valid dan payload sesuai | HTTP 202 | Terbukti pada sesi uji |
| API ADC di luar rentang | HTTP 422 | Terbukti |
| MQTT tanpa kredensial | `CONNACK (5)`, not authorised | Terbukti ditolak |
| MQTT dengan akun lab | `CONNACK (0)` | Terbukti |
| MQTT subscription | `SUBACK` | Terbukti |
| MQTT payload delivery | Subscriber menerima `BC-TEST-001` | Terbukti pada sisi klien |

## 6. Batasan dan tindak lanjut

1. Pastikan token API yang pernah terekspos telah dirotasi sebelum penggunaan lanjutan. Retest setelah rotasi harus dicatat terpisah dan tidak dianggap sudah selesai tanpa output baru.
2. Jangan menampilkan token API atau password MQTT di terminal yang direkam, screenshot, Markdown, atau command history.
3. Hasil MQTT membuktikan publish-subscribe pada broker, bukan integrasi consumer ke API atau MySQL.
4. Hasil Red Team disampaikan kepada Blue Team untuk diperiksa terhadap log Target dan telemetry Security Onion.
5. Retest pascamitigasi hanya dinyatakan selesai apabila skenario yang sama diulang dan outputnya disimpan.

## 7. Kesimpulan

Pengujian Red Team Minggu 11 berhasil memvalidasi perilaku autentikasi dan validasi API melalui hasil HTTP 401, 202, dan 422. Pengujian MQTT menunjukkan koneksi anonim ditolak, autentikasi akun lab diterima, subscription berhasil, dan subscriber menerima payload dummy `BC-TEST-001` pada topic yang ditentukan.

Seluruh pengujian dilakukan pada replika dan menggunakan data sintetis. Hasil yang tersedia mendukung perilaku layanan pada sisi klien. Retest API setelah rotasi token final dan verifikasi respons server yang lebih lengkap tetap menjadi tindak lanjut. Tidak ada klaim bahwa pesan MQTT diteruskan ke API atau MySQL.
