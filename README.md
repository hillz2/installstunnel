# CARA INSTALL STUNNEL SSH SSL/TLS DI UBUNTU & DEBIAN UNTUK KPN REVOLUTION
**Tutorial by Galih Prastowo Aji**

## 1. Install dropbear dulu
Di terminal ketik:
```shell
apt-get update
apt-get upgrade
apt-get install -y dropbear
nano /etc/default/dropbear
```
Isi sebagai berikut:
```shell
# disabled because OpenSSH is installed
# change to NO_START=0 to enable Dropbear
NO_START=0

# the TCP port that Dropbear listens on
DROPBEAR_PORT=3128

# any additional arguments for Dropbear
DROPBEAR_EXTRA_ARGS=

# specify an optional banner file containing a message to be
# sent to clients before they connect, such as "/etc/issue.net"
DROPBEAR_BANNER=

# RSA hostkey file (default: /etc/dropbear/dropbear_rsa_host_key)
#DROPBEAR_RSAKEY="/etc/dropbear/dropbear_rsa_host_key"

# DSS hostkey file (default: /etc/dropbear/dropbear_dss_host_key)
#DROPBEAR_DSSKEY="/etc/dropbear/dropbear_dss_host_key"

# Receive window size - this is a tradeoff between memory and
# network performance
DROPBEAR_RECEIVE_WINDOW=65536
```
Lalu jalankan dropbear:
```shell
/etc/init.d/dropbear start
```
## 1. Install stunnel
Di terminal ketik:
```shell
apt-get update
apt-get install -y stunnel4
```
Sekarang buat konfigurasi stunnel:
```shell
nano /etc/stunnel/stunnel.conf
```
Isi sebagai berikut:
```
cert = /etc/stunnel/stunnel.pem
client = no
socket = a:SO_REUSEADDR=1
socket = l:TCP_NODELAY=1
socket = r:TCP_NODELAY=1

[dropbear]
connect = 127.0.0.1:3128
accept = 443
```
Sekarang kita buat sertifikat SSL. **PERHATIAN: Copy command dibawah ini satu - satu, jangan di copy & paste 3 commands sekaligus.**
```shell
openssl genrsa -out key.pem 2048
openssl req -new -x509 -key key.pem -out cert.pem -days 1095
cat key.pem cert.pem >> /etc/stunnel/stunnel.pem
```
Sekarang kita buat stunnel jalan secara otomatis ketika VPS di reboot:
```shell
nano /etc/default/stunnel4
```
Isi sebagai berikut:
```shell
# /etc/default/stunnel
# Julien LEMOINE <speedblue@debian.org>
# September 2003

# Change to one to enable stunnel automatic startup
ENABLED=1
FILES="/etc/stunnel/*.conf"
OPTIONS=""

# Change to one to enable ppp restart scripts
PPP_RESTART=0
```
Sekarang start stunnel di VPS kita:
```shell
/etc/init.d/stunnel4 start
```
Untuk mengecek apakah stunnel & dropbear berjalan normal di VPS, kita bisa menggunakan command:
```shell
netstat -nlpt
```
Maka hasilnya akan seperti ini jika keduanya berjalan normal:
```shell
root@NewDeb:~# netstat -nlpt
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name
tcp        0      0 0.0.0.0:443             0.0.0.0:*               LISTEN      2554/stunnel4   
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      2123/sshd       
tcp        0      0 0.0.0.0:3128            0.0.0.0:*               LISTEN      2708/dropbear   
tcp        0      0 127.0.0.1:25            0.0.0.0:*               LISTEN      2362/exim4      
tcp6       0      0 :::22                   :::*                    LISTEN      2123/sshd       
tcp6       0      0 :::3128                 :::*                    LISTEN      2708/dropbear   
tcp6       0      0 ::1:25                  :::*                    LISTEN      2362/exim4
```
## 3. Sekarang kita coba konek SSH pakai stunnel di KPN Revolution
Kita buat akun dulu di VPS kita
```shell
useradd test
passwd test
```
Di KPN Revolution isikan akun SSH sebagai berikut:
```text
Host: IP VPS KITA
Username: test
Password: test
SSL/SSH Port: 443
```
