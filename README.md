# IRU Labor - Linux
## Elindulás
- le kell tötlteni githubról a megadott file okat a gazdagépen
- WinScp-vel fel kell másolni a virtuális gépre
- a kapcsolódáshoz a virtuális gépen a 'hostname -I' parancsal lekérjük az IP-t
- belépünk WinScp-vel labor userként és felmásoljuk a home/laboruser/Downloads mappába
- a virt gépen átváltunk root-ra és elnavigálunk a Downloads mappához
- cp iru-data.iru /root
- 'iru-test NEPTUN' vel futtatjuk
- a parancsokat a root felhasználóval kell kiadni a feladatokban
- egy nano-ban megnyitott fájlt CTRL+X -> Y > ENTER kombinációval lehet elmenteni
## 1. feladatcsoport
### 1.4
- az alábbi parancsoknál az ip-t ki kell cserélni arra az ip-re amely a virtuális gép helyi hálozata
```
ip addr
iptables -A INPUT -p tcp --dport 22 -s 192.168.193.0/24 -j ACCEPT
iptables -A INPUT -p tcp --dport 22 -j DROP
```
### 1.5
```
iptables -A INPUT -p icmp --icmp-type echo-request -j DROP
```
### 1.6
```
adduser mekkelek --gecos "" --disabled-password
passwd mekkelek
```
### 1.7
```
nano etc/sudoers
```
- a megnyitott fálhoz hozzá kell adni a következő sort
```
mekkelek  All(ALL:ALL) ALL
```
### 1.8
```
nano etc/ssh/sshd_config
```
- megnyitott konfigba fel kell venni a köv sort:
```
PermitRootLogin no
```
### 1.10
```
apt-get install mysql-server
mysql_secure_installation (a jelszó legyen root)
mysql -u root -p
update mysql.user set authentication_string = password('root') where user = 'root' and host = 'localhost';
update mysql.user set plugin='mysql_native_password' where user = 'root';
flush privileges;
```
### 1.14
```
cd /root
mysql -u root -p < students.sql
```
### 1.16
- az utolsó insertnél csekkold hogy a beszúrt rekordok azok azzal az ID-val jöttek e létre
- vigyázz cseréld ki az insertben a saját NEPTUN-odra a szöveget
```
mysql -u root -p -D students
INSERT INTO students(name,dateofbirth,neptun) VALUES ('Mekk Elek','1974-04-01','NEPTUN');
INSERT INTO courses(name,credit) VALUES ('Mesterseges inteligencia',5);
INSERT INTO results(studentid,courseid,result) VALUES (8,10,5);
```

## 2. feladatcsoport
### 2.1
```
apt-get install apache2
```
### 2.3
```
cd /etc/apache2/sites-available
cp 000-default.conf irulabor.conf
nano irulabor.conf
```
- a megnyitott konfig fálból ki kell tötölni a köv sort:
```
DocumentRoot /var/wwww/index
```
- majd hozzá kel adni a köv 3 sort
```
ServerName irulabor.vmware
ServerAlias *.irulabor.vmware
DocumentRoot /var/wwww/irulabor
```
- menteni a fájlt
```
cd /var/www
mkdir irulabor
chown mekkelek /var/www/irulabor
```
- WinScp-vel be kell lépni mekkelek.el és felmásolni a web mappa tartalmát az irulabor mappaba
```
a2ensite irulabor
systemctl restart apache2
```
### 2.4
```
nano /etc/apache2/sites-available/irulabor.conf
```
- a megnyitott fájlba hozzá kell adni a következő blokkot
```
<Directory /var/www/irulabor/vedett>
    Require all denied
    Require ip 127.0.0.1
</Directory>
```
- menteni a fájlt
```
systemctl restart apache2
```
### 2.5
- a .htpasswd-ben mekkelek.nek a NEPTUN kódod legyen a jelszava
```
mkdir /etc/apache2/passwd
htpasswd -c /etc/apache2/passwd/.htpasswd mekkelek
nano /etc/apache2/sites-available/irulabor.conf
```
- a megnyitott konfig fájlban add hozzá a következő blokkot:
```
<Directory /var/www/irulabor/vedett>
    Options Indexes FollowSymLinks MultiViews
    AllowOverride None
    Require all denied
    <RequireAll>
        Require ip 127.0.0.1
        Require valid-user
    </RequireAll>
    AuthType Basic
    AuthName "IRULabor"
    AuthUserFile /etc/apache2/passwd/.htpasswd
</Directory>
```
- mentsd el a szerkesztett configot
```
systemctl restart apache2
```
### 2.6
```
apt-get install pwauth
sudo a2enmod authnz_external
nano /etc/apache2/sites-available/irulabor.conf
```
- a megnyitott konfig fájlba írd bele a következő blokkokat
```
<IfModule mod_authnz_external.c>
    AddExternalAuth pwauth /usr/sbin/pwauth
    SetExternalAuthMethod pwauth pipe
</IfModule>

<Directory /var/www/irulabor/nagyonvedett>
    Options Indexes FollowSymLinks MultiViews
    AllowOverride None
    AuthType Basic
    AuthName "IRULabor - nagyonvedett"
    AuthBasicProvider external
    AuthExternal pwauth
    Require valid-user
</Directory>
```
- mentsd el a szerkesztett configot
```
systemctl restart apache2
```
### 2.8
```
nano /etc/apache2/sites-available/irulabor.conf
```
- a megnyitott konfig fájlba írd bele a következő blokkokat
```
<Directory /var/www/irulabor/nyilvanos>
    Options Indexes FollowSymLinks MultiViews
    AllowOverride All
    Require all granted
</Directory>
```
- mentsd el a szerkesztett configot
```
systemctl restart apache2
```
### 2.9
```
nano /var/www/irulabor/nyilvanos/.htaccess
```
- töröld ki a fájl tartalmát. majd írd bele a köv sort:
```
Options +Indexes
```
- mentsd el a fájlt
```
systemctl restart apache2
```


