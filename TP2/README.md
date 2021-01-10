# TP 2 - Serveur web qui nécessite une base de données, une base de données, un reverse proxy qui permet d'augmenter le niveau de sécurité global

Création des 3 VMs avec duplication

Ajout de la carte réseau x3

ifup sur toute les cartes réseau  x3
```
sudo ifup ensXX
> Connection sucess...
```

Connection en SSH depuis MobaXterm

Modification du nom de domaine x3
```
sudo vi /etc/hosts
127.0.0.1   localhost localhost.gema rp.tp2.cesi rp.tp2.cesi.cesi.gema
::1         localhost localhost.gema rp.tp2.cesi rp.tp2.cesi.cesi.gema
```
Modification nom serveur
```
/etc/hostname
localhost.gema
```


Configuration réseau :
Ajouté une carte NAT (DHCP ON - réseau aléatoire)
Ajouté une carte localhost (DHCP OFF - 10.99.99.0 / 24)

```
ip a
sudo ifup ens 33 (si elle est pas acctivé)
sudo vim /etc/sysconfig/network-scripts/ifcfg-ens33
ONBOOT=yes (pour acctivé au démarrage)
```

sudo vi /etc/sysconfig/network-scripts/ifcfg-ens37 (paramétrage de la carte localhost)
```
NAME=ens37
DEVICE=ens37

BOOTPROTO=static
ONBOOT=yes

IPADDR=10.99.99.12
NETMASK=255.255.255.0

La suite est optionnelle
GATEWAY=10.99.99.1
DNS1=1.1.1.1
```

redémarrer la carte après 
```
sudo ifdown ens37
sudo ifup ens37

Connection successfully activated 
```

SELinux doit toujours être désactivé
```
$ sudo setenforce 0
$ sudo sed -i 's/SELINUX=enforcing/SELINUX=permissive/g' /etc/selinux/config
```


-------

# Installtion de la base de donnée
`yum install mariadb-server -yes`

`Complete!`

Activation de mariadb-server
```
systemctl enable mariadb
systemctl start mariadb
systemctl status mariadb
```

```
mariadb.service - MariaDB database server
Loaded: loaded (/usr/lib/systemd/system/mariadb.service; enabled; vendor preset: disabled)
Active: active (running) since Wed 2021-01-06 06:02:38 EST; 1s ago
```

Connection à mysql
`myslq -u root p`
Voir les tables
`show tables`

création d'une database
`CREATE DATABASE bdd`

```
Query OK, 1 row affected (0.00 sec)
```


création d'une table :
```
CREATE TABLE utilisateur
(
id INT PRIMARY KEY NOT NULL,
nom VARCHAR(100),
prenom VARCHAR(100),
email VARCHAR(255),
date_naissance DATE,
pays VARCHAR(255),
ville VARCHAR(255),
code_postal VARCHAR(5),
nombre_achat INT
)

Query OK, 1 row affected (0.00 sec)
```

création user :
```
CREATE USER 'pierre'@'localhost' IDENTIFIED BY 'abc';
GRANT ALL PRIVILEGES ON *.* TO 'pierre'@'localhost'
WITH GRANT OPTION;

Query OK, 0 rows affected (0.00 sec)

CREATE USER 'pierre'@'%' IDENTIFIED BY 'abc';
GRANT ALL PRIVILEGES ON *.* TO 'pierre'@'%'
WITH GRANT OPTION;

Query OK, 0 rows affected (0.00 sec)
```

------------

**Ouverture de port de la bdd**
Trouvé le port : 
```
sudo ss -alnpt
mysql = *:3306
```

```
firewall-cmd --zone=public --add-port=3306/tcp --permanent
success
firewall-cmd --reload
success
```

--------
Test de connectivité 
```
mysql -u pierre -p -h 10.99.99.12 -P 3306
ok
```


