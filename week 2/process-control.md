# Bab 4: Process Control

![Process Control](https://www.sifars.com/blog/wp-content/uploads/2019/10/Procs-1024x577-1.png)

## Komponen Process

Sebuah process terdiri dari ruang alamat dan sekumpulan struktur data dalam kernel. Ruang alamat adalah sekumpulan halaman memori yang ditandai oleh kernel untuk penggunaan process. (Halaman adalah unit dalam pengelolaan memori. Biasanya berukuran 4KiB atau 8KiB.) Halaman-halaman ini digunakan untuk menyimpan kode, data, dan stack process. Struktur data dalam kernel melacak status process, prioritasnya, parameter penjadwalan, dan sebagainya.

Anggap process sebagai wadah untuk sekumpulan sumber daya yang dikelola kernel atas nama program yang berjalan. Sumber daya ini termasuk halaman memori yang menyimpan kode dan data program, file descriptor yang merujuk ke file terbuka, dan berbagai atribut yang menggambarkan status process.

Struktur data internal kernel mencatat berbagai informasi tentang setiap process:

- Peta ruang alamat process
- Status saat ini dari process (berjalan, tidur, dan sebagainya)
- Prioritas process
- Informasi tentang sumber daya yang digunakan process (CPU, memori, dan sebagainya)
- Informasi tentang file dan port jaringan yang dibuka oleh process
- Mask sinyal process (kumpulan sinyal yang saat ini diblokir)
- Pemilik process (ID pengguna dari pengguna yang memulai process)

"Thread" adalah konteks eksekusi dalam sebuah process. Sebuah process dapat memiliki beberapa thread, yang semuanya berbagi ruang alamat dan sumber daya lainnya. Thread digunakan untuk mencapai paralelisme dalam sebuah process. Thread juga dikenal sebagai process ringan karena pembuatan dan penghancurannya jauh lebih murah daripada process.

Sebagai contoh untuk memahami konsep process dan thread, pertimbangkan server web. Server web mendengarkan koneksi masuk dan kemudian membuat thread baru untuk menangani setiap permintaan masuk. Setiap thread menangani satu permintaan pada satu waktu, tetapi server web secara keseluruhan dapat menangani banyak permintaan secara bersamaan karena memiliki banyak thread. Di sini, server web adalah sebuah process, dan setiap thread adalah konteks eksekusi terpisah dalam process tersebut.

### PID: nomor ID process

Setiap process diidentifikasi oleh nomor ID process unik, atau PID. PID adalah bilangan bulat yang ditetapkan kernel ke setiap process saat process dibuat. PID digunakan untuk merujuk pada process dalam berbagai panggilan sistem, misalnya, untuk mengirim sinyal ke process.

Konsep "namespace" process memungkinkan process yang berbeda memiliki PID yang sama. Namespace digunakan untuk membuat container, yang merupakan lingkungan terisolasi dengan tampilan sistem mereka sendiri. Container digunakan untuk menjalankan beberapa instance aplikasi pada sistem yang sama, masing-masing dalam lingkungan terisolasi mereka sendiri.

### PPID: nomor ID process induk

Setiap process juga dikaitkan dengan process induk, yaitu process yang membuatnya. Nomor ID process induk, atau PPID, adalah PID dari process induk. PPID digunakan untuk merujuk pada process induk dalam berbagai panggilan sistem, misalnya, untuk mengirim sinyal ke process induk.

### UID dan EUID: ID pengguna dan ID pengguna efektif

ID pengguna, atau UID, adalah ID pengguna dari pengguna yang memulai process. ID pengguna efektif, atau EUID, adalah ID pengguna yang digunakan process untuk menentukan sumber daya apa yang dapat diakses oleh process. EUID digunakan untuk mengontrol akses ke file, port jaringan, dan sumber daya lainnya.


## Siklus Hidup Process

Untuk membuat process baru, sebuah process menyalin dirinya dengan panggilan sistem **fork**. **fork** membuat salinan dari process asli, dan salinan itu sebagian besar identik dengan induknya. Process baru memiliki **PID** yang berbeda dan memiliki informasi akuntingnya sendiri. (Secara teknis, sistem Linux menggunakan **clone**, superset dari **fork** yang menangani thread dan mencakup fitur tambahan. **fork** tetap ada di kernel untuk kompatibilitas mundur tetapi memanggil **clone** secara internal.)

Ketika sistem boot, kernel secara otonom membuat dan menginstal beberapa process. Yang paling menonjol adalah **init** atau **systemd**, yang selalu process nomor 1. Process ini menjalankan skrip startup sistem, meskipun cara persisnya berbeda sedikit antara UNIX dan Linux. Semua process selain yang dibuat oleh kernel adalah keturunan dari process primordial ini.

### Sinyal

Sinyal adalah cara untuk mengirim notifikasi ke suatu process. Mereka digunakan untuk memberi tahu process bahwa peristiwa tertentu telah terjadi.

Sekitar tiga puluh jenis berbeda didefinisikan, dan mereka digunakan dalam berbagai cara:

- Mereka dapat dikirim antar process sebagai sarana komunikasi.
- Mereka dapat dikirim oleh driver terminal untuk membunuh, mengganggu, atau menangguhkan process ketika tombol seperti <Control-C> dan <Control-Z> ditekan.
- Mereka dapat dikirim oleh administrator (dengan kill) untuk mencapai berbagai tujuan.
- Mereka dapat dikirim oleh kernel ketika process melakukan pelanggaran seperti pembagian dengan nol.
- Mereka dapat dikirim oleh kernel untuk memberi tahu process tentang kondisi "menarik" seperti kematian process anak atau ketersediaan data pada saluran I/O.

![Signals](https://liujunming.top/images/2018/12/71.png)

Sinyal KILL, INT, TERM, HUP, dan QUIT semuanya terdengar seolah-olah berarti kurang lebih sama, tetapi penggunaannya sebenarnya cukup berbeda.

- **KILL** tidak dapat diblokir dan menghentikan sebuah process di tingkat kernel. Process tidak pernah benar-benar menerima atau menangani sinyal ini.
- **INT** dikirim oleh driver terminal ketika pengguna mengetik <Control-C>. Ini adalah permintaan untuk menghentikan operasi saat ini. Program sederhana harus keluar (jika mereka menangkap sinyal) atau membiarkan diri mereka dibunuh, yang merupakan default jika sinyal tidak ditangkap. Program yang memiliki baris perintah interaktif (seperti shell) harus berhenti dari apa yang mereka lakukan, membersihkan, dan menunggu input pengguna lagi.
- **TERM** adalah permintaan untuk menghentikan eksekusi sepenuhnya. Diharapkan process penerima akan membersihkan statusnya dan keluar.
- **HUP** dikirim ke process ketika terminal pengendali ditutup. Awalnya digunakan untuk menunjukkan "hang up" koneksi telepon, sekarang sering digunakan untuk menginstruksikan process daemon untuk menghentikan dan memulai ulang, sering untuk memperhitungkan konfigurasi baru. Perilaku pastinya tergantung pada process spesifik yang menerima sinyal HUP.
- **QUIT** mirip dengan TERM, kecuali defaultnya menghasilkan core dump jika tidak ditangkap. Beberapa program mengkanibal sinyal ini dan menafsirkannya untuk berarti sesuatu yang lain.

### kill: kirim sinyal

Seperti namanya, perintah **kill** paling sering digunakan untuk menghentikan process. **kill** dapat mengirim sinyal apa pun, tetapi secara default mengirim TERM. **kill** dapat digunakan oleh pengguna normal pada process mereka sendiri atau oleh root pada process apa pun. Sintaksnya adalah:

```bash
kill [-signal] pid
```

di mana *signal* adalah nomor atau nama simbolik dari sinyal yang akan dikirim dan *pid* adalah nomor identifikasi process dari process target.

**kill** tanpa nomor sinyal tidak menjamin bahwa process akan mati, karena sinyal TERM dapat ditangkap, diblokir, atau diabaikan. Perintah **kill -9 pid** dijamin akan membunuh process karena sinyal KILL tidak dapat ditangkap, diblokir, atau diabaikan.

**killall** membunuh process berdasarkan nama daripada ID process. Ini tidak tersedia di semua sistem.
Contoh:

```bash
killall firefox
```

Perintah **pkill** mirip dengan **killall** tetapi menyediakan lebih banyak opsi.

Contoh:

```bash
pkill -u abdoufermat # bunuh semua process yang dimiliki oleh pengguna abdoufermat
```

## PS: Memantau Process

Perintah **ps** adalah alat utama administrator sistem untuk memantau process. Meskipun versi **ps** berbeda dalam argumen dan tampilannya, semuanya menyampaikan informasi yang pada dasarnya sama.

**ps** dapat menunjukkan PID, UID, prioritas, dan terminal kontrol dari process. Ini juga memberi tahu Anda berapa banyak memori yang digunakan oleh process, berapa banyak waktu CPU yang telah dikonsumsinya, dan apa status saat ini (berjalan, berhenti, tidur, dan sebagainya).

Anda dapat memperoleh gambaran umum yang berguna tentang sistem dengan menjalankan **ps aux**. Opsi **a** memberi tahu **ps** untuk menampilkan process semua pengguna, dan opsi **u** memberi tahu untuk memberikan informasi rinci tentang setiap process. Opsi **x** memberi tahu **ps** untuk menampilkan process yang tidak terkait dengan terminal.

```bash
$ ps aux | head -8
```
![image](https://github.com/user-attachments/assets/6951ed6d-d0ec-4267-901c-a9b73521eda4)

![process-explanation](./data/process-explanation.png)

Seperangkat argumen berguna lainnya adalah **lax**, yang memberikan informasi teknis lebih banyak tentang process. **lax** sedikit lebih cepat daripada **aux** karena tidak perlu menyelesaikan nama pengguna dan grup.

```bash
$ ps lax | head -8
```
![image](https://github.com/user-attachments/assets/4e3fff1a-0370-4273-9a08-4cb77ff2170f)


Untuk mencari process tertentu, Anda dapat menggunakan **grep** untuk memfilter output **ps**.

```bash
$ ps aux | grep -v grep | grep firefox
```

Kita dapat menentukan PID sebuah process dengan menggunakan **pgrep**.

```bash
$ pgrep firefox
```

atau **pidof**.

```bash
$ pidof /usr/bin/firefox
```

## Pemantauan interaktif dengan top

Perintah **top** menyediakan tampilan dinamis real-time dari sistem yang berjalan. Ini dapat menampilkan informasi ringkasan sistem serta daftar process atau thread yang saat ini dikelola oleh kernel Linux. Jenis informasi ringkasan sistem yang ditampilkan dan jenis, urutan, dan ukuran informasi yang ditampilkan untuk process semuanya dapat dikonfigurasi oleh pengguna dan konfigurasi itu dapat dibuat persisten di seluruh restart.

Secara default, tampilan diperbarui setiap 1-2 detik, tergantung pada sistem.

Ada juga perintah **htop**, yang merupakan penampil process interaktif untuk sistem Unix. Ini adalah aplikasi mode teks (untuk konsol atau terminal X) dan memerlukan ncurses. Ini mirip dengan top, tetapi memungkinkan Anda untuk menggulir secara vertikal dan horizontal, sehingga Anda dapat melihat semua process yang berjalan pada sistem, bersama dengan baris perintah lengkap mereka. **htop** juga memiliki antarmuka pengguna yang lebih baik dan lebih banyak opsi untuk operasi.

## Nice dan renice: mengubah prioritas process

**Niceness** adalah petunjuk numerik ke kernel tentang bagaimana process harus diperlakukan dalam kaitannya dengan process lain yang bersaing untuk CPU.

Niceness tinggi berarti prioritas rendah untuk process Anda: Anda akan menjadi baik. Nilai rendah atau negatif berarti prioritas tinggi: Anda tidak terlalu baik!

Rentang nilai niceness yang diizinkan bervariasi antar sistem. Di Linux rentangnya adalah -20 hingga +19, dan di FreeBSD -20 hingga +20.

Process prioritas rendah adalah yang tidak terlalu penting. Ini akan mendapatkan waktu CPU lebih sedikit daripada process prioritas tinggi. Process prioritas tinggi adalah yang penting dan harus diberi lebih banyak waktu CPU daripada process prioritas rendah.

Misalnya, jika Anda menjalankan pekerjaan intensif CPU yang ingin Anda jalankan di latar belakang, Anda dapat memulainya dengan nilai niceness tinggi. Ini akan memungkinkan process lain berjalan tanpa diperlambat oleh pekerjaan Anda.

Perintah **nice** digunakan untuk memulai process dengan nilai niceness tertentu. Sintaksnya adalah:

```bash
nice -n nice_val [command]

# Contoh
nice -n 10 sh infinite.sh &
```

Perintah **renice** digunakan untuk mengubah nilai niceness dari process yang sedang berjalan. Sintaksnya adalah:

```bash
renice -n nice_val -p pid

# Contoh
renice -n 10 -p 1234
```

**Nilai prioritas** adalah prioritas sebenarnya process yang digunakan oleh kernel Linux untuk menjadwalkan tugas.
Dalam sistem Linux prioritas adalah 0 hingga 139 di mana 0 hingga 99 untuk real-time dan 100 hingga 139 untuk pengguna.

Hubungan antara nilai nice dan prioritas adalah sebagai berikut:

> nilai_prioritas = 20 + nilai_nice

Nilai nice default adalah 0. Semakin rendah nilai nice, semakin tinggi prioritas process.

## Sistem file /proc

Versi **ps** dan **top** Linux membaca informasi status process mereka dari direktori **/proc**, sebuah sistem file pseudo di mana kernel mengekspos berbagai informasi menarik tentang status sistem.

Meskipun namanya, **/proc** berisi informasi lain selain hanya process (statistik yang dihasilkan oleh sistem, dll).

Process diwakili oleh direktori di **/proc**, dan setiap process memiliki direktori yang diberi nama sesuai PID-nya. Direktori **/proc** berisi berbagai file yang memberikan informasi tentang process, seperti baris perintah, variabel lingkungan, file descriptor, dan sebagainya.

![process-information](./data/process-information.png)

## Strace dan truss

Untuk mengetahui apa yang dilakukan oleh process, Anda dapat menggunakan **strace** di Linux atau **truss** di FreeBSD. Perintah ini melacak panggilan sistem dan sinyal. Mereka dapat digunakan untuk men-debug program atau untuk memahami apa yang sedang dilakukan program.

Misalnya, log berikut dihasilkan oleh strace yang dijalankan terhadap salinan aktif top (yang berjalan sebagai PID 5810):

```bash
$ strace -p 5810

gettimeofday({1197646605,  123456}, {300, 0}) = 0
open("/proc", O_RDONLY|O_NONBLOCK|O_LARGEFILE|O_DIRECTORY) = 7
fstat64(7, {st_mode=S_IFDIR|0555, st_size=0, ...}) = 0
fcntl64(7, F_SETFD, FD_CLOEXEC)          = 0
getdents64(7, /* 3 entries */, 32768)   = 72
getdents64(7, /* 0 entries */, 32768)   = 0
stat64("/proc/1", {st_mode=S_IFDIR|0555, st_size=0, ...}) = 0
open("/proc/1/stat", O_RDONLY)           = 8
read(8, "1 (init) S 0 1 1 0 -1 4202752"..., 1023) = 168
close(8)                                = 0

[...]
```

**top** mulai dengan memeriksa waktu saat ini. Kemudian membuka dan memeriksa direktori **/proc**, dan membaca file **/proc/1/stat** untuk mendapatkan informasi tentang process **init**.

## Process yang tidak terkendali

Terkadang process akan berhenti merespons sistem dan berjalan liar. Process ini mengabaikan prioritas penjadwalan mereka dan bersikeras mengambil 100% CPU. Karena process lain hanya dapat mendapatkan akses terbatas ke CPU, mesin mulai berjalan sangat lambat. Ini disebut process yang tidak terkendali.

Perintah **kill** dapat digunakan untuk menghentikan process yang tidak terkendali. Jika process tidak merespons sinyal TERM, Anda dapat menggunakan sinyal KILL untuk menghentikannya.

```bash
kill -9 pid

atau

kill -KILL pid
```

Kita dapat menyelidiki penyebab process yang tidak terkendali dengan menggunakan **strace** atau **truss**. Process yang tidak terkendali yang menghasilkan output dapat mengisi seluruh sistem file.

Anda dapat menjalankan **df -h** untuk memeriksa penggunaan sistem file. Jika sistem file penuh, Anda dapat menggunakan perintah **du** untuk menemukan file dan direktori terbesar.


Anda juga dapat menggunakan perintah **lsof** untuk mengetahui file mana yang dibuka oleh process yang tidak terkendali.

```bash
lsof -p pid
```

## Process berkala

### cron: jadwalkan perintah

Daemon cron (crond di RedHat: ya aneh!) adalah alat tradisional untuk menjalankan perintah pada jadwal yang telah ditentukan. Ini dimulai ketika sistem boot dan berjalan selama sistem aktif.

**cron** membaca file konfigurasi yang berisi daftar baris perintah dan waktu kapan mereka akan dipanggil. Baris perintah dieksekusi oleh **sh**, jadi hampir semua yang bisa Anda lakukan secara manual dari shell juga bisa dilakukan dengan **cron**.

File konfigurasi cron disebut "crontab," singkatan dari "cron table." Crontab untuk pengguna individual disimpan di bawah **/var/spool/cron** (Linux) atau **/var/cron/tabs** (FreeBSD).

### format crontab

File crontab memiliki lima bidang untuk menentukan hari, tanggal dan waktu diikuti oleh perintah yang akan dijalankan pada interval tersebut.

```bash
*     *     *     *     *  perintah yang akan dieksekusi
-     -     -     -     -
|     |     |     |     |
|     |     |     |     +----- hari dalam seminggu (0 - 6) (Minggu=0)
|     |     |     +------- bulan (1 - 12)
|     |     +--------- hari dalam bulan (1 - 31)
|     +----------- jam (0 - 23)
+------------- menit (0 - 59)
```

Beberapa contoh:

```bash
# Jalankan perintah pada pukul 2:30 pagi setiap hari
30 2 * * * command

# Jalankan perintah pada pukul 10:30 malam pada tanggal 1 setiap bulan
30 22 1 * * command

# Jalankan skrip Python setiap tanggal 1 bulan pada pukul 2:30 pagi
30 2 1 * * /usr/bin/python3 /path/to/script.py
```

Jadwal berikut: 0,30 * 13 * 5 berarti bahwa perintah akan dieksekusi pada menit 0 dan 30 melewati jam ke-13 pada hari Jumat. Jika Anda ingin menjalankan perintah setiap 30 menit, Anda dapat menggunakan jadwal berikut: */30 * * * *.

**manajemen crontab**

Perintah **crontab** digunakan untuk membuat, memodifikasi, dan menghapus crontab. Opsi **-e** digunakan untuk mengedit file crontab, opsi **-l** digunakan untuk mencantumkan file crontab, dan opsi **-r** digunakan untuk menghapus file crontab.

### Timer Systemd

Timer systemd adalah file konfigurasi unit yang namanya berakhir dengan **.timer**. Timer systemd dapat digunakan sebagai alternatif untuk pekerjaan cron. Mereka lebih fleksibel dan lebih kuat daripada pekerjaan cron.

Unit timer diaktifkan oleh unit layanan yang sesuai. Unit layanan dipicu oleh unit timer pada waktu yang ditentukan dalam unit timer. Unit timer juga dapat diaktifkan oleh boot sistem atau oleh suatu peristiwa.

Perintah **systemctl** digunakan untuk mengelola unit systemd. Opsi **list-timers** digunakan untuk mencantumkan timer aktif.

```bash
$ systemctl list-timers

NEXT                         LEFT          LAST                         PASSED       UNIT                         ACTIVATES
Fri 2021-10-15 00:00:00 UTC  1h 1min left Thu 2021-10-14 00:00:00 UTC  22h ago      logrotate.timer              logrotate.service

1 timers listed.
```

Dalam contoh di atas, unit **logrotate.timer** dijadwalkan untuk mengaktifkan unit **logrotate.service** pada tengah malam setiap hari.

Berikut tampilan unit **logrotate.timer**:

```bash
$ cat /usr/lib/systemd/system/logrotate.timer

[Unit]
Description=Daily rotation of log files
Documentation=man:logrotate(8) man:logrotate.conf(5)

[Timer]
OnCalendar=daily
AccuracySec=1h
Persistent=true

[Install]
WantedBy=timers.target

```

Opsi **OnCalendar** digunakan untuk menentukan kapan timer harus mengaktifkan layanan. Opsi **AccuracySec** digunakan untuk menentukan keakuratan timer. Opsi **Persistent** digunakan untuk menentukan apakah timer harus mengejar ketinggalan pada eksekusi yang terlewatkan.


### Penggunaan umum untuk tugas terjadwal

**Mengirim surat**

Anda dapat secara otomatis mengirim email output laporan harian atau hasil eksekusi perintah menggunakan **cron** atau timer **systemd**.

Misalnya:
    
```bash
30 4 25 * * /usr/bin/mail -s "Laporan Bulanan"
    abdou@admin.com%Terima laporan bulanan untuk bulan Juli!%%Dengan hormat,%cron%
```

**Membersihkan sistem file**

Anda dapat menggunakan **cron** atau timer **systemd** untuk menjalankan skrip yang membersihkan sistem file. Misalnya, Anda dapat menggunakan skrip untuk membersihkan isi direktori sampah setiap hari pada tengah malam.

```bash
0 0 * * * /usr/bin/find /home/abdou/.local/share/Trash/files -mtime +30 -exec /bin/rm -f {} \;
```

**Merotasi file log**

Merotasi file log berarti membaginya menjadi segmen berdasarkan ukuran atau tanggal, menjaga beberapa versi log yang lebih lama tetap tersedia setiap saat. Karena rotasi log adalah peristiwa berulang dan terjadi secara teratur, ini adalah tugas ideal untuk dijadwalkan.

**Menjalankan pekerjaan batch**

Beberapa perhitungan yang berjalan lama terbaik dijalankan sebagai pekerjaan batch. Misalnya, pesan dapat terakumulasi dalam antrian atau database. Anda dapat menggunakan pekerjaan cron untuk memproses semua pesan yang antri sekaligus sebagai ETL (Extract, Transform, Load) ke lokasi lain, seperti gudang data.

**Backup dan mirroring**

Anda dapat menggunakan tugas terjadwal untuk secara otomatis mencadangkan direktori ke sistem jarak jauh.
Mirror adalah salinan byte-per-byte dari sistem file atau direktori yang di-host di sistem lain. Mereka dapat digunakan sebagai bentuk backup atau sebagai cara untuk mendistribusikan file di beberapa sistem.
Anda dapat menggunakan eksekusi berkala **rsync** untuk menjaga mirror tetap up to date.
