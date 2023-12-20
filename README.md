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
```bash
#update repo terlebih dahulu
sudo apt-get update
#installasi haproxy
sudo apt-get install haproxy

```
## Install NginX + SSL (Let's Encrypt)
![download](https://github.com/dword32bit/SysAdmin/assets/114817148/e3318239-a3a4-449d-bd86-79edc65c4b7f)
kita menggunakan NginX untuk mengelola Web
```bash
#Installasi NginX
sudo apt install nginx

#Periksa status NginX
sudo systemctl status nginx
```
![image](https://github.com/dword32bit/SysAdmin/assets/114817148/4640fe36-9040-4bf5-ad76-410252ad6855)

### Konfigurasi NginX
untuk melakukan konfigurasi menggunakan nano
```bash
sudo nano /etc/nginx/sites-available/danangtri.my.id.conf
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
sudo ln -s /etc/nginx/sites-available/danangtri.my.id.conf /etc/nginx/sites-enabled/danangtri.my.id.conf

#Verifikasi konfigurasi nginx
sudo nginx -t

#jika muncul test is successful berarti tidak ada kendala dalam konfigurasi nginx

#restart nginx
sudo systemctl restart nginx
```
