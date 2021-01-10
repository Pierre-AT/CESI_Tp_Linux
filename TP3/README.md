# Mise en place d'un DNS

Création de la VM (2 interface réseau)

Mise en place des interface
```
ip a 
ens33 = no ip
ens34 = no ip
```

```
sudo ifup ens33
```

vi /etc/sysconfig/network-script/ifcfg-ens34
```
NAME=ens37
DEVICE=ens37
```

```
BOOTPROTO=static
ONBOOT=yes

IPADDR=10.99.99.14
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
```

Connection successfully activated 

Modification du nom et du domaine :
`vi /etc/hosts`
modifier le localhost par "cesi"
```
vi /etc/hostname
```

```
sudo yum install bind -y
sudo yum install bond-utils -y
sudo yum install epel-release -y
secceeded
```


sudo vim /etc/named.conf
```
//
// named.conf
//
// Provided by Red Hat bind package to configure the ISC BIND named(8) DNS
// server as a caching only nameserver (as a localhost DNS resolver only).
//
// See /usr/share/doc/bind*/sample/ for example named configuration files.
//
// See the BIND Administrator's Reference Manual (ARM) for details about the
// configuration located in /usr/share/doc/bind-{version}/Bv9ARM.html

options {
	listen-on port 53 { 127.0.0.1; 10.99.99.14; };
		listen-on-v6 port 53 { ::1; };
			directory 	"/var/named";
				dump-file 	"/var/named/data/cache_dump.db";
					statistics-file "/var/named/data/named_stats.txt";
						memstatistics-file "/var/named/data/named_mem_stats.txt";
							recursing-file  "/var/named/data/named.recursing";
								secroots-file   "/var/named/data/named.secroots";
									allow-query     { 10.99.99.0/24; };

										/* 
											 - If you are building an AUTHORITATIVE DNS server, do NOT enable recursion.
											 	 - If you are building a RECURSIVE (caching) DNS server, you need to enable 
												 	   recursion. 
													   	 - If your recursive DNS server has a public IP address, you MUST enable access 
														 	   control to limit queries to your legitimate users. Failing to do so will
															   	   cause your server to become part of large scale DNS amplification 
																   	   attacks. Implementing BCP38 within your network would greatly
																	   	   reduce such attack surface 
																		   	*/
																				recursion yes;

																					dnssec-enable yes;
																						dnssec-validation yes;

																							/* Path to ISC DLV key */
																								bindkeys-file "/etc/named.root.key";

																									managed-keys-directory "/var/named/dynamic";

																										pid-file "/run/named/named.pid";
																											session-keyfile "/run/named/session.key";
																											};

																											logging {
																											        channel default_debug {
																												                file "data/named.run";
																														                severity dynamic;
																																        };
																																	};

																																	zone "." IN {
																																		type hint;
																																			file "named.ca";
																																			};

																																			zone "tp3.cesi" IN {
																																			         
																																				          type master;
																																					          
																																						           file "/var/named/tp3.cesi.db";

																																							            allow-update { none; };
																																								    };

																																								    zone "99.99.10.in-addr.arpa" IN {
																																								              
																																									                type master;
																																											          
																																												            file "/var/named/99.99.10.db";
																																													             
																																														               allow-update { none; };
																																															       };

																																															       include "/etc/named.rfc1912.zones";
																																															       include "/etc/named.root.key";
																																															       ```


																																															       sudo vi /etc/named/tp3.cesi.db
																																															       ```
																																															       $TTL    604800
																																															       @   IN  SOA     ns1.cesi. root.cesi. (
																																															                                                       1001    ;Serial
																																																					                                                       3H      ;Refresh
																																																											                                                       15M     ;Retry
																																																																	                                                       1W      ;Expire
																																																																							                                                       1D      ;Minimum TTL
																																																																													                                                       )

																																																																																			       ;Name Server Information
																																																																																			       @ IN  NS      ns1.tp3.cesi.

																																																																																			       14.99.99.10 IN PTR ns1.tp3.cesi.


																																																																																			       ;PTR Record IP address to HostName
																																																																																			       11      IN  PTR     web.cesi.
																																																																																			       12      IN  PTR     db.cesi.
																																																																																			       13      IN  PTR     rp.cesi.

																																																																																			       sudo vi /var/named/99.99.10.db
																																																																																			       $TTL    604800
																																																																																			       @   IN  SOA     ns1.cesi. root.cesi. (
																																																																																			                                                       1001    ;Serial
																																																																																									                                                       3H      ;Refresh
																																																																																															                                                       15M     ;Retry
																																																																																																					                                                       1W      ;Expire
																																																																																																											                                                       1D      ;Minimum TTL
																																																																																																																	                                                       )
																																																																																																																							       ```
																																																																																																																							       sudo vi /var/named/99.99.10.db
																																																																																																																							       ```
																																																																																																																							       ;Name Server Information
																																																																																																																							       @ IN  NS      ns1.cesi.

																																																																																																																							       14.99.99.10 IN PTR ns1.cesi.

																																																																																																																							       ;PTR Record IP address to HostName
																																																																																																																							       11      IN  PTR     web.cesi.
																																																																																																																							       12      IN  PTR     db.cesi.
																																																																																																																							       13      IN  PTR     rp.cesi.
																																																																																																																							       ```


																																																																																																																							       Ouverture des port sur le pare-feu
																																																																																																																							       ```
																																																																																																																							       sudo firewall-cmd --zone=public --add-port=53/udp --permanent
																																																																																																																							       success
																																																																																																																							       [pierre@ns1 ~]$ sudo firewall-cmd --zone=public --add-port=53/tcp --permanent
																																																																																																																							       success
																																																																																																																							       ```

																																																																																																																							       `systemctl restart named `

																																																																																																																							       ---------
																																																																																																																							       Sur un autre VM

																																																																																																																							       Installation des outils DNS
																																																																																																																							       ```
																																																																																																																							       sudo yum install bind-utils
																																																																																																																							       complete!
																																																																																																																							       ```
																																																																																																																							       Test :
																																																																																																																							       ```
																																																																																																																							       dig web.cesi
																																																																																																																							       SERVER: 192.168.73.2#53(192.168.73.2)
																																																																																																																							       ```

																																																																																																																							       ```
																																																																																																																							       dig -x 10.99.99.13
																																																																																																																							       SERVER: 192.168.73.2#53(192.168.73.2)
																																																																																																																							       ```

																																																																																																																							       Le DNS fonctionne !

																																																																																																																							       Optionnel :
																																																																																																																							       Ajout de facon permanente :
																																																																																																																							       ```
																																																																																																																							       vi /etc/resolv.conf 
																																																																																																																							       nameserver 10.99.99.14
																																																																																																																							       ```




																																																																																																																							       -------------------

																																																																																																																							       # Mise en place d'un Backup

																																																																																																																							       Déja fais en entreprise

																																																																																																																							       -------------------
																																																																																																																							       # Mise en place de Docker

																																																																																																																							       Déinstallation de Httpd et Apache
																																																																																																																							       ```
																																																																																																																							       sudo yum erase httpd httpd-tools apr apr-util -yes succeeded
																																																																																																																							       ```

																																																																																																																							       Déinstallation de Wordpress
																																																																																																																							       ```
																																																																																																																							       cd /var/www/html
																																																																																																																							       sudo rm -rf *
																																																																																																																							       cd
																																																																																																																							       yum remove mariadb-server
																																																																																																																							       mysql -u root -p 
																																																																																																																							       show databases;
																																																																																																																							       drop database wordpress;
																																																																																																																							       ```


																																																																																																																							       Installation 
																																																																																																																							       ```
																																																																																																																							       sudo yum install -y yum-utils
																																																																																																																							       sudo yum-config-manager \
																																																																																																																							           --add-repo \
																																																																																																																								       https://download.docker.com/linux/centos/docker-ce.repo

																																																																																																																								       repo saved to /etc/yum.repos.d/docker-ce.repo
																																																																																																																								       ```

																																																																																																																								       ```
																																																																																																																								       sudo yum install docker-ce docker-ce-cli containerd.io -y 
																																																																																																																								       succeeded
																																																																																																																								       ```


																																																																																																																								       Création d'un docker :
																																																																																																																								       ```
																																																																																																																								       sudo docker run --name cesi-wp -d \
																																																																																																																								         -e WORDPRESS_DB_HOST=10.99.99.12 \
																																																																																																																									   -e WORDPRESS_DB_USER=cesi \
																																																																																																																									     -e WORDPRESS_DB_PASSWORD=cesi \
																																																																																																																									       wordpress
																																																																																																																									       Docker OK 
																																																																																																																									       ```


																																																																																																																									       Commande à savoir : (Docker)
																																																																																																																									       * docker ps = Liste les conteneur en ligne
																																																																																																																									       * docker image = permet de voir les images crée ou récupérer
																																																																																																																									       * docker hub = Makerplace d'image docker 
																																																																																																																									       * docker pull = télécharge les image 
																																																																																																																									       * docker run = lance limage


