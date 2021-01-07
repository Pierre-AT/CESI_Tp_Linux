apt yum update -y
Complete!

Installer vim sur la machine afin de faciliter l'édition des fichiers.
sudo yum install vim
Complete !

sudo vim /etc/selinux/config
LINUX = permisive
ou sudo sed -i 's/SELINUX=enforcing/SELINUX=permissive/g' /etc/selinux/config

sudo adduser toto -m -s /bin/bash
sudo groupadd admins

sudo visudo
admins  ALL=(ALL)       ALL

sudo usermod -aG admins toto

création de la rsa
ssh-keygen -t rsa -b 4096
mkdir .ssh
cd .ssh
vim authorized_key
clé publique
:wq
reboot
chmod 600 /home/user/.ssh/authorized_keys
chmod 700 /home/user/.ssh


sudo vim /etc/resolv.conf
nameserver 1.1.1.1
:wq

vim /etc/hostname
:wq

Ajout de 2 disque
lsblk
sudo pvcreate /dev/sdb
sudo pvcreate /dev/sdc

Vérif
sudo pvs

sudo vgcreate data /dev/sdb /dev/sdc

Vérif
sudo vgs

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

Vérif
sudo umount /mnt/part1            # démonter la partition si elle est déjà montée
sudo mount -av                             # remonter la partition, en utilisant les infos renseignées dans /etc/fstab
sudo reboot                                    # non nécessaire si le mount -av fonctionne correctement




mkfs.ext4 /dev/sdb1


systemctl is-active firewalld
sudo systemctl enable firewalld

sudo vim web.service
[Unit]
Description=Very simple web service

[Service]
ExecStart=/bin/python2 -m SimpleHTTPServer 8888

[Install]
WantedBy=multi-user.target

sudo firewall-cmd --add-port=80/tcp --permanent
sudo firewall-cmd --reload


sudo adduser web
sudo vim web.service
User=web
WorkingDirectory = /srv/web

