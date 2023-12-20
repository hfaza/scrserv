# Kelompok ScrSrv

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

## Install haproxy
![haproxy_images](https://github.com/Xzhacts-Crew/scrserv/assets/114817148/bc74202b-2efe-4281-8a06-421b13eb7b59)
```bash
#update repo terlebih dahulu
sudo apt-get update
#installasi haproxy
sudo apt-get install haproxy

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
