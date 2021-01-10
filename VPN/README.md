# Mise en place d'un VPN
Création de la VM (2 interfaces réseaux)

Mise en place des interface
```
ip a 
ens33 = no ip
ens34 = no ip
```

Allumage de la carte
`sudo ifup ens33`

```
vi /etc/sysconfig/network-script/ifcfg-ens34
NAME=ens37
DEVICE=ens37

BOOTPROTO=static
ONBOOT=yes

IPADDR=10.99.99.14
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

Modification du nom et du domaine :
`vi /etc/hosts`
modifier le localhost par "cesi"
`vi /etc/hostname`

Téléchargement d'OpenVPN
```
cd
curl -SLO https://github.com/OpenVPN/easy-rsa/releases/download/v3.0.8/EasyRSA-3.0.8.tgz
100%
```

`mkdir ~/ca` <- création du dossier dédier a CA

```
ls 
ca  EasyRSA-3.0.8.tgz
```
Décompréser l'archige
`tar xvzf EasyRSA-3.0.8.tgz` 

```
ls ca
ChangeLog   doc      gpl-2.0.txt  openssl-easyrsa.cnf  README.quickstart.md  x509-types
COPYING.md  easyrsa  mktemp.txt   README.md            vars.example
```

mise en place de la CA
`cd ~/ca`
`cp vars.example vars`

```
vi vars
set_var EASYRSA_REQ_COUNTRY     "FR"
set_var EASYRSA_REQ_PROVINCE    "France"
set_var EASYRSA_REQ_CITY        "Dordogne"
set_var EASYRSA_REQ_ORG         "CESI"
set_var EASYRSA_REQ_EMAIL       "pierre.authier.pro@gmail.com"
set_var EASYRSA_REQ_OU          "bap"
```

`cd ~/ca`
`./easyrsa init-pki`
```
Using SSL: openssl OpenSSL 1.0.2k-fips  26 Jan 2017
Generating RSA private key, 2048 bit long modulus
```



`cd ~/ca`
```
./easyrsa build-ca nopass
```
```
CA creation complete and you may now import and sign cert requests.
```

OpenVPN serveur
```
sudo yum install -y epel-release
Complete!
```

```
sudo yum install -y openvpn
Complete!
```

`cd`
Récupération de EasyRSA. Dernière version à jour au 06/01/2021.
`curl -SLO https://github.com/OpenVPN/easy-rsa/releases/download/v3.0.8/EasyRSA-3.0.8.tgz`

Création d'un répertoire dédié au serveur VPN
`mkdir ~/ovpn`

Extraction de l'archive récupérée avec le curl
`tar xvzf EasyRSA-3.0.8.tgz`

`mv EasyRSA-3.0.8/* ovpn/`

`cd ~/ovpn`
```
./easyrsa init-pki
init-pki complete;
```






***PAS FINI !***
