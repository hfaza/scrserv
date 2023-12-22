# Kelompok ScrServ
- Danang Tri Atmaja     - 22.83.0826
- Ma'rifah Hadaina Faza - 22.83.0842
- Kuncoro Sihna Mahendra- 22.83.0846
- Ridha Nurrachmat      - 22.83.0857
- Vito Desla Fergiyan   - 22.83.0877

## Server
- Load Balancer Haproxy
- NginX Web Server
- Monitoring NetData
- DNS Server
  
## Operating System
Ubuntu Server 20.04 
https://releases.ubuntu.com/focal/ubuntu-20.04.6-live-server-amd64.iso

### Install SSH di Ubuntu 20.04
```bash
sudo apt-get update
sudo apt-get install openssh-server
# konfigurasi sshd.conf
# hilangkan tanda (#) pada :
Port 1515
PermitRootLogin prohibit-password
# izinkan SSH untuk melewati firewall
sudo ufw allow ssh
```
SSH sudah terinstall, anda bisa menggunakannya pada cmd/terminal
menggunakan command:
> ssh user@ip_address
contoh
> ssh and@192.168.1.5 -p 1515
karena konfigurasi port pada /etc/ssh/sshd_config telah diganti menjadi 1515 maka,
harus menambahkan parameter -p / port

## Aktifasi UFW
```bash
#perintah UFW sudah terpasang dengan menjalankan perintah sebagai berikut :
sudo apt update
sudo apt install ufw
#aktfasi UFW
sudo ufw enable
#cek status UFW
sudo ufw status
```

