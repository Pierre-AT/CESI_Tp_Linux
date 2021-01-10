# **Commande TP1**

ifup sur toutes les cartes réseau  x3
sudo ifup ensXX
> Connection sucess...

Connection en SSH depuis MobaXterm

Modification du nom de domaine x3
sudo vi /etc/hosts
```
127.0.0.1   localhost localhost.gema rp.tp2.cesi rp.tp2.cesi.cesi.gema
::1         localhost localhost.gema rp.tp2.cesi rp.tp2.cesi.cesi.gema
```


Configuration réseau :
Ajouté une carte NAT (DHCP ON - réseau aléatoire)
Ajouté une carte localhost (DHCP OFF - 10.99.99.0 / 24)

```
ip a
sudo ifup ens 33 (si elle est pas acctivé)
sudo vim /etc/sysconfig/network-scripts/ifcfg-ens33 : 
ONBOOT=yes (pour acctivé au démarrage)
```

sudo vi /etc/sysconfig/network-scripts/ifcfg-ens37 (paramétrage de la carte localhost)
```
NAME=ens37
DEVICE=ens37
```

```
BOOTPROTO=static
ONBOOT=yes
```

```
IPADDR=10.99.99.12
NETMASK=255.255.255.0
```

La suite est optionnelle
```
GATEWAY=10.99.99.1
DNS1=1.1.1.1
```

redémarrer la carte après 
```
sudo ifdown ens37
sudo ifup ens37
Connection successfully activated 
```




Optionel : Ajout d'un DNS
sudo vim /etc/resolv.conf
`nameserver 1.1.1.1`

Modification du nom du serveur
vim /etc/hostname
```
tp1.vm.cesi
```

Mise a jour des paquets
```
apt yum update -y`
Complete
```

Installer vim sur la machine afin de faciliter l'édition des fichiers.
```
sudo yum install vim
Complete !
```
Rendre le systeme linux plus 'Permissive'
```
sudo vim /etc/selinux/config
LINUX = permisive
ou sudo sed -i 's/SELINUX=enforcing/SELINUX=permissive/g' /etc/selinux/config
```
Création d'un user nommé toto :

```
sudo adduser toto -m -s /bin/bash
sudo groupadd admins
```

Mise en place des droits admin

```
sudo visudo <- modifcation du fichier sudo
admins  ALL=(ALL)       ALL
```
Ajout du groupe
`sudo usermod -aG admins toto`

Création de la rsa
```
ssh-keygen -t rsa -b 4096
mkdir .ssh
cd .ssh
```
Clé publique

`vim authorized_key`


Mise en place des droits sur les fichiers
```
chmod 600 /home/user/.ssh/authorized_keys
chmod 700 /home/user/.ssh
```




Ajout de 2 disque
Commande pour créer les partitions et les ajouté
```
lsblk
sudo pvcreate /dev/sdb
sudo pvcreate /dev/sdc
```

Vérif
`sudo pvs`

`sudo vgcreate data /dev/sdb /dev/sdc`

Vérif
`sudo vgs`

```
sudo lvcreate -L 2G data -n part1
sudo lvcreate -L 2G data -n part2
sudo lvcreate -L 1.99G data -n part3

mkfs -t ext4 /dev/data/part1
mkfs -t ext4 /dev/data/part2
mkfs -t ext4 /dev/data/part3

mkdir /mnt/part1
mkdir /mnt/part2
mkdir /mnt/part3

sudo mount /dev/data/part1 /mnt/part1
sudo mount /dev/data/part2 /mnt/part2
sudo mount /dev/data/part3 /mnt/part3

vim /etc/fstab

/dev/data/part1 /mnt/part1 ext4 defaults 0 0
/dev/data/part3 /mnt/part2 ext4 defaults 0 0
/dev/data/part3 /mnt/part3 ext4 defaults 0 0
```

Vérif
`sudo umount /mnt/part1`            = démonter la partition si elle est déjà montée
`sudo mount -av `                            = remonter la partition, en utilisant les infos renseignées dans /etc/fstab
`sudo reboot`                                    = non nécessaire si le mount -av fonctionne correctement

`mkfs.ext4 /dev/sdb1`


Activer le pare-feu
```
systemctl is-active firewalld
sudo systemctl enable firewalld
```

sudo vim web.service
```
[Unit]
Description=Very simple web service

[Service]
ExecStart=/bin/python2 -m SimpleHTTPServer 8888

[Install]
WantedBy=multi-user.target

sudo firewall-cmd --add-port=80/tcp --permanent
sudo firewall-cmd --reload
```


```
sudo adduser web
sudo vim web.service
User=web
WorkingDirectory = /srv/web
```

