# LAPORAN MATA KULIAH PRAKTIKUM

# ADMINISTRASI JARINGAN

## **PRAKTIKUM 3: SMB - SERVER MESSAGE BLOCK**

![SMB Header Image](media/image11.png)

Dosen Pengampu:
Dr Ferry Astika Saputra ST, M.Sc

Dibuat oleh:
Muhammad Alif Aditya
3123600016
Teknik Informatika D4 A

**POLITEKNIK ELEKTRONIKA NEGERI SURABAYA**

## Apa itu SMB?

Server Message Block (SMB) adalah protokol jaringan yang memungkinkan pengguna untuk berbagi berkas, printer, dan sumber daya lainnya. Protokol ini juga dikenal sebagai Common Internet File System (CIFS).

## Cara kerja SMB

- SMB menggunakan metode permintaan-respons untuk berkomunikasi antara klien dan server
- SMB beroperasi pada lapisan aplikasi model OSI
- SMB menggunakan protokol NTLM atau Kerberos untuk autentikasi pengguna
- SMB menyediakan mekanisme komunikasi antarproses (IPC) yang diautentikasi

Samba adalah implementasi sisi server SMB pada sistem UNIX dan Linux. Menggunakan Samba itu sangat gampang Anda hanya menginstal satu paket di sisi server; tidak perlu software tambahan khusus di sisi Windows. Di dunia Windows, sistem berkas atau direktori yang tersedia melalui jaringan dikenal sebagai "share" Samba juga dapat mengimplementasikan berbagai layanan lintas platform lainnya selain berbagi berkas:

- Autentikasi & otorisasi
- Pencetakan jaringan
- Resolusi nama
- Pengumuman layanan (penemuan server berkas dan pencetak)

Sebagian besar fungsionalitas Samba diimplementasikan oleh dua daemon, smbd dan nmbd. smbd mengimplementasikan layanan berkas dan cetak serta autentikasi dan otorisasi. nmbd bertanggung jawab atas komponen SMB utama lainnya: name resolution and service announcement.

Tidak seperti NFS, yang memerlukan dukungan tingkat kernel, Samba tidak memerlukan driver atau modifikasi kernel dan berjalan sepenuhnya sebagai proses pengguna. Ia mengikat soket yang digunakan untuk permintaan SMB dan menunggu permintaan klien untuk mengakses sumber daya. Setelah permintaan diautentikasi, smbd membagi instans dirinya sendiri yang berjalan sebagai pengguna yang membuat permintaan. Hasilnya, semua izin akses file normal (termasuk izin grup) dipatuhi. Satu-satunya fungsi khusus yang ditambahkan smbd di atas ini adalah layanan penguncian file yang memberikan sistem Windows semantik penguncian yang biasa mereka gunakan. Dan hampir semua OS support samba. Berikut adalah cara-cara instalasi samba:

## 1. **Instalasi dan konfigurasi NTP**

Pertama yaitu dengan menginstal ntp yaitu network time protocol, yang berfungsi untuk mengubah protokol network menggunakan waktu indonesia

![NTP Installation](media/image3.png)

Command menginstal:
```
sudo apt-get update && sudo apt install ntp
```

Setelah instalasi selesai anda bisa mengubah configurasi ntp:
```
sudo nano /etc/ntpsec/ntp.conf
```

Setelah itu anda bisa scroll kebawah sampai menemukan konfigurasi server, anda bisa mengganti konfigurasi server menjadi berikut

![NTP Configuration](media/image1.png)

Klik ctrl + x lalu "y" setelah itu klik enter untuk keluar

## 2. **Instalasi dan Konfigurasi samba Limited/public Share**

Sebagian besar distribusi Linux menyertakan Samba secara default, termasuk laptop saya.

Namun jika belum ada, anda bisa menginstall dengan command
```
sudo apt install samba && sudo apt install smbclient
```

Anda bisa mengonfigurasi Samba dengan mengedit berkas `/etc/samba/smb.conf` (`/usr/local/etc/smb4.conf` pada FreeBSD). Berkas tersebut menentukan direktori yang akan dibagikan, hak aksesnya, dan parameter operasional umum Samba (ketik `testparm -v` untuk melihat semua opsi konfigurasi)

Penggunaan Samba yang paling umum adalah untuk berbagi berkas dengan klien Windows. Akses ke berbagi berkas ini harus diautentikasi melalui akun pengguna dengan salah satu dari dua opsi. Tambahkan konfigurasi berikut ke file `/etc/samba/smb.conf` untuk mengatur limited share dan public share:

![Samba Configuration 1](media/image4.png)

![Samba Configuration 2](media/image8.png)

## 3. **Membuat Direktori untuk Limited dan Public Share**

Pada langkah ini, kita akan membuat dua direktori, satu untuk berbagi terbatas (limited share) dan satu untuk berbagi publik (public share).

```
sudo mkdir -p sambashare
sudo mkdir -p sambapublic
```

Setelah direktori dibuat, kita perlu menetapkan izin akses yang sesuai.

![Creating Directories 1](media/image7.png)

![Creating Directories 2](media/image7.png)

## 4. **Memulai Samba Service**

Untuk memastikan Samba berjalan setelah konfigurasi dilakukan, jalankan perintah berikut:

```
sudo systemctl start smbd
sudo systemctl enable smbd
```

Perintah ini memastikan bahwa layanan Samba berjalan secara otomatis saat sistem dinyalakan.

![Start Samba Service](media/image6.png)

## 5. **Membuat Grup SMB dan Menambahkan Pengguna**

Kita perlu membuat grup khusus untuk pengguna yang akan memiliki akses terbatas.

```
sudo groupadd smbgroup
sudo usermod -aG smbgroup lfiathan
```

![Create SMB Group](media/image5.png)

Pengguna "lfiathan" sekarang menjadi anggota grup "smbgroup".

## 6. **Memeriksa Grup Pengguna dan Memberikan Kepemilikan**

Setelah menambahkan pengguna ke grup, pastikan bahwa kepemilikan direktori sudah benar.

```
sudo chown -R root:smbgroup /sambapublic
sudo chmod -R 770 /sambashare
```

Untuk folder public share, atur izin agar semua orang dapat mengaksesnya:

```
sudo chown -R nobody:nogroup /sambashare
sudo chmod -R 777 /sambapublic
```

![Set Permissions](media/image2.png)

## 7. **Result**

Untuk bisa mengecek apakah samba berhasil, anda bisa menggunakan vm, docker atau device teman kalian untuk mengakses folder yang telah anda buat dengan command berikut (untuk linux)

Untuk public:
```
smbclient //ipadress/sambapublic
```

Untuk limited share:
```
smbclient //ipaddress/sambashare -U lfiathan
```

Dan untuk windows, anda bisa menggunakan menu network files.

Public share:
![Public Share Access](media/image12.png)

Limited share:
![Limited Share Access 1](media/image9.png)

![Limited Share Access 2](media/image10.png)
