

<div align="center">
  
# AdminJaringan2025

## Analisis dan Penjelasan HTTP Capture  

dibuat oleh:

Nama  : Alif Aditya

NRP : 3123600016

Dosen Pengampu

Dr. Ferry Astika Saputra, S.T., M.Sc

GitHub:([@ferryastika](https://github.com/ferryastika))

**Workshop Administrasi Jaringan**

</div>


### 1. Analisis Versi HTTP  
HTTP (*Hypertext Transfer Protocol*) adalah protokol komunikasi yang digunakan untuk mentransfer data di web.  

**Langkah pengecekan**:  
- Klien mengirim permintaan HTTP dengan metode `GET`.  
- Permintaan dikirim ke server dengan versi `HTTP/1.1`.  
- Header permintaan berisi informasi tambahan seperti `Host`, `User-Agent`, dan `Accept`.  

Pada bagian ini, terlihat bahwa permintaan HTTP yang dikirim oleh klien menggunakan versi **HTTP/1.1**:  

![Analisis HTTP](https://github.com/user-attachments/assets/044ed21b-7d72-4f6a-82e1-997752c53fd5)  

```
GET /download.html HTTP/1.1  
Host: www.ethereal.com  
```

- **Metode HTTP**: `GET` digunakan untuk meminta sumber daya dari server.  
- **Versi HTTP**: `HTTP/1.1`, yang memungkinkan koneksi persisten secara default.  
- **Host**: Permintaan dikirim ke `www.ethereal.com`.  

### 2. Alamat IP dari Klien ke Server  
Alamat IP (*Internet Protocol*) adalah identitas numerik unik yang digunakan untuk mengidentifikasi perangkat dalam jaringan.  

**Langkah-langkah**:  
1. Klien dengan IP `145.254.160.237` mengirim permintaan ke server dengan IP `65.208.228.223`.  
2. IP ini membantu dalam mengarahkan data ke tujuan yang benar di jaringan.  

Pada bagian ini, terdapat informasi tentang alamat IP sumber dan tujuan:  

![Analisis IP](https://github.com/user-attachments/assets/a8d48443-4b3d-4e6a-9719-6a83ef691914)  

- **Alamat IP Sumber (Client)**: `145.254.160.237`  
- **Alamat IP Tujuan (Server)**: `65.208.228.223`  

### 3. Three-Way Handshake  
*Three-way handshake* adalah proses negosiasi tiga langkah dalam protokol TCP untuk membangun koneksi yang andal antara klien dan server.  

**Langkah-langkah**:  
1. **SYN**: Klien mengirim paket `SYN` untuk meminta koneksi ke server.  
2. **SYN-ACK**: Server merespons dengan `SYN-ACK` untuk mengonfirmasi penerimaan.  
3. **ACK**: Klien mengirim `ACK` untuk menyelesaikan koneksi.  

Proses *three-way handshake* terjadi sebelum pengiriman data:  

![Three-Way Handshake](https://github.com/user-attachments/assets/815cc70d-2df7-4fa0-ba38-d835e75ed692)  

1. **SYN** dari klien (`145.254.160.237`) ke server (`65.208.228.223`).  
2. **SYN-ACK** dari server (`65.208.228.223`) ke klien (`145.254.160.237`).  
3. **ACK** dari klien ke server untuk menyelesaikan koneksi.  

### 4. Waktu Klien Mengirim Permintaan  
Permintaan HTTP adalah pesan yang dikirim oleh klien ke server untuk mengambil sumber daya web tertentu.  

**Langkah-langkah**:  
1. Setelah koneksi TCP berhasil dibuat, klien mengirim permintaan `GET /download.html HTTP/1.1`.  
2. Permintaan berisi berbagai header HTTP seperti `User-Agent`, `Accept`, dan `Referer`.  
3. Server menerima permintaan dan memprosesnya untuk mengembalikan data yang diminta.  

Setelah koneksi terbentuk, klien mengirimkan permintaan HTTP dengan rincian berikut:  

![Permintaan HTTP](https://github.com/user-attachments/assets/d0736592-6924-429b-9520-c94b56f3e226)  

- **Header HTTP**: Mengandung informasi seperti `User-Agent`, `Accept`, `Accept-Encoding`, `Accept-Charset`, dan `Referer`.  
- **Panjang Data**: 479 byte.  

### 5. Waktu Server Menerima Permintaan HTTP dari Klien  
Respon HTTP adalah data yang dikirim oleh server sebagai tanggapan terhadap permintaan klien.  

**Langkah-langkah**:  
1. Server menerima permintaan HTTP dari klien.  
2. Server memproses permintaan dan mengirimkan respons dengan kode status **200 OK**.  
3. Respons berisi konten dalam format `text/html` dengan panjang 478 byte.  

Server menerima permintaan HTTP dan merespons dengan status **HTTP/1.1 200 OK**, yang menunjukkan bahwa permintaan berhasil diproses.  

![Respon HTTP](https://github.com/user-attachments/assets/dca4f50b-f513-42e3-9ca5-651a9d916a14)  

- **Waktu Transfer**: 4.846969 detik sejak awal komunikasi.  
- **Respons Server**: Berisi konten HTML (`text/html`).  
- **Ukuran Respons**: 478 byte.  

### 6. Waktu Transfer Data antara Klien dan Server  
Waktu transfer data adalah durasi yang diperlukan untuk mengirim dan menerima paket data antara klien dan server.  

**Langkah-langkah**:  
1. Klien mengirim permintaan HTTP dalam 0.911310 detik.  
2. Server mengirimkan respons dalam 4.846969 detik setelah menerima permintaan.  
3. Ukuran data yang ditransfer bervariasi antara 214 byte hingga 775 byte.  

Tabel berikut menunjukkan waktu dan ukuran paket yang dipertukarkan:  

![Waktu Transfer](https://github.com/user-attachments/assets/6b06987d-460e-40dc-91f3-6cdcdbd700b4)  

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

