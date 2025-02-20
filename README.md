# AdminJaringan2025
# Analisis dan Penjelasan HTTP Capture

## 1. Analisis Versi HTTP
Pada bagian ini, terlihat bahwa permintaan HTTP yang dikirim oleh klien menggunakan versi **HTTP/1.1**:<br>
![image](https://github.com/user-attachments/assets/044ed21b-7d72-4f6a-82e1-997752c53fd5)
```
GET /download.html HTTP/1.1
Host: www.ethereal.com
```
- **Metode HTTP**: `GET` digunakan untuk meminta sumber daya dari server.
- **Versi HTTP**: `HTTP/1.1`, yang memungkinkan koneksi persisten secara default.
- **Host**: Permintaan ini dikirim ke `www.ethereal.com`.

## 2. Alamat IP dari Klien ke Server
Pada bagian ini, terdapat informasi tentang alamat IP sumber dan tujuan:<br>
![image](https://github.com/user-attachments/assets/a8d48443-4b3d-4e6a-9719-6a83ef691914)
- **Sumber (Client IP)**: `145.254.160.237`
- **Tujuan (Server IP)**: `65.208.228.223`

## 3. Three-Way Handshake
Proses *three-way handshake* terjadi sebelum pengiriman data:<br>
![image](https://github.com/user-attachments/assets/815cc70d-2df7-4fa0-ba38-d835e75ed692)
1. **SYN** dari klien (`145.254.160.237`) ke server (`65.208.228.223`), menunjukkan permintaan koneksi.
2. **SYN-ACK** dari server (`65.208.228.223`) ke klien (`145.254.160.237`), mengonfirmasi permintaan.
3. **ACK** dari klien ke server untuk menyelesaikan koneksi.

## 4. Waktu Klien Mengirim Permintaan
Setelah koneksi terbentuk, klien mengirimkan permintaan HTTP dengan rincian berikut:<br>
![image](https://github.com/user-attachments/assets/d0736592-6924-429b-9520-c94b56f3e226)
- **Header HTTP**: Mengandung informasi seperti `User-Agent`, `Accept`, `Accept-Encoding`, `Accept-Charset`, dan `Referer`.
- **Panjang Data**: 479 byte.

## 5. Waktu Server Menerima Permintaan HTTP dari Klien
Server menerima permintaan HTTP dan merespons dengan status **HTTP/1.1 200 OK**, yang menunjukkan bahwa permintaan berhasil diproses.<br>
![image](https://github.com/user-attachments/assets/dca4f50b-f513-42e3-9ca5-651a9d916a14)
- **Waktu Transfer**: 4.846969 detik sejak awal komunikasi.
- **Respons Server**: Berisi konten HTML (`text/html`).
- **Ukuran Respons**: 478 byte.

## 6. Waktu Transfer Data antara Klien dan Server
Tabel berikut menunjukkan waktu dan ukuran paket yang dipertukarkan:<br>
![image](https://github.com/user-attachments/assets/6b06987d-460e-40dc-91f3-6cdcdbd700b4)

| No | Waktu | Sumber | Tujuan | Protokol | Ukuran |
|----|------------|----------------|----------------|---------|--------|
| 4  | 0.911310  | 145.254.160.237 | 65.208.228.223 | HTTP | 533 B |
| 18 | 2.948291  | 145.254.160.237 | 216.239.59.99  | HTTP | 775 B |
| 27 | 3.955678  | 216.239.59.99   | 145.254.160.237 | HTTP | 214 B |
| 38 | 4.846969  | 65.208.228.223  | 145.254.160.237 | HTTP | 478 B |

### Kesimpulan
Dari analisis ini, dapat disimpulkan bahwa:
- Proses komunikasi diawali dengan *three-way handshake* sebelum pengiriman data.
- Permintaan HTTP menggunakan versi **HTTP/1.1**.
- Server merespons dalam waktu yang wajar dengan status **200 OK**.
- Waktu transfer data menunjukkan bahwa ada latensi dalam komunikasi jaringan.

Analisis ini dapat digunakan untuk memahami alur komunikasi antara klien dan server serta mengidentifikasi kemungkinan kendala dalam koneksi jaringan.

