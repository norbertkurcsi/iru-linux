# IRU Labor - Linux (A csoport)
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
Az alábbi parancsoknál az ip-t ki kell cserélni arra az ip-re amely a virtuális gép helyi hálozata.
```bash
ip addr
iptables -A INPUT -p tcp --dport 22 -s '192.168.193.0/24' -j ACCEPT
iptables -A INPUT -p tcp --dport 22 -j DROP

# -A append to chain hozzáadja a lánchoz a megadott filtert
#       INPUT a lánc neve amihez hozzáfűzünk, a rendszerbe bejövő kérésekkel foglalkozik
# -p megadja a protokollt amire a filter illeszkedni fog
# -s a source ip cím
# -j jump a target amire ugrani fog ACCEPT vagy DROP
#       ACCEPT továbbengedi a csomagot
#       DROP eldobja a csomagot
```

### 1.5
```bash
iptables -A INPUT -p icmp --icmp-type echo-request -j DROP
```

### 1.6
```bash
adduser mekkelek --gecos "" --disabled-password
passwd mekkelek

# adduser Hozzáad egy felhasználót a rendszerhez
#       --gecos "" Nem adunk meg plusz adatokat a felhasználóról
#       --disabled-password Nem fogunk egyből egy jelszót rendelni a felhasználóhoz, addig nem is lehet vele belépni amíg ezt meg nem tesszük.
# passwd Itt adjuk meg a jelszót a felhasználóhoz
```

### 1.7
```bash
nano etc/sudoers

# Nem minden felhasználónak van joga hozzá hogy rendszergazda üzemmódban futtathasson programokat, a felhasználók amelyek rendszergazdaként futtathatnak parancsokat az /etc/sudoers fájlban vannak, ezt a fájlt kell kiegészíteni egy olyan sorral mint amilyen a root-nak is van:
# mekkelek  All=(ALL:ALL) ALL
```

### 1.8
```bash
nano etc/ssh/sshd_config

# A fájl tartalmazza az ssh démon (server) configurációját, itt kell felvenni a következő sort:
# PermitRootLogin no
```

### 1.10
```bash
apt-get install -y mysql-server # Telepíti a mysql servert
mysql_secure_installation # a jelszó legyen root
mysql -u root -p
```

```sql
UPDATE mysql.user SET authentication_string = PASSWORD('root') WHERE user = 'root' and host = 'localhost';
UPDATE mysql.user SET plugin='mysql_native_password' WHERE user = 'root';
FLUSH PRIVILEGES;
```

### 1.14
```bash
mysql -u root -p < /root/students.sql
```

## 2. feladatcsoport

### 2.1
```bash
apt-get install -y apache2
```

### 2.3
```bash
mkdir /var/www/irulabor
cd /etc/apache2/sites-available
cp 000-default.conf irulabor.conf
nano irulabor.conf
```

Követkekező beállításokat kell beállítani
```
ServerName irulabor.vmware
ServerAlias *.irulabor.vmware
DocumentRoot /var/www/irulabor
```

Fel kell másolni a web mappa tartalmát az irulabor mappaba.

```bash
a2ensite irulabor
systemctl restart apache2
```

### 2.4
```bash
nano /etc/apache2/sites-available/irulabor.conf
```

Következő sorokat kell hozzáadni:
```
<Directory /var/www/irulabor/vedett>
    Require all denied
    Require ip 127.0.0.1
</Directory>
```

Újraindítani: `systemctl restart apache2`

### 2.5
- a .htpasswd-ben mekkelek.nek a NEPTUN kódod legyen a jelszava
```bash
mkdir /etc/apache2/passwd
htpasswd -c /etc/apache2/passwd/.htpasswd mekkelek
nano /etc/apache2/sites-available/irulabor.conf
```

Következő sorokat kell hozzáadni *(a \<Directory>-n belül az első három sor szerintem nem szükséges)*:
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

Újraindítani: `systemctl restart apache2`

### 2.6
```bash
apt-get install pwauth
sudo a2enmod authnz_external
nano /etc/apache2/sites-available/irulabor.conf
```

Következő sorokat kell hozzáadni:
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

Újraindítani: `systemctl restart apache2`

### 2.8
```bash
nano /etc/apache2/sites-available/irulabor.conf
```

Következő sorokat kell hozzáadni:
```
<Directory /var/www/irulabor/nyilvanos>
    Options Indexes FollowSymLinks MultiViews
    AllowOverride All
    Require all granted
</Directory>
```

Újraindítani: `systemctl restart apache2`

### 2.9
```bash
nano /var/www/irulabor/nyilvanos/.htaccess
```

Következő sort kell hozzáadni:
```
Options +Indexes
```

Újraindítani: `systemctl restart apache2`


## 3. feladatcsoport
- root felhasználót kell használni itt is
- létre kell hozni egy bin mappát a laboruser felhasználónak
```bash
cd /home/laboruser
mkdir bin
cd bin
```

### 3.1
```
nano 3_1.sh
```
Szöveg a nano-ba:
```
lscpu | grep 'Vendor ID' | cut -d ":" -f2 | tr -d '[:space:]'
echo ""
```
Hogy futtatni lehessen:
```
chmod +x 3_1.sh
```

### 3.2
```bash
echo "cat - | grep '^$' | wc -l" >> 3_2.sh
chmod +x "3_2.sh"
```

### 3.3

```bash
cat - | awk '/^[1-9][0-9]*/ && $1 % '$1' == 0 { t = $2; $2 = $5; $5 = t; print }'
```
Futtathatóvá: `chmod +x 3_3.sh`

### 3.4 Ez lehet nem jó
```bash
whoami
date +"%Y. %m. %d."
who | cut -d " " -f1 | sort | uniq
who -b | cut -d" " -f 13-14
echo $$
```

Futtathatóvá: `chmod +x 3_4.sh`

### 3.6
```bash
diff -u $1 $2 | tail -n +3 | grep '^-.*' | wc -l
diff -u $2 $1 | tail -n +3 | grep '^-.*' | wc -l
```

Futtathatóvá: `chmod +x 3_6.sh`

### 3.7
```bash
cat - | awk -F , '
{ if ($1 in array == 0) array[$1] = $2", "$3 }

END {
  for (i in array)
    print i ": " array[i]
}'
```

Futtathatóvá: `chmod +x 3_6.sh`
