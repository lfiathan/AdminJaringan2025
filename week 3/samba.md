# SMB - SERVER MESSAGE BLOCK

Dibuat oleh

Nama: Muhammad Alif Aditya

NRP: 3123600016

Kelas: Teknik Informatika D4 A

Dosen Pengampu

Dr. Ferry Astika Saputra, S.T., M.Sc

GitHub:([@ferryastika](https://github.com/ferryastika))
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

![image](https://github.com/user-attachments/assets/f664e231-b08b-42db-98c3-459d896e6f1f)


Command menginstal:
```
sudo apt-get update && sudo apt install ntp
```

Setelah instalasi selesai anda bisa mengubah configurasi ntp:
```
sudo nano /etc/ntpsec/ntp.conf
```

Setelah itu anda bisa scroll kebawah sampai menemukan konfigurasi server, anda bisa mengganti konfigurasi server menjadi berikut

![image](https://github.com/user-attachments/assets/3c5c57c1-e320-4001-86ed-f543f711c351)

Klik ctrl + x lalu "y" setelah itu klik enter untuk keluar

## 2. **Instalasi dan Konfigurasi samba Limited/public Share**

Sebagian besar distribusi Linux menyertakan Samba secara default, termasuk laptop saya.

Namun jika belum ada, anda bisa menginstall dengan command
```
sudo apt install samba && sudo apt install smbclient
```

Anda bisa mengonfigurasi Samba dengan mengedit berkas `/etc/samba/smb.conf` (`/usr/local/etc/smb4.conf` pada FreeBSD). Berkas tersebut menentukan direktori yang akan dibagikan, hak aksesnya, dan parameter operasional umum Samba (ketik `testparm -v` untuk melihat semua opsi konfigurasi)

Penggunaan Samba yang paling umum adalah untuk berbagi berkas dengan klien Windows. Akses ke berbagi berkas ini harus diautentikasi melalui akun pengguna dengan salah satu dari dua opsi. Tambahkan konfigurasi berikut ke file `/etc/samba/smb.conf` untuk mengatur limited share dan public share:
![image](https://github.com/user-attachments/assets/67bbf8d7-dd8f-4a45-86df-dd87de1dbe34)

![image](https://github.com/user-attachments/assets/c5f736a6-e623-4024-8965-da653591e55a)


## 3. **Membuat Direktori untuk Limited dan Public Share**

Pada langkah ini, kita akan membuat dua direktori, satu untuk berbagi terbatas (limited share) dan satu untuk berbagi publik (public share).

```
sudo mkdir -p sambashare
sudo mkdir -p sambapublic
```

Setelah direktori dibuat, kita perlu menetapkan izin akses yang sesuai.
![image](https://github.com/user-attachments/assets/efede50c-5d9c-44da-a19b-51247854e22f)


## 4. **Memulai Samba Service**

Untuk memastikan Samba berjalan setelah konfigurasi dilakukan, jalankan perintah berikut:

```
sudo systemctl start smbd
sudo systemctl enable smbd
```

Perintah ini memastikan bahwa layanan Samba berjalan secara otomatis saat sistem dinyalakan.

![image](https://github.com/user-attachments/assets/87db638b-1822-41fc-a555-8695d65bd3b1)


## 5. **Membuat Grup SMB dan Menambahkan Pengguna**

Kita perlu membuat grup khusus untuk pengguna yang akan memiliki akses terbatas.

```
sudo groupadd smbgroup
sudo usermod -aG smbgroup lfiathan
```
![image](https://github.com/user-attachments/assets/f73e9460-057b-4a43-a7b8-18d99c640844)


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
![image](https://github.com/user-attachments/assets/77aa1afb-eee4-4391-aaa9-87b9174c3570)


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
![image](https://github.com/user-attachments/assets/d17f7a9a-b112-4d7f-bd26-4ce971649ebe)


Limited share:
![image](https://github.com/user-attachments/assets/83ae7d27-768b-45e7-854e-6e8f46e12aa6)
![image](https://github.com/user-attachments/assets/e6305052-4ae3-47ff-af26-600730f1d0b2)



![Limited Share Access 2](media/image10.png)
