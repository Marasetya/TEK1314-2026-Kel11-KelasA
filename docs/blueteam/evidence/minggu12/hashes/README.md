# Integritas Bukti dan Manifest

`SHA256SUMS_M12.csv` dan `SHA256SUMS_M12.txt` dibuat oleh skrip PowerShell setelah file bukti dan dokumen Minggu 12 disalin ke direktori proyek.

File manifest menghitung seluruh file di folder `minggu12`, kecuali kedua file manifest itu sendiri dan `screenshots/07-sha256-manifest.png`. Screenshot 07 dikecualikan karena screenshot tersebut menampilkan manifest yang sedang dibuat. Memasukkan screenshot itu ke dalam manifest yang ditampilkannya akan membuat referensi hash melingkar.

`SHA256SUMS_UBUNTU_ACQUISITION.txt` adalah manifest awal yang dibuat pada Ubuntu. Manifest awal tersebut dibuat sebelum PCAP tambahan, sehingga tidak menggantikan manifest final Windows.

`PCAP_SHA256_VERIFICATION.md` membandingkan hash PCAP hasil salinan Windows dengan nilai SHA-256 yang diberikan oleh output Ubuntu Target.
