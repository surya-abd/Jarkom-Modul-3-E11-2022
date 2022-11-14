<table>
<tr>
<th>Nama</th>
<th>NRP </th>
</tr>
<tr>
<td>Surya Abdillah</td>
<td>5025201229 </td>
</tr>
<tr>
<td>Muhammad Afdal Abdallah</td>
<td>5025201163 </td>
</tr>
<tr>
<td>Kadek Ari Dharmika</td>
<td>5025201239 </td>
</tr>
</table>

# Modul 3

## TOPOLOGI
![image](https://drive.google.com/uc?export=view&id=1MGJJk0U8ifnUyenxbHAHxFIxiN1IHe1S)

## SOAL PRAKTIKUM
### Nomor 1
> Loid bersama Franky berencana membuat peta tersebut dengan kriteria WISE sebagai DNS Server, Westalis sebagai DHCP Server, Berlint sebagai Proxy Server

Pada node ```Ostania```
```
iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE -s 10.27.0.0/16
```

Pada node ```WISE```
```
echo "nameserver 192.168.122.1" > /etc/resolv.conf

apt-get update
apt-get install bind9 -y
service bind9 restart
```
Pada node ```Westalis```
```
echo "nameserver 192.168.122.1" > /etc/resolv.conf

apt-get update
apt-get install isc-dhcp-server -y

echo "
INTERFACES=\"eth0\"
" > /etc/default/isc-dhcp-server
```
Pada node ```Berlint```
```
echo "nameserver 192.168.122.1" > /etc/resolv.conf

apt-get update
apt-get install squid -y

service squid restart
```

### Nomor 2
> dan Ostania sebagai DHCP Relay 
Pada node ```Ostania```
```
apt-get update
apt-get install isc-dhcp-relay -y

echo "
SERVERS=\"10.27.2.4\"
INTERFACES=\"eth1 eth2 eth3\"
" > /etc/default/isc-dhcp-relay

service isc-dhcp-relay restart
```


### Nomor 3-6
> Semua client yang ada HARUS menggunakan konfigurasi IP dari DHCP Server. <br>
Client yang melalui Switch1 mendapatkan range IP dari [prefix IP].1.50 - [prefix IP].1.88 dan [prefix IP].1.120 - [prefix IP].1.155 (3) <br>
Client yang melalui Switch3 mendapatkan range IP dari [prefix IP].3.10 - [prefix IP].3.30 dan [prefix IP].3.60 - [prefix IP].3.85 (4) <br>
Client mendapatkan DNS dari WISE dan client dapat terhubung dengan internet melalui DNS tersebut. (5) <br>
Lama waktu DHCP server meminjamkan alamat IP kepada Client yang melalui Switch1 selama 5 menit sedangkan pada client yang melalui Switch3 selama 10 menit. Dengan waktu maksimal yang dialokasikan untuk peminjaman alamat IP selama 115 menit. (6)

Konfigurasi pada ```SSS, Garden, Eden, NewstonCastle, KemonoPark```
```
auto eth0
iface eth0 inet dhcp
```

Konfigurasi pada node ```Westalis``` ```/etc/dhcp/dhcpd.conf``` untuk nomor 3-6
```
echo "
ddns-update-style none;

option domain-name \"example.org\";
option domain-name-servers ns1.example.org, ns2.example.org;

default-lease-time 600;
max-lease-time 7200;

log-facility local7;

subnet 10.27.1.0 netmask 255.255.255.0 {
        range 10.27.1.50 10.27.1.88;
        range 10.27.1.120 10.27.1.155;
        option routers 10.27.1.1;
        option broadcast-address 10.27.1.255;
        option domain-name-servers 10.27.2.2;
        default-lease-time 300;
        max-lease-time 6900;
}

subnet 10.27.2.0 netmask 255.255.255.0 {
}


subnet 10.27.3.0 netmask 255.255.255.0 {
        range 10.27.3.10 10.27.3.30;
        range 10.27.3.60 10.27.3.85;
        option routers 10.27.3.1;
        option broadcast-address 10.27.3.255;
        option domain-name-servers 10.27.2.2;
        default-lease-time 600;
        max-lease-time 6900;
}
" > /etc/dhcp/dhcpd.conf
```

> Hasil

![image](https://user-images.githubusercontent.com/103357229/201679929-9b48440b-d4bd-46f4-9725-4ef9c3f4fc21.png)

### Nomor 7
> Loid dan Franky berencana menjadikan Eden sebagai server untuk pertukaran informasi dengan alamat IP yang tetap dengan IP [prefix IP].3.13 (7)
> IP eden sebelum dilakukan fixed prefix

![image](https://user-images.githubusercontent.com/103357229/201677899-af7cd1fe-3537-4242-b99f-b45ec8cff74d.png)

> Westalis
Melakukan konfigurasi pada dhcpd.conf, dengan menambah kode:
```
host Eden {
        hardware ethernet 46:c0:e6:ce:a3:9d;
        fixed-address 10.27.3.13;
}
```

> EDEN
Melakukan konfigurasi pada interfaces 
```
auto eth0
iface eth0 inet dhcp

hwaddress ether 46:c0:e6:ce:a3:9d
```

> dilakukan restart pada node eden dan menadapatkan:

![image](https://user-images.githubusercontent.com/103357229/201680164-f8d80265-c7e2-4d92-a5fa-1e8ef9848fda.png)

### PROXY
> SSS, Garden, dan Eden digunakan sebagai client Proxy agar pertukaran informasi dapat terjamin keamanannya, juga untuk mencegah kebocoran data. Pada Proxy Server di Berlint, Loid berencana untuk mengatur bagaimana Client dapat mengakses internet. Artinya setiap client harus menggunakan Berlint sebagai HTTP & HTTPS proxy. Adapun kriteria pengaturannya adalah sebagai berikut:

> Melakukan aktivasi proxy pada klien

```
apt-get install lynx

export http_proxy="http://10.27.2.3:8080"
env | grep -i proxy
```

> Berlint

```
mv /etc/squid/squid.conf /etc/squid/squid.conf.bak
```

> Merubah konfigurasi squid.conf

```
http_port 8080

http_access allow all

visible_hostname Berlint
```

> restart

```
service squid restart
```

### Nomor 1
> Client hanya dapat mengakses internet diluar (selain) hari & jam kerja (senin-jumat 08.00 - 17.00) dan hari libur (dapat mengakses 24 jam penuh)

#### BERLINT

> Membuat file /etc/squid/acl.conf dan melakukan menambahkan waktu:

```
acl deny_1 time MTWHF 08:00-17:00
acl pagi time MTWHF 00:01-08:00
acl malam time MTWHF 17:00-23:59
acl weekend time AS 00:00-23:59
```

> melakukan konfigurasi pada /etc/squid/squid.conf

```
include /etc/squid/acl.conf

http_port 8080
http_access deny deny_1
visible_hostname Berlint
```

> HASIL TESTING pad klien

> hari libur

![image](https://user-images.githubusercontent.com/103357229/201683297-5f800e42-5d0d-472d-9a73-2489906d7568.png)

> hari dan Jam kerja

![image](https://user-images.githubusercontent.com/103357229/201683488-98dd4743-7de3-41e9-8984-5a56166b14e6.png)

> hari kerja dan jam luar kerja

![image](https://user-images.githubusercontent.com/103357229/201683883-fb341f64-2f5c-44cd-86e0-27667cb9787f.png)

### Nomor 2
> Adapun pada hari dan jam kerja sesuai nomor (1), client hanya dapat mengakses domain loid-work.com dan franky-work.com (IP tujuan domain dibebaskan)

> Membuat domain pada WISE sebagai DNS server:

```
echo "
zone \"franky-work.com\" {
        type master;
        file \"/etc/bind/franky/franky-work.com\";
};

zone \"loid-work.com\" {
        type master;
        file \"/etc/bind/loid/loid-work.com\";
};
" > /etc/bind/named.conf.local

mkdir /etc/bind/franky
mkdir /etc/bind/loid

cp /etc/bind/db.local /etc/bind/franky/franky-work.com
cp /etc/bind/db.local /etc/bind/loid/loid-work.com

echo "
;
; BIND data file for local loopback interface
;
$TTL    604800
@       IN      SOA     franky-work.com. root.franky-work.com. (
                              2         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
@       IN      NS      franky-work.com.
@       IN      A       10.27.2.2       ; IP WISE
@       IN      AAAA    ::1
" > /etc/bind/franky/franky-work.com

echo "
;
; BIND data file for local loopback interface
;
$TTL    604800
@       IN      SOA     loid-work.com. root.loid-work.com. (
                              2         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
@       IN      NS      loid-work.com.
@       IN      A       10.27.2.2       ; IP WISE
@       IN      AAAA    ::1
" > /etc/bind/loid/loid-work.com

service bind9 restart

```

> KONFIGURASI PADA BERLINT

> KONFIGURASI UNTUK NOMOR 2 HINGGA 5

```
echo "
franky-work.com
loid-work.com
" > /etc/squid/sites.allow.kerja.txt
```

> Merubah konfigurasi /etc/squid/squid.conf

```
include /etc/squid/acl.conf

acl WORKING_SITES dstdomain "/etc/squid/sites.allow.kerja.txt"
acl SSL_PORT port 443
delay_pools 1
delay_class 1 1
delay_access 1 allow weekend
delay_access 1 deny all
delay_parameters 1 16000/16000

http_port 8080
visible_hostname Berlint

#http_access allow all
http_access allow SSL_PORT !WORKING_SITES !deny_1
http_access allow WORKING_SITES deny_1
http_access deny all
```

> Hasil Testing:

#### HTTP

> Senin 10:00

![image](https://user-images.githubusercontent.com/103357229/201689041-ef91e6fe-0bde-40d5-a441-0b0c7b663910.png)

> Senin 20:00

![image](https://user-images.githubusercontent.com/103357229/201688761-477160c1-681f-4249-b709-1b84173ba844.png)

> Sabtu 10:00

![image](https://user-images.githubusercontent.com/103357229/201688508-34223bab-4ae3-42fa-8f27-352847af14c2.png)

#### HTTPS

> Senin 10:00

![image](https://user-images.githubusercontent.com/103357229/201689188-cb2ac4cf-9c2a-4f2f-b599-eeb0d0c3e0e5.png)

> Senin 20:00

![image](https://user-images.githubusercontent.com/103357229/201688689-972aefbc-cf82-42a4-8eae-339314b47fa6.png)

> Sabtu 10:00

![image](https://user-images.githubusercontent.com/103357229/201688377-916822ca-6e62-453e-b20b-a516232afd10.png)


### Nomor 3
> Saat akses internet dibuka, client dilarang untuk mengakses web tanpa HTTPS. (Contoh web HTTP: http://example.com)

> Senin 10:00



> Senin 20:00

> Sabtu 10:00

### Nomor 4
> Agar menghemat penggunaan, akses internet dibatasi dengan kecepatan maksimum 128 Kbps pada setiap host (Kbps = kilobit per second; lakukan pengecekan pada tiap host, ketika 2 host akses internet pada saat bersamaan, keduanya mendapatkan speed maksimal yaitu 128 Kbps)

### Nomor 5
> Setelah diterapkan, ternyata peraturan nomor (4) mengganggu produktifitas saat hari kerja, dengan demikian pembatasan kecepatan hanya diberlakukan untuk pengaksesan internet pada hari libur