`show databases;`
+--------------------+
| Database           |
+--------------------+
| information_schema |
| ------------------ |
| bdd                |
| mysql              |
| performance_schema |
| **test**           |
```
+--------------------+
5 rows in set (0.00 sec)
```

------------
# Serveur WEB
Installation du service
```
sudo yum install apache2
sudo systemctl start httpd.service
```

ouverture du port
```
sudo firewall-cmd --permanent --zone=public --add-service=http 

sudo firewall-cmd --permanent --zone=public --add-service=https

sudo firewall-cmd --reload
sucess
```

Le site web est up !

# Installation php
```
sudo yum install http://rpms.remirepo.net/enterprise/remi-release-7.rpm -y
sudo yum install -y yum-utils
```

On supprime d'éventuelles vieilles versions de PHP précédemment installées
```
sudo yum remove -y php
```

Activation du nouveau dépôt
```
sudo yum-config-manager --enable remi-php56  
```

Installation de PHP 5.6.40 et de librairies récurrentes
```
sudo yum install php php-mcrypt php-cli php-gd php-curl php-mysql php-ldap php-zip php-fileinfo -y
```

```
Zend Engine v3.3.26, Copyright (c) 1998-2018 Zend Technologies
```

pour validé : 
`sudo vi /var/www/html/info.php -> <?php phpinfo(); ?> -> http://adresseIPduServeur/info.php`

# Mise en place de Wordpress :

Création d'un user / bdd

```
mysql> CREATE USER 'web'@'localhost' IDENTIFIED BY 'abc';
mysql> GRANT ALL PRIVILEGES ON *.* TO 'monty'@'localhost'
->     WITH GRANT OPTION;
mysql> CREATE USER 'web'@'%' IDENTIFIED BY 'abc';
mysql> GRANT ALL PRIVILEGES ON *.* TO 'web'@'%'
->     WITH GRANT OPTION;

FLUSH PRIVILEGES;
exit
```

**Installtion de Wordpress :**
```
sudo yum install php-gd
Sudo  yum install wget
success 
wget http://wordpress.org/latest.tar.gz 
100%
```

Décompréser l'archive
`cp latest.tar.gz /var/www/html`
`tar xzvf /var/www/html/latest.tar.gz`
`sudo mkdir /var/www/html/wordpress/wp-content/uploads`

Mise en place des permision 
```
sudo chown -R apache:apache /var/www/html/*
```

Confiuration de WP
```
cd /var/www/html/wordpress
cp wp-config-sample.php wp-config.php
vi wp-config.php
```


-----------

# Reverse proxy

Installation du service
`sudo yum install epel-release`

```
sudo yum install nginx
systemctl enable nginx 
systemctl start nginx
systemctl status nginx
successfully
```

Configuration du fichier :
sudo vi /etc/nginx/nginx.conf
```
server {
listen       80 default_server;
listen       [::]:80 default_server;
server_name  web.gema;
root         /usr/share/nginx/html;

# Load configuration files for the default server block.
include /etc/nginx/default.d/*.conf;

location / {
proxy_pass      http://10.99.99.11/wordpress/;

```

Ouverture de port : 
```
sudo ss -alnpt = port nginx 80
firewall-cmd --zone=public --add-port=80/tcp --permanent
firewall-cmd --reload
success
```

**Le site est accèsible depuis le server proxy !**

configuration de la bdd vers le web 
`sudo vi wp-config.php`


```
// ** MySQL settings - You can get this info from your web host ** //
/** The name of the database for WordPress */
define( 'DB_NAME', 'wordpress' );

/** MySQL database username */
define( 'DB_USER', 'pierre' );

/** MySQL database password */
define( 'DB_PASSWORD', 'abc' );

/** MySQL hostname */
define( 'DB_HOST', '10.99.99.12' );

/** Database Charset to use in creating database tables. */
define( 'DB_CHARSET', 'utf8' );

/** The Database Collate type. Don't change this if in doubt. */
define( 'DB_COLLATE', '' );
```