## Install haproxy
![haproxy_images](https://github.com/Xzhacts-Crew/scrserv/assets/114817148/bc74202b-2efe-4281-8a06-421b13eb7b59)
```bash
#update repo terlebih dahulu
sudo apt-get update
#installasi haproxy
sudo apt-get install haproxy

```
## Konfigurasi Haproxy
```bash
#edit file haproxy
frontend web
  bind *:80
  default_backend app_servers
backend app_servers
  balance roundrobin
  sever web1 192.168.1.20:80 check
  sever web2 192.168.1.30:80 check
```

## Install dan Konfigurasi DNS Server
```bash
# Install DNS Server
sudo apt update && sudo apt upgrade
sudo apt install bind9
# konfigurasi DNS Server
cd /etc/bind
sudo cp db.127 db.ip
sudo cp db.local db.domain
```

```bash
# setelah 2 file tersebut terbuka edit file db.domain
sudo nano db.domain
;
; BIND data file for local loopback interface
;
$TTL    604800
@       IN      SOA     scrserv.com. root.scrserv.com. (
                              2         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
@       IN      NS      scrserv.com.
@       IN      A       192.168.1.5
www     IN      A       192.168.1.5
blog    IN      A       192.168.1.5
```

```bash
# jika ingin menambahkan mx record yang akan digunakan ikuti konfigurasi seperti dibawah
#konfigurasi
;
; BIND data file for local loopback interface
;
$TTL    604800
@       IN      SOA     scrserv.com. root.sceserv.com. (
                              2         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
@       IN      NS      scrserv.com.
        IN      MX      10 scrserv.com.
@       IN      A       192.168.1.5
www     IN      A       192.168.1.5
mail    IN      A       192.168.1.5
blog    IN      CNAME   www
```

```bash
#konfigurasi pada db.ip
;
; BIND reverse data file for local loopback interface
;
$TTL    604800
@       IN      SOA     scrserv.com. root.scrserv.com. (
                              1         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
@       IN      NS      scrserv.com.
5       IN      PTR     scrserv.com.
```
```bash
sudo nano named.conf.local
#konfigurasi
//
// Do any local configuration here
//

// Consider adding the 1918 zones here, if they are not used in your
// organization
//include "/etc/bind/zones.rfc1918";

zone "scrserv.com"{
        type master;
        file "/etc/bind/db.domain";
};

zone "1.1.10.in-addr.arpa"{
        type master;
        file "/etc/bind/db.ip";
};
```

```bash
# melakukan forwarding untuk mengakses domain seperti google.com, facebook.com. disini menggunakan DNS dari google 8.8.8.8 sebagai forwarding
# konfigurasi
sudo nano named.conf.option
# konfigurasi seperti berikut

options {
        directory "/var/cache/bind";

        // If there is a firewall between you and nameservers you want
        // to talk to, you may need to fix the firewall to allow multiple
        // ports to talk.  See http://www.kb.cert.org/vuls/id/800113

        // If your ISP provided one or more IP addresses for stable
        // nameservers, you probably want to use them as forwarders.
        // Uncomment the following block, and insert the addresses replacing
        // the all-0's placeholder.

        forwarders {
        8.8.8.8;
        };

        //========================================================================
        // If BIND logs error messages about the root key being expired,
        // you will need to update your keys.  See https://www.isc.org/bind-keys
        //========================================================================
        dnssec-validation no;

        listen-on-v6 { any; };
};
```
#melakukan restart & status
```bash
sudo systemctl restart bind9.service
sudo systemctl status bind9.service
```

### Install SSL
```bash
#Konfigurasi Netplan
sudo nano /etc/netplan/00-installer-config.yaml
#masukan konfigurasi seperti dibawah
network:
 ethernets:
  enp0s3:
   dhcp4: false
   addresses: [192.168.1.5/24]
   gateway4: 192.168.1.1
   nameservers:
    search: [scrserv.xy]
    addresses: [192.168.1.5, 192.168.1.1]
  version: 2
```

### Install  netdata
```bash
#langkah install
sudo apt update
sudo apt install -y netdata
# konfigurasi netdata
sudo nano /etc/netdata/netdata.conf
#ganti bind socket menjadi 0.0.0.0 supaya dapat diakses semua network
outout :
  [global]
        run as user = netdata
        web files owner = root
        web files group = root
        # Netdata is not designed to be exposed t o potentially hostile
        # network. See https://github.com/netdata/netdata/issues/164
        bind socket to IP = 0.0.0.0
#cek status
sudo systemctl start netdata
sudo systenctl enable netdata
sudo systemctl status netdata
  output
#perizinan
#allow port menggunakan ufw. netdata bekerja pada port 19999
sudo ufw allow 19999/tcp
```

## Install NginX
![download](https://github.com/dword32bit/SysAdmin/assets/114817148/e3318239-a3a4-449d-bd86-79edc65c4b7f)
Saya menggunakan NginX untuk mengelola Web saya yang berada dalam dua sistem operasi yang terpisah dengan server

```bash
#Installasi NginX
sudo apt install nginx

#Periksa status NginX
sudo systemctl status nginx
```

### Konfigurasi NginX
untuk melakukan konfigurasi menggunakan nano
```bash
sudo nano /etc/nginx/sites-available/scrsrv.my.id.conf
```
```bash
server {
        listen 80;

        root /var/www/html;
        index index.html index.htm index.nginx-debian.html;

        server_name scrsrv.my.id www.scrsrv.my.id;

        location / {
                try_files $uri $uri/ =404;
        }

        error_log /var/log/nginx/scrsrv.my.id.error;
        access_log /var/log/nginx/scrsrv.my.id.access;

}

#Installasi FTP server
sudo apt install vsftpd

#Menyalin file konfigurasi sebagai backup
sudo cp /etc/vsftpd.conf /etc/vsftpd.conf.orig

#Izinkan port berkaitan dengan ftp, Kemudian dengan FTP kirimkan file-file pendukung web server
sudo ufw allow 20,21,990/tcp

#Konfigurasi ip address pada masing-masing sistem operasi web server
#Sistem operasi pertama
nano /etc/netplan/00-installer-config.yaml
network:
  ethernets:
    enp0s3:
      dhcp4: no
      addresses: [192.168.1.20/24]
      gateway: 192.168.1.1
  version: 2

#Sistem operasi kedua
nano /etc/netplan/00-installer-config.yaml
network:
  ethernets:
    enp0s3:
      dhcp4: no
      addresses: [192.168.1.30/24]
      gateway: 192.168.1.1
  version: 2

#Setelah semua konfigurasi selesai matikan kembali port yang tidak digunakan
#Hapus semua port allow ufw, dan hanya server yang dapat mengakses web server
ufw allow deny 80/tcp
ufw allow from 192.168.1.5 to any port www
```
![download](https://github.com/Xzhacts-Crew/scrserv/blob/main/webserv.jpg)

simpan konfigurasi dan jalankan konfigurasi tersebut
```bash
sudo ln -s /etc/nginx/sites-available/scrsrv.my.id.conf /etc/nginx/sites-enabled/scrsrv.my.id.conf

#Verifikasi konfigurasi nginx
sudo nginx -t

#jika muncul test is successful berarti tidak ada kendala dalam konfigurasi nginx

#restart nginx
sudo systemctl restart nginx
```
