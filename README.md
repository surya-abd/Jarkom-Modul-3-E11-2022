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
