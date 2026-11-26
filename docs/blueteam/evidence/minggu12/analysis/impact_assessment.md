# Impact Assessment Minggu 12

## 1. Ringkasan penilaian

Berdasarkan bukti yang tersedia, aktivitas yang dapat dikonfirmasi adalah komunikasi jaringan dari Kali ke Ubuntu, event scan historis pada Sguil, percobaan menggunakan nama pengguna SSH yang tidak valid, serta sesi uji konektivitas SSH tambahan pada 26 November. Tidak ada bukti langsung bahwa host berhasil diambil alih.

| Area | Temuan berbasis bukti | Dampak yang dapat dinyatakan | Status |
|---|---|---|---|
| Confidentiality | Tidak ada bukti login tidak sah dari Kali yang berhasil atau eksfiltrasi data pada bukti yang dibagikan. | Tidak ada kebocoran kerahasiaan yang terbukti. Permukaan layanan SSH, API, dan MQTT tetap perlu dibatasi dan dipantau. | Tidak ada dampak terkonfirmasi |
| Integrity | Tidak ada bukti perubahan file, konfigurasi, atau data yang dilakukan tanpa izin pada periode yang ditinjau. | Tidak ada kerusakan integritas yang terbukti. | Tidak ada dampak terkonfirmasi |
| Availability | Port SSH merespons pada uji tambahan. Tidak ada bukti layanan berhenti atau mengalami denial of service. | Tidak ada gangguan ketersediaan yang terbukti. | Tidak ada dampak terkonfirmasi |
| Detection and monitoring | Sguil menampilkan event historis, tetapi korelasi 25 November belum terverifikasi. Fail2Ban snapshot M12 menunjukkan 0 failed dan 0 banned, meski log SSH berisi invalid user. | Ada potensi blind spot atau ketidakselarasan pencatatan deteksi. | Perlu investigasi |
| Application services | Bukti klien menunjukkan API 401, 202, 422 dan MQTT authenticated publish-subscribe. Bukti server-side belum lengkap. | Perilaku sisi klien terlihat, tetapi korelasi dan audit server belum menyeluruh. | Terbukti sebagian |

## 2. Tingkat risiko sementara

Klasifikasi sementara adalah aktivitas reconnaissance dan authentication probing dengan tingkat risiko operasional **sedang untuk ditindaklanjuti**, bukan tingkat kompromi yang telah terbukti. Penetapan ini didasarkan pada sumber internal lab yang melakukan scan dan percobaan nama pengguna tidak valid, serta kebutuhan memastikan monitoring dan Fail2Ban mencatat aktivitas tersebut.

## 3. Batas penilaian

1. Tidak ada bukti login tidak sah dari `192.168.11.100` yang berhasil pada potongan log yang diberikan.
2. Entri `Accepted publickey` berasal dari `10.0.2.2`, bukan Kali.
3. PCAP tambahan 26 November hanya merekam sesi uji TCP SSH yang baru dilakukan.
4. Event Sguil 22 Oktober dan 4 November tidak membuktikan deteksi untuk 25 November.
5. Log API server belum tersedia secara cukup untuk mencocokkan hasil 401, 202, dan 422 dari sisi klien.
6. Log MQTT telah dikumpulkan, tetapi belum ada isi lengkap yang ditinjau dalam bahan ini.

## 4. Kesimpulan dampak

Tidak ada dampak aktual terhadap kerahasiaan, integritas, atau ketersediaan yang dapat dinyatakan terbukti berdasarkan bukti yang tersedia. Temuan utama adalah aktivitas probing dan kebutuhan memperbaiki pembuktian deteksi, retensi PCAP, serta korelasi log antar-komponen.
