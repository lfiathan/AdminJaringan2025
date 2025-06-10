---
<div align="center">
  
# Routing dan DNS

## Praktikum Ke-5: Routing dan Konfigurasi DNS

dibuat oleh:

Nama  : Alif Aditya

NRP : 3123600016

Dosen Pengampu

Dr. Ferry Astika Saputra, S.T., M.Sc

GitHub:([@ferryastika](https://github.com/ferryastika))

**Workshop Administrasi Jaringan**

</div>
## Tujuan Praktikum
- Membuat jaringan virtual menggunakan VirtualBox yang terdiri dari 3 VM.
- Mengkonfigurasi VM1 sebagai router (gateway NAT) menggunakan iptables.
- Mengkonfigurasi VM2 sebagai DNS Server menggunakan BIND9.
- Menguji koneksi dan fungsionalitas DNS dari VM3.

---

## Topologi
```
[Internet]
    |
  [VM1] (Router)
  - enp0s3 (Bridge)
  - enp0s8 (Internal: 192.168.200.1)
    |
  [Internal Network]
    |-------------------|
  [VM2] (192.168.200.2)   [VM3] (192.168.200.3)
  - DNS Server            - Client Test
```

---

## Langkah-Langkah Praktikum

### **VM1 - Router/NAT Gateway**
1. **Set Interface di VirtualBox:**
   - Adapter 1: Bridge Adapter
   - Adapter 2: Internal Network (misal: `intnet`)

2. **Cek nama interface:**
   ```bash
   ip a
   ```

3. **Set IP statis untuk enp0s8:**
   ```bash
   sudo nano /etc/network/interfaces
   ```
   Tambahkan:
   ```
   auto enp0s8
   iface enp0s8 inet static
     address 192.168.200.1
     netmask 255.255.255.0
     network 192.168.200.0
     broadcast 192.168.200.255
     dns-nameserver 1.1.1.1
   ```

4. **Restart jaringan:**
   ```bash
   sudo systemctl restart networking
   ```

5. **Install iptables dan aktifkan IP forwarding:**
   ```bash
   sudo apt install iptables -y
   echo 1 | sudo tee /proc/sys/net/ipv4/ip-forward
   ```

6. **Tambahkan aturan iptables NAT dan forwarding:**
   ```bash
   sudo iptables -t nat -A POSTROUTING -o enp0s3 -j MASQUERADE
   sudo iptables -A FORWARD -i enp0s8 -o enp0s3 -j ACCEPT
   sudo iptables -A FORWARD -i enp0s3 -o enp0s8 -j ACCEPT
   ```

7. **Simpan aturan iptables:**
   ```bash
   sudo apt install iptables-persistent -y
   sudo netfilter-persistent save
   ```

---

### **VM2 - DNS Server & Client**
1. **Set Adapter:** Internal Network (nama sama dengan VM1)

2. **Set IP statis melalui GUI/terminal:**
   - IP: 192.168.200.2
   - Netmask: 255.255.255.0
   - Gateway: 192.168.200.1
   - DNS: 192.168.200.1, 1.1.1.1

3. **Install BIND9:**
   ```bash
   sudo apt install bind9 bind9utils bind9-doc -y
   ```

4. **Konfigurasi DNS Forwarder:**
   Edit `/etc/bind/named.conf.options`:
   ```
   forwarders {
     1.1.1.1;
   };
   ```

5. **Tambahkan zone di `/etc/bind/named.conf.local`:**
   ```
   zone "kelompok2.com" {
     type master;
     file "/etc/bind/db.kelompok2";
   };

   zone "200.168.192.in-addr.arpa" {
     type master;
     file "/etc/bind/db.192";
   };
   ```

6. **Salin dan ubah file zona:**
   ```bash
   sudo cp /etc/bind/db.local /etc/bind/db.kelompok2
   sudo cp /etc/bind/db.127 /etc/bind/db.192
   ```

   **Isi `/etc/bind/db.kelompok2`:**
   ```
   @ IN SOA kelompok2.com. root.kelompok2.com. (
       1 ; Serial
       604800 ; Refresh
       86400 ; Retry
       2419200 ; Expire
       604800 ) ; Negative Cache TTL
   ;
   @ IN NS kelompok2.com.
   www IN A 192.168.200.2
   ```

   **Isi `/etc/bind/db.192`:**
   ```
   @ IN SOA kelompok2.com. root.kelompok2.com. (
       1 ; Serial
       604800 ; Refresh
       86400 ; Retry
       2419200 ; Expire
       604800 ) ; Negative Cache TTL
   ;
   @ IN NS kelompok2.com.
   2 IN PTR www.kelompok2.com.
   ```

7. **Restart BIND9:**
   ```bash
   sudo systemctl restart bind9
   ```

---

### **VM3 - Client Uji Coba**
1. **Set Adapter:** Internal Network

2. **Set IP statis:**
   - IP: 192.168.200.3
   - Netmask: 255.255.255.0
   - Gateway: 192.168.200.1
   - DNS: 192.168.200.2

3. **Uji Konektivitas:**
   ```bash
   ping 192.168.200.1
   ping 192.168.200.2
   ping 8.8.8.8
   ```

4. **Uji DNS:**
   ```bash
   dig @192.168.200.2 www.kelompok2.com
   ```

---

## Kesimpulan
Dengan mengikuti instruksi ini, seluruh VM dapat saling terhubung menggunakan internal network. VM1 berhasil berfungsi sebagai router NAT, sementara VM2 berfungsi sebagai DNS server untuk domain internal `kelompok2.com`. Pengujian menggunakan ping dan `dig` menunjukkan bahwa jaringan dan layanan DNS berjalan dengan baik.

---

## Saran Tambahan
- Pastikan semua VM menyala dan berada di network yang sama.
- Gunakan snapshot di VirtualBox sebelum memulai untuk menghindari kesalahan fatal.
- Cek log jika ada error: `journalctl -xe`, `/var/log/syslog`, atau `named-checkzone` untuk debugging DNS.