**Connexion réussis !**

--------------------------------------------------

# FAIL2BAN :
Installation et lancement du service
```
sudo yum install epel-release
sudo yum install fail2ban
sudo systemctl enable fail2ban
```
COnfiguration du fichier
`sudo nano /etc/fail2ban/jail.local`
```
[DEFAULT]
# Ban hosts for one hour:
bantime = 3600

# Override /etc/fail2ban/jail.d/00-firewalld.conf:
banaction = iptables-multiport

[sshd]
enabled = true
sudo systemctl restart fail2ban

sudo systemctl restart fail2ban
sudo fail2ban-client status

/etc/fail2ban/fail2ban.conf

Lire les logs ssh :
[sshd]
# To use more aggressive sshd modes set filter parameter "mode" in jail.local:
# normal (default), ddos, extra or aggressive (combines all).
# See "tests/files/logs/sshd" or "filter.d/sshd.conf" for usage example and details.
#mode   = normal
port    = ssh
logpath = %(sshd_log)s
backend = %(sshd_backend)s

Configuration pour les retry :
[sshd]
enabled = true <- activé la vérif sshd
maxretry = 10 <- retry max
findtime = 120 <- laps de temps pendant lequel on considère les occurences
bantime = 1200

Configuration du parfeau avec FAIL2BAN pour bloqué les ips qui veulent ce conencté :
vi /etc/fail2ban/jail.local

[sshd]
enabled = true
sudo systemctl start fail2ban.service
sudo systemctl enable fail2ban.service
succes
```
Voir la régle
`sudo firewall-cmd --direct --get-all-rules`
Voir les tentative
`sudo ipset list fail2ban-sshd`

----------

# HTTPS :

Mise en place su TLS sur le reverser proxy
```
sudo openssl req -new -newkey rsa:2048 -days 365 -nodes -x509 -keyout web.cesi.key -out web.cesi.crt
Generating a 2048 bit RSA private key
..............................................................................+++
..........................................................+++
writing new private key to 'web.cesi.key'
-----
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [XX]:FR
string is too long, it needs to be less than  2 bytes long
Country Name (2 letter code) [XX]:FR
State or Province Name (full name) []:France
Locality Name (eg, city) [Default City]:Perigueux
Organization Name (eg, company) [Default Company Ltd]:cesi
Organizational Unit Name (eg, section) []:cesi
Common Name (eg, your name or your server's hostname) []:web.gema
Email Address []:
```


Vérification des clé 
```
ls /etc/nginx/
web.cesi.crt web.cesi.key
```

Modification du fichier conf 
sudo vi /etc/nginx/nginx.conf

```
events {}

http {
server {
listen       443 ssl;

ssl on;

ssl_certificate /etc/nginx/web.cesi.crt;
ssl_certificate_key /etc/nginx/web.cesi.key;

server_name web.gema;

location / {
proxy_pass   http://10.99.99.11/wordpress/;
}
}
}
```
Relancé le service
`systemctl restart nginx`

--------------
# installation de Netdata 
Télécharger Netdata
`bash <(curl -Ss https://my-netdata.io/kickstart.sh) <installattion`
Configuration du port
```
firewall-cmd --zone=public --add-port=19999/tcp --permanent

sudo firewall-cmd --reload
```

Configuration alterte discord :
```
cd /etc/netdata/
sudo ./edit-config health_alarm_notify.conf
```
Ajout du serveur dans le fichier de conf
serveur -> `795956179544834078`



------

# Interface Web :
```
sudo yum install -y epel-release
nothing to do <- déja installé
```
Installation du service
```
sudo yum install -y cockpit
Complete!
```

Ouverture du port
```
sudo firewall-cmd --zone=public --add-port=9090/tcp --permanent
succes
sudo firewall-cmd --reload
success
```
Relance du service 
`systemctl start cockpit`

vérification sur le navigateur
url = 10.99.99.11:9090
**Site accès interface est up !**


