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
SSH sudah terinstall, anda bisa menggunakan nya pada cmd/terminal
menggunakan command :

> ssh user@ip_address

contoh
> ssh and@103.82.92.91 -p 1515

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
  bind*;80
  default_backend app_servers
backend app_servers
  balance roundrobin
  sever web1 192.168.179.5:80 check
  sever web2 192.168.179.10:80 check
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
Saya menggunakan NginX untuk mengelola Web saya

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
```
simpan konfigurasi dan jalankan konfigurasi tersebut
```bash
sudo ln -s /etc/nginx/sites-available/scrsrv.my.id.conf /etc/nginx/sites-enabled/scrsrv.my.id.conf

#Verifikasi konfigurasi nginx
sudo nginx -t

#jika muncul test is successful berarti tidak ada kendala dalam konfigurasi nginx

#restart nginx
sudo systemctl restart nginx
```
