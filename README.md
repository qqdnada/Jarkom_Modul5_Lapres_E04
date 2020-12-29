# Jarkom_Modul5_Lapres_E04

Repository untuk Laporan Resmi Praktikum Modul 5 Jarkom 2020 - E04

* Michael Ricky
  0511184000078
  
* Qatrunada Qori Darwati
  05111840000059

### Subnetting & Routing

Kami menggunakan metode perhitungan CIDR ketika melakukan pembagian subnet.

![](https://github.com/qqdnada/Jarkom_Modul5_Lapres_E04/blob/main/CIDR-Modul-5.JPG)

Berikut tabel IP hasil pembagian subnet

|      | Pembagian IP      |                 |
|------|-------------------|-----------------|
| A1   | Network ID        | 192.168.1.8     |
|      | Netmask           | 255.255.255.252 |
|      | Broadcast Address | 192.168.1.12    |
| A2   | Network ID        | 192.168.3.0     |
|      | Netmask           | 255.255.255.252 |
|      | Broadcast Address | 192.168.2.255   |
| A3   | Network ID        | 192.168.0.0     |
|      | Netmask           | 255.255.255.0   |
|      | Broadcast Address | 192.168.0.255   |
| A4   | Network ID        | 192.168.2.0     |
|      | Netmask           | 255.255.255.0   |
|      | Broadcast Address | 192.168.2.255   |
| A5   | Network ID        | 192.168.1.0     |
|      | Netmask           | 255.255.255.248 |
|      | Broadcast Address | 192.168.1.7     |

Berdasarkan pembagian tersebut, dilakukan routing pada SURABAYA untuk mengenalkannya kepada subnet A1, A2, A5, dan Server MALANG-MOJOKERTO dengan ```ip route add [NID]/[Netmask] via [Gateway]```. Selain melakukan routing pada SURABAYA, dilakukan juga default routing pada KEDIRI dan BATU.

### DHCP

Karena MOJOKERTO diminta menjadi DHCP Server, maka terlebih dulu kita harus menginstall-nya dengan ```apt-get install isc-dhcp-server``` setelah sebelumnya melakukan ```aptget update```. Langkah berikutnya sama seperti pada [Modul 3 - DHCP](https://github.com/arsitektur-jaringan-komputer/Modul-Jarkom/tree/modul-3/DHCP) hanya saja ketika melakukan konfigurasi pada file /etc/dhcp/dhcpd.conf , tambahkan script seperti di bawah ini.

```
subnet 192.168.0.0 netmask 255.255.255.0 {
    range 192.168.0.3 192.168.0.212;
    option routers 192.168.0.1;
    option broadcast-address 192.168.0.255;
    option domain-name-servers 10.151.71.43, 202.46.129.2;
    default-lease-time 600;
    max-lease-time 7200;
}

subnet 192.168.2.0 netmask 255.255.255.0 {
    range 192.168.2.3 192.168.2.202;
    option routers 192.168.2.1;
    option broadcast-address 192.168.2.255;
    option domain-name-servers 10.151.71.43, 202.46.129.2;
    default-lease-time 600;
    max-lease-time 7200;
}
```

Kemudian pada BATU, KEDIRI, dan SURABAYA install DHCP Relay dengan perintah ```apt-get install isc-dhcp-relay```. Isi IP Server dengan IP MOJOKERTO yaitu 10.151.71.43. Jangan lupa untuk mengubah interface klien, SIDOARJO dan GRESIK, dengan konfigurasi berikut.

```
auto eth0
iface eth0 inet dhcp
```

Terakhir, restart semua UML.

### Firewall

**Soal #1**

Agar topologi yang kalian buat dapat mengakses keluar, kalian diminta untuk mengkonfigurasi SURABAYA menggunakan iptables, namun tidak menggunakan MASQUERADE.

Jika biasanya kita menggunakan ```iptables –t nat –A POSTROUTING –o eth0 –j MASQUERADE –s 192.168.0.0/16``` untuk mengakses jaringan luar, disini kita mengganti MASQUERADE nya dengan SNAT --to-source IP_tuntap_surabaya. Adapun konfigurasinya seperti di bawah ini.

```
iptables –t nat –A POSTROUTING –o eth0 –j SNAT --to-source 10.151.70.22 –s 192.168.0.0/16
```

**Soal #2**

Kalian diminta untuk mendrop semua akses SSH dari luar Topologi (UML) kalian pada server yang memiliki IP DMZ (DHCP dan DNS SERVER) pada SURABAYA demi menjaga keamanan.

Disini digunakan ```-A FORWARD``` karena SURABAYA memfilter paket yang menuju MALANG dan MOJOKERTO. Selain itu digunakan juga ```-p tcp --dport 22``` karena kita perlu tahu mana saja yang merupakan akses SSH. Konfigurasi lengkapnya adalah seperti di bawah.

```
iptables -A FORWARD -i eth0 -d 10.151.71.40 -p tcp --dport 22 -j DROP
```

**Soal #3**

Karena tim kalian maksimal terdiri dari 3 orang, Bibah meminta kalian untuk membatasi DHCP dan DNS server hanya boleh menerima maksimal 3 koneksi ICMP secara bersamaan yang berasal dari mana saja menggunakan iptables pada masing masing server, selebihnya akan di DROP.

Lakukan konfigurasi berikut pada MALANG dan MOJOKERTO untuk membatasi akses koneksi ICMP. Pembatasan banyak koneksi diperoleh dengan konfigurasi ```-m connlimit --connlimit-above jumlah_koneksi```.

```
iptables -A INPUT -p icmp -m connlimit --connlimit-above 3 --connlimit-mask 0 -j DROP
```
