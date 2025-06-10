# Filesystem

Dibuat oleh

Muhammad Alif Aditya / 3123600016

Teknik Informatika D4 A

Dosen Pengampu

Dr. Ferry Astika Saputra, S.T., M.Sc

GitHub:([@ferryastika](https://github.com/ferryastika))
![filesystem-icon](https://miro.medium.com/v2/resize:fit:752/1*quw0WvsLLCxad3WC6fjQ1Q.png)

Tujuan dasar dari sebuah filesystem adalah untuk merepresentasikan dan mengorganisasi sumber daya penyimpanan sistem.

Filesystem dapat dianggap terdiri dari empat komponen utama:

1. Namespace - cara untuk menamai hal-hal dan mengaturnya dalam hierarki
2. API - serangkaian system call untuk menavigasi dan memanipulasi objek
3. Model keamanan - skema untuk melindungi, menyembunyikan, dan berbagi hal-hal
4. Implementasi - perangkat lunak untuk menghubungkan model logis ke perangkat keras

Filesystem berbasis disk yang dominan adalah filesystem ext4, XFS, dan UFS, bersama dengan ZFS dari Oracle dan Btrfs. Banyak lainnya juga tersedia, termasuk VxFS dari Verittas dan JFS dari IBM.

Kita juga memiliki beberapa filesystem asing seperti FAT dan NTFS yang digunakan oleh Windows dan filesystem ISO 9660 yang digunakan oleh CD dan DVD.

Sebagian besar filesystem modern berusaha mengimplementasikan fungsionalitas filesystem tradisional dengan cara yang lebih cepat dan lebih andal, atau menambahkan fitur ekstra sebagai lapisan di atas semantik filesystem standar.

## Pathnames

Kata "folder" hanyalah kebocoran linguistik dari dunia Windows dan macOS. Artinya sama dengan **directory**, yang lebih teknis. Jangan gunakan istilah "folder" dalam konteks teknis kecuali jika Anda siap menerima tatapan aneh!!

Daftar direktori yang mengarah ke sebuah file disebut **pathname**. Pathname adalah string yang menggambarkan lokasi file dalam hierarki filesystem.
Pathname bisa berupa **absolute** (misalnya `/home/username/file.txt`) atau **relative** (misalnya `./file.txt`).

## Mounting dan Unmounting Filesystem

Filesystem terdiri dari potongan-potongan kecil--juga disebut "filesystem"--yang masing-masing terdiri dari satu direktori dan subdirektori serta file-filenya. Kita menggunakan istilah **file tree** untuk merujuk pada tata letak keseluruhan dan menggunakan kata **filesystem** untuk cabang-cabang yang terpasang pada pohon.

Dalam kebanyakan situasi, filesystem dipasang ke pohon dengan perintah `mount`. Perintah `mount` memetakan sebuah direktori dalam file tree yang ada, disebut **mount point**, ke root filesystem baru.

Contoh:

```bash
# Mount the filesystem on /dev/sda4 to /users
mount /dev/sda4 /users
```

Linux memiliki opsi unmount malas (**umount -l**) yang menghapus filesystem dari hierarki penamaan tetapi tidak benar-benar melepasnya sampai tidak lagi digunakan.

**umount -f** adalah unmount paksa, yang berguna ketika filesystem sedang sibuk.

Daripada menggunakan **umount -f**, Anda dapat menggunakan **lsof** atau **fuser** untuk mencari tahu proses mana yang menggunakan filesystem dan kemudian menghentikannya.

Contoh:
![image](https://github.com/user-attachments/assets/8c523ed8-b2f7-4900-b5f9-e5b88c423018)


Untuk menyelidiki proses yang menggunakan filesystem, Anda bisa menggunakan perintah **ps**.

Contoh:

![image](https://github.com/user-attachments/assets/4c5b9eae-4a29-4b0e-bb1a-7e125bf6e12a)

## Pengorganisasian file tree

Sistem UNIX tidak pernah terorganisir dengan baik! Berbagai konvensi penamaan yang tidak kompatibel digunakan secara bersamaan, dan berbagai jenis file tersebar secara acak di seluruh namespace. **Itulah mengapa sulit untuk meningkatkan sistem operasi**.

Root filesystem mencakup setidaknya direktori root dan sekumpulan minimal file dan subdirektori. File yang berisi kernel OS biasanya berada di **/boot**, tetapi nama dan lokasinya yang tepat dapat bervariasi. Di bawah BSD dan beberapa sistem UNIX lainnya, kernel tidak benar-benar merupakan file tunggal tetapi lebih seperti sekumpulan komponen.

**/etc** berisi file sistem dan konfigurasi yang kritis. **/sbin** dan **/bin** untuk utilitas penting, dan terkadang **/tmp** untuk file sementara. **/dev** secara tradisional adalah bagian dari root filesystem, tetapi saat ini merupakan filesystem virtual yang dipasang secara terpisah.

Beberapa sistem menyimpan file library bersama dan beberapa hal aneh lainnya, seperti preprosesor C, di direktori **/lib** atau **/lib64**. Yang lain telah memindahkan item-item ini ke **/usr/lib**, terkadang meninggalkan **/lib** sebagai link simbolis.

**/usr** dan **/var** juga sangat penting. **/usr** adalah tempat di mana sebagian besar program standar namun tidak kritis untuk sistem disimpan, bersama dengan berbagai hal lain seperti manual online dan sebagian besar library. FreeBSD menyimpan banyak konfigurasi lokal di bawah **/usr/local**. **/var** menampung direktori spool, file log, informasi akuntansi, dan berbagai item lain yang tumbuh atau berubah dengan cepat dan bervariasi pada setiap host. Baik **/usr** dan **/var** harus tersedia agar sistem dapat naik sepenuhnya ke mode multi-pengguna.
![image](https://github.com/user-attachments/assets/7c5b43d6-a830-47da-ab0e-a12f94f62e5d)


## Tipe file

Sebagian besar implementasi filesystem mendefinisikan tujuh jenis file:

1. File reguler
2. Direktori
3. File perangkat karakter
4. File perangkat blok
5. Socket domain lokal
6. Named pipes (FIFO)
7. Link simbolis

Anda dapat menentukan jenis file dengan menggunakan perintah **file** (ketik **man file** untuk informasi lebih lanjut).

![image](https://github.com/user-attachments/assets/fa2378b2-ac80-4e88-8d51-14a60ce6473b)


Anda juga dapat menggunakan **ls -ld**, flag **-d** memaksa **ls** untuk menampilkan informasi tentang direktori daripada menampilkan isi direktori.

![image](https://github.com/user-attachments/assets/1f1d10d8-3cbb-4f5a-be26-944887bbafda)


**File reguler** terdiri dari serangkaian byte; filesystem tidak memaksakan struktur pada isinya. File teks, file data, program yang dapat dieksekusi, dan library bersama semuanya disimpan sebagai file reguler.

**Direktori** adalah referensi bernama ke file lain.

**Hard links** adalah cara untuk memberikan nama ganda pada satu file. Perintah **ln** membuat hard link baru ke file yang ada. Opsi **-i** pada **ls** menyebabkannya menampilkan jumlah hard link ke setiap file.

Contoh

```bash
$ ln /etc/passwd /tmp/passwd
```

**File perangkat karakter dan blok**

File perangkat memungkinkan program berkomunikasi dengan perangkat keras dan periferal sistem. Kernel menyertakan (atau memuat) software driver untuk setiap perangkat sistem. Software ini mengurus detail yang rumit dalam mengelola setiap perangkat sehingga kernel itu sendiri dapat tetap relatif abstrak dan independen dari perangkat keras.

Perbedaan antara perangkat karakter dan blok itu halus dan tidak perlu ditinjau secara detail.

File perangkat ditandai oleh nomor perangkat utama dan kecil mereka. Nomor utama mengidentifikasi driver yang mengontrol perangkat, dan nomor kecil biasanya memberi tahu driver unit fisik mana yang harus dialamatkan. Misalnya, nomor perangkat utama 4 pada sistem Linux menunjukkan driver serial. Port serial pertama (**/dev/tty0**) akan memiliki nomor perangkat utama 4 dan nomor perangkat kecil 0, port serial kedua (**/dev/tty1**) akan memiliki nomor perangkat utama 4 dan nomor perangkat kecil 1, dan seterusnya.

Di masa lalu, **/dev** adalah direktori generik dan perangkat dibuat dengan **mknod** dan dihapus dengan **rm**. Sayangnya, sistem yang kasar ini tidak siap untuk menangani lautan driver dan jenis perangkat yang tak ada habisnya yang telah muncul selama beberapa dekade terakhir. Ini juga memfasilitasi semua jenis potensi ketidakcocokan konfigurasi: file perangkat yang tidak merujuk ke perangkat aktual, perangkat yang tidak dapat diakses karena tidak memiliki file perangkat, dan sebagainya.

Saat ini, direktori **/dev** biasanya dipasang sebagai jenis filesystem khusus, dan isinya otomatis dikelola oleh kernel bersama dengan daemon tingkat pengguna.

**Socket domain lokal** adalah cara bagi proses untuk berkomunikasi satu sama lain. Mereka mirip dengan socket jaringan, tetapi terbatas pada host lokal.

Syslog dan X Window System adalah contoh fasilitas standar yang menggunakan socket domain lokal.

**Named pipes** seperti socket domain lokal memungkinkan proses yang berjalan untuk berkomunikasi satu sama lain dalam host yang sama.

**Link simbolis** atau soft link menunjuk ke file berdasarkan nama. Mereka adalah cara untuk memberikan beberapa nama pada satu file, tetapi lebih fleksibel daripada hard link. Mereka dapat menunjuk ke file di filesystem yang berbeda, dan mereka dapat menunjuk ke direktori.

Misalnya, direktori **/usr/bin** sering kali merupakan link simbolis ke **/bin**. Ini adalah cara untuk menjaga agar root filesystem tetap kecil dan untuk memudahkan berbagi software yang sama di antara beberapa host.

```bash
$ ln -s /bin /usr/bin

$ ls -l /usr/bin
lrwxrwxrwx 1 root root 4 Mar  1  2020 /usr/bin -> /bin
```

## Atribut file

Di bawah model filesystem Unix dan Linux, setiap file memiliki sembilan bit izin, yang menentukan siapa yang dapat membaca, menulis, dan mengeksekusi file. Bersama dengan tiga bit lain yang terutama memengaruhi operasi program yang dapat dieksekusi, bit-bit ini membentuk mode file.

Dua belas bit mode ini disimpan bersama dengan empat bit informasi tipe file. Empat bit tipe file ditetapkan ketika file dibuat dan tidak dapat diubah, tetapi pemilik file dan superuser dapat memodifikasi dua belas bit mode dengan perintah **chmod**.

![file-attributes](https://cdn.storyasset.link/nlFtWFR5rySdmletT0jhDUQ0tXl2/ms-yxhcfoletf.jpg)

### Bit izin

Bit izin dibagi menjadi tiga kelompok masing-masing tiga bit. Kelompok pertama dari tiga bit adalah untuk pemilik file, kelompok kedua adalah untuk grup file, dan kelompok ketiga adalah untuk semua orang lain.

Anda dapat menggunakan nama **Hugo** untuk mengingat urutan kelompok: **u** untuk pemilik (user/owner), **g** untuk grup, dan **o** untuk orang lain (others).

Dimungkinkan juga untuk menggunakan notasi oktal (basis 8) karena setiap digit dalam notasi oktal mewakili tiga bit. Tiga bit teratas (dengan nilai oktal 400, 200, dan 100) mewakili pemilik file, tiga bit tengah (dengan nilai oktal 40, 20, dan 10) mewakili grup file, dan tiga bit terbawah (dengan nilai oktal 4, 2, dan 1) mewakili semua orang lain.

Pada file reguler, bit baca memungkinkan file untuk dibaca, bit tulis memungkinkan file untuk dimodifikasi atau dipotong; namun, kemampuan untuk menghapus atau mengganti nama (atau menghapus dan kemudian membuat ulang!) file dikendalikan oleh izin pada direktori induknya, di mana pemetaan nama-ke-ruang data dipertahankan.

Bit eksekusi memungkinkan file untuk dieksekusi. Ada dua jenis file yang dapat dieksekusi: biner, yang dijalankan langsung oleh CPU, dan skrip, yang harus diinterpretasikan oleh program seperti shell atau Python. Secara konvensi, skrip dimulai dengan baris **shebang** yang memberi tahu kernel interpreter mana yang akan digunakan.

```bash
#!/usr/bin/perl
```

File yang dapat dieksekusi non-biner yang tidak menentukan interpreter diasumsikan sebagai skrip **sh**.

Kernel memahami sintaks *#!* (shebang) dan bertindak langsung. Namun, jika interpreter tidak ditentukan secara lengkap dan benar, kernel akan menolak file tersebut. Kemudian shell mengambil alih dan mencoba menginterpretasikan file sebagai skrip shell.

Untuk direktori, bit eksekusi (sering disebut bit pencarian atau pemindaian) memungkinkan direktori untuk dimasuki atau dilalui saat pathname dievaluasi, tetapi tidak memungkinkan isinya dicantumkan. Kombinasi bit baca dan eksekusi memungkinkan direktori untuk dibaca dan isinya dicantumkan. Kombinasi bit tulis dan eksekusi memungkinkan file dibuat, dihapus, dan diganti namanya di dalam direktori.

### Bit setuid dan setgid

Bit dengan nilai oktal 4000 dan 2000 masing-masing adalah bit setuid dan setgid. Ketika bit setuid diatur pada file, pemilik file sementara diubah menjadi pemilik file ketika file dieksekusi. Ketika bit setgid diatur pada file, grup file sementara diubah menjadi grup file ketika file dieksekusi.

Ketika diatur pada direktori, bit setgid menyebabkan file yang baru dibuat dalam direktori mengambil kepemilikan grup dari direktori daripada grup default dari pengguna yang membuat file. Ini memudahkan berbagi file di antara sekelompok pengguna.

### Bit sticky

Bit dengan nilai oktal 1000 adalah bit sticky. Ketika diatur pada direktori, bit sticky mencegah pengguna menghapus atau mengganti nama file yang bukan milik mereka. Ini berguna untuk direktori seperti **/tmp** yang dibagikan di antara banyak pengguna.

### ls: mendaftar dan memeriksa file

Perintah **ls** mendaftar file dan direktori. Ini juga dapat digunakan untuk memeriksa atribut file dan direktori.

Opsi **-l** menyebabkan **ls** menampilkan format panjang, yang mencakup mode file, jumlah hard link ke file, pemilik file, grup file, ukuran file dalam byte, waktu modifikasi file, dan nama file.

Semua direktori memiliki setidaknya dua hard link: satu dari direktori itu sendiri (entri **.**) dan satu dari direktori induknya (entri **..**).

Output **ls** sedikit berbeda untuk file perangkat. Sebagai contoh:

```bash
$ ls -l /dev/tty0
```
![image](https://github.com/user-attachments/assets/1352e1df-8e39-4b03-b28f-60b0bfc46659)

**c** di awal baris menunjukkan bahwa file tersebut adalah file perangkat karakter. **4, 0** di akhir baris adalah nomor perangkat utama dan kecil.

### chmod: mengubah izin

Perintah **chmod** mengubah mode file. Anda dapat menggunakan notasi oktal atau notasi simbolik.

![image](https://github.com/user-attachments/assets/74d4209f-e76c-4e89-8d3a-d8c2dde90d58)


Contoh sintaks mnemonik chmod:

| Spesifier  | Arti                                                                    |
| ---------- | ----------------------------------------------------------------------- |
| u+w        | Menambahkan izin tulis untuk pemilik file                               |
| ug=rw,o=r  | Memberikan izin r/w kepada pemilik dan grup, dan izin r kepada lainnya  |
| a-x        | Menghapus izin eksekusi untuk semua pengguna                            |
| ug=srx, o= | Mengatur bit setuid, setgid, dan sticky untuk pemilik dan grup (r/x)    |
| g=u        | Membuat izin grup sama dengan izin pemilik                              |

**Tips**: Anda juga dapat menentukan mode yang akan ditetapkan dengan menyalin mode dari file lain dengan opsi **--reference** (misalnya **chmod --reference=sourcefile targetfile**)

### chown: mengubah kepemilikan

Perintah **chown** mengubah pemilik dan grup file. Opsi **-R** menyebabkan **chown** mengubah kepemilikan isi file secara rekursif.

```bash
$ chown -R lfiathan:users /home/lfiathan
```

### chgrp: mengubah grup

Perintah **chgrp** mengubah grup file. Opsi **-R** menyebabkan **chgrp** mengubah grup isi file secara rekursif.

```bash
$ chgrp -R users /home/lfiathan
```

### umask: mengatur izin default

Perintah **umask** mengatur izin default untuk file dan direktori baru. Perintah **umask** adalah bit mask yang dikurangkan dari izin default untuk menentukan izin sebenarnya.

Contoh:

```bash
$ umask 022
```

| Oktal | Biner | Izin | Oktal | Biner | Izin |
| ----- | ----- | ---- | ----- | ----- | ---- |
| 0     | 000   | rwx  | 4     | 100   | -wx  |
| 1     | 001   | rw-  | 5     | 101   | -w-  |
| 2     | 010   | r-x  | 6     | 110   | --x  |
| 3     | 011   | r--  | 7     | 111   | ---  |

Sebagai contoh, **umask 027** mengizinkan rwx kepada pemilik, rx kepada grup, dan tidak ada izin kepada orang lain.

## Daftar Kontrol Akses

Model izin Unix tradisional sederhana dan efektif, tetapi memiliki keterbatasan. Misalnya, sulit untuk memberikan file beberapa pemilik, dan sulit untuk memberikan sekelompok pengguna izin yang berbeda pada file yang berbeda.

Daftar Kontrol Akses (ACL) adalah cara untuk memperluas model izin Unix tradisional. ACL memungkinkan Anda memberikan file beberapa pemilik dan memberikan sekelompok pengguna izin yang berbeda pada file yang berbeda.

Setiap aturan dalam ACL disebut **access control entry** (ACE). ACE terdiri dari **penentuan pengguna atau grup**, **mask izin**, dan **tipe**. Penentuan pengguna atau grup bisa berupa nama pengguna, nama grup, atau kata kunci khusus seperti **owner** atau **other**. Mask izin adalah seperangkat izin, dan tipe adalah **allow** atau **deny**.

Perintah **getfacl** menampilkan ACL file, dan perintah **setfacl** mengatur ACL file.

```bash
$ getfacl /etc/passwd
```

```bash
$ setfacl -m u:lfiathan:rw /etc/passwd
```

Ada dua jenis ACL: **POSIX ACL** dan **NFSv4 ACL**. POSIX ACL adalah ACL Unix tradisional, dan NFSv4 ACL adalah jenis ACL yang lebih baru dan lebih kuat.

### Implementasi ACL

Secara teori, tanggung jawab untuk memelihara dan menegakkan ACL dapat diberikan kepada beberapa komponen sistem yang berbeda. ACL dapat diimplementasikan oleh kernel atas nama semua filesystem sistem, oleh masing-masing filesystem, atau mungkin oleh software tingkat yang lebih tinggi seperti server NFS dan SMB.

### POSIX ACL

POSIX ACL adalah ACL Unix tradisional. ACL ini didukung oleh sebagian besar sistem operasi mirip Unix, termasuk Linux, FreeBSD, dan Solaris.

**Entri yang dapat muncul dalam POSIX ACL**

| Format                | Contoh           | Mengatur izin untuk           |
| --------------------- | ---------------- | ----------------------------- |
| user::perms           | user:rw-         | Pemilik file                  |
| user:username:perms   | user:abdou:rw-   | Pengguna bernama username     |
| group::perms          | group:r-x        | Grup file                     |
| group:groupname:perms | group:users:r-x  | Grup bernama groupname        |
| mask::perms           | mask::rwx        | Izin maksimum                 |
| other::perms          | other::r--       | Semua orang lain              |

Contoh:

```bash
$ setfacl -m user:lfiathan:rwx,group:users:rwx,other::r /home/lfiathan

$ getfacl --omit-header /home/lfiathan
```
![image](https://github.com/user-attachments/assets/725a1933-08ac-4f5a-b5c1-d838a445fb96)

### NFSv4 ACL

NFSv4 ACL adalah jenis ACL yang lebih baru dan lebih kuat. ACL ini didukung oleh beberapa sistem operasi mirip Unix, termasuk Linux dan FreeBSD.

NFSv4 ACL mirip dengan POSIX ACL, tetapi memiliki beberapa fitur tambahan. Misalnya, NFSv4 ACL memiliki **default ACL** yang digunakan untuk mengatur ACL file dan direktori baru.

**Izin file NFSv4**
![image](https://github.com/user-attachments/assets/fbe36649-1383-46d3-81dc-9380fc62df38)

