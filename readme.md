# B1 Réseau 2018 - TP6

## TP 6 - Une topologie qui ressemble un peu à quelque chose, enfin ?

## Lab 1 : Simple OSPF

Petit lab simple pour comprendre le concept.

## Lab 2 : Un peu de complexité (et d'utilité ?...)

### 2. Mise en place du lab

* Définition des IPs statiques :
    * Sur ```r1.tp6.b1```, ```r2.tp6.b1```, ```r3.tp6.b1```, ```r4.tp6.b1``` et ```r5.tp6.b1``` utilisez les commandes suivantes :
        * ```show ip int br```
        * ```conf t```
        * ```interface ethernet <NUMERO>```
        * ```ip address <IP> <MASK>```
        * ```no shut```
        * ```exit```
        * Et enfin : ```exit```
    * Vérifiez avec la commande : 
        * ```show ip int br```

* Définition d'un nom de domaine :
    * Sur ```r1.tp6.b1```, ```r2.tp6.b1```, ```r3.tp6.b1```, ```r4.tp6.b1``` et ```r5.tp6.b1``` utilisez les commandes suivantes :
        * ```conf t```
        * ```hostname <HOSTNAME>```
        * ```exit```
        
* Enlevez la carte NAT de ```client1.tp6.b1``` :
    * Allez dans Virtualbox
    * Cliquez sur votre machine puis sur ```Configuration```
    * Allez dans la catégorie ```Réseau```
    * Ensuite dans l'onglet ```Carte 1```, vérifiez que ```Activer la carte réseau``` soit coché et que dans ```Mode d'accès réseau``` il y a ```Réseau privé hôte``` (si ce n'est pas le cas faites le)
    * Vérifiez que les autres cartes ne soient pas activées

Réalisez les même manipulations sur ```client2.tp6.b1``` et ```server1.tp6.b1```.

* Définir les IPs statiques :
    * Sur ```client1.tp6.b1```, ```client2.tp6.b1``` et ```server1.tp6.b1``` utilisez les commandes suivantes :
        * ip a
        * sudo nano /etc/sysconfig/network-scripts/ifcfg-enp0s8
        * Remplir avec comme sur l'[Annexe 1](###Annexe1:) mais avec vos paramètres :
        * sudo ifdown <INTERFACE_NAME>
        * sudo ifup <INTERFACE_NAME>

* Définir le nom de domaine :
    * Sur ```client1.tp6.b1```, ```client2.tp6.b1``` et ```server1.tp6.b1``` utilisez les commandes suivantes :
        * ```sudo hostname <NEW_HOSTNAME>``` (temporaire)
        * OU ```echo 'NEW_HOSTNAME' | sudo tee /etc/hostname``` (permanent)

* Remplir les fichiers hosts :
    * ```sudo nano /etc/hosts```

Vous pouvez vérifier le fonctionnement en utilisant la commande ```ping```.

* Configuration OSPF :
    * Sur ```r1.tp6.b1```, ```r2.tp6.b1```, ```r3.tp6.b1```, ```r4.tp6.b1``` et ```r5.tp6.b1```, utilisez les commandes suivantes :
        * ```conf t```
        * ```router ospf 1```
        * ```router-id 1.1.1.1```
        * ```network 10.6.100.0 0.0.0.3 area 0```

* Vérification :
    * ```show ip route```
    * ```show ip protocols```
    * ```show ip ospf neighbor```
    * ```ping```
    * ```traceroute```

## Lab 3 : Let's end this properly

### 1. NAT : accès internet

* Sur ```r4.tp6.b1``` utilisez les commandes suivantes :

    * ```show ip int br```
    * ```conf t```
    * ```interface fastEthernet 0/0```
    * ```ip address dhcp```
    * ```no shut```
    * Attendez un peu (ou spammez la commande) puis : ```show ip int br```
    * Pour voir la passerelle : ```show ip route```
    * Ping google avec : ```ping 8.8.8.8```

* Maintenant on va définir quelles interface sont à l'intérieur du réseau, et laquelle point vers l'extérieur avec les commandes suivantes :
    * conf t
    * interface fastEthernet 0/0
    * ip nat outside
    * exit
    * interface fastEthernet 1/0
    * ip nat inside
    * exit
    * interface fastEthernet 2/0
    * ip nat inside
    * exit


* Ensuite entrez : 
    * ```conf t```
    * ```ip nat inside source list 1 interface fastEthernet 0/0 overload```
    * ```access-list 1 permit any```

* Maintenant que le NAT fonctionne, il faut partager la route par défaut à tout le monde. Sur nos routeurs c'est désactivé par défaut :
    * ```conf t```
    * ```router ospf 1```
    * ```default-information originate```
    * Et enfin pour vérifier que tous cela fontcionne ```show ip route```

* Nous allons désormais faire des requêtes Web :
    * ```conf t```
    * Activation du lookup DNS:  ```ip domain-lookup```
    * Configuration du serveur DNS (celui de google) : ```ip name-server 8.8.8.8```
    * Requête web vers un site full HTTP, avec résolution de nom: ```exit```
    * Et enfin pour finir  : ```telnet trip-hop.net 80```

### 2. Un service d'infra

* Assurez vous que le firewall est démarré avec les commandes suivantes :
    * ```sudo systemctl status firewalld```
    * ```sudo systemctl start firewalld```

* Ajouter une règle pour le trafic web :
    * ```sudo firewall-cmd --add-port=80/tcp --permanent```
    * ```sudo firewall-cmd --reload```

* Installez le serveur web avec :
    * ```sudo yum install -y epel-release```
    * ```sudo yum install -y nginx```

* Lancez le serveur web avec la commande suivante :
    * ```sudo systemctl start nginx```

* ```(facultatif)``` Si vous voulez personnaliser la super page d'accueil il faut faire : 
    * ```sudo nano /usr/share/nginx/html/index.html```

* Assurez vous que le serveur fonctionne et répond :
    * ```curl localhost```

### 3. Serveur DHCP

* Installez le serveur DHCP avec :
    * ```sudo yum install -y dhcp```

* Remplir le fichier /etc/dhcp/dhcpd.conf avec l'exemple l'[Annexe 2](###Annexe2:)

* Démarrer le serveur DHCP avec la commande :
    * ```sudo systemctl start dhcpd```

* Faire en sorte que le serveur DHCP démarre au boot de la machine :
    * ```sudo systemctl enable dhcpd```

* Faire un test :
    * Sur ```client1.tp6.b1``` :
        * ```sudo nano /etc/sysconfig/network-scripts/ifcfg-enp0s8```
        * Allez à la ligne ```BOOTPROTO=dhcp``` et ```ONBOOT=yes```
        * Entrez les commandes :
            * ```sudo ifdown <INTERFACE_NAME>```
            * ```sudo ifup <INTERFACE_NAME>```

### 4. Serveur DNS

* Sur ```server1.tp6.b1``` :

    * Installation du serveur DNS :
        * ```sudo yum install -y bind*```

    * Edition du fichier de configuration (voir fichier exemples dans ./dns/) :
        * ```sudo nano /etc/named.conf```

    *  Edition des fichiers de zone (voir fichiers exemples dans ./dns/) :
        * ```sudo nano /var/named/forward.tp6.b1```
        * ```sudo nano /var/named/reverse.tp6.b1```

    * Changement du propriétaire des deux fichiers créés à l'étape du dessus pour que le serveur DNS puisse les utiliser :
        * ```sudo chown named:named /var/named/*tp6.b1```

    #### Ouvrir les ports firewall concernés (53/TCP et 53/UDP)

    * Démarrage du service DNS :
        * ```sudo systemctl start named```

    * Faire en sorte que le service démarre au boot de la VM :
        * ```sudo systemctl enable named```

    * Remplir le fichier /etc/resolv.conf pour mettre l'IP de notre nouveau DNS :
        * ```echo "nameserver 10.6.202.10" | sudo tee /etc/resolv.conf```

     #### On peut lui poser des questions avec la commande dig maintenant :

    * A quelle IP correspond "server1.tp6.b1" ?
        * ```dig server1.tp6.b1```

    * A quelle IP correspond "client2.tp6.b1" ?
        * ```dig client2.tp6.b1```

    * A quel nom de domaine correspond l'IP 10.6.201.10 ?
        * ```dig -x 10.6.201.10```
        * Ou : ```ping client2.tp6.b1```


### 5. Serveur NTP

* Sur ```server1.tp6.b1``` :
    * Faire la commande : ```sudo yum install -y chrony```
    * Éditer le fichier ```/etc/chrony.conf``` avec :
        * ```sudo nano /etc/chrony.conf```
    * Y emmetre ce que vous trouverai dans l'[Annexe 3](###Annexe3:)
    * Réaliser les commandes suivantes afin d'ouvrir le port 123/UDP :
        * ```sudo firewall-cmd --list-all```
        * ```sudo firewall-cmd --add-port=123/udp --permanent```
        * ```sudo firewall-cmd --remove-port=123/udp --permanent``` (si on veux fermer le port)
        * ```sudo firewall-cmd --reload```
    * Lancer le service ```chronyd``` :
        * ```sudo systemctl start chronyd```
    * Utiliser les commandes :
        * ```chronyc sources```
        * ```chronyc tracking```

* Sur toutes les autres machines :
    * Éditer le fichier ```/etc/chrony.conf``` avec la commande :
        * ```sudo nano /etc/chrony.conf```
        * Copier/Coller ce qu'il y a dans l'[Annexe 4](###Annexe4:)
    * Réaliser les commandes suivantes afin d'ouvrir le port 123/UDP :
        * ```sudo firewall-cmd --list-all```
        * ```sudo firewall-cmd --add-port=123/udp --permanent```
        * ```sudo firewall-cmd --remove-port=123/udp --permanent``` (si on veux fermer le port)
        * ```sudo firewall-cmd --reload```
    * Lancer le service ```chronyd``` :
        * ```sudo systemctl start chronyd```
### Annexe 1 :
```
NAME=enp0s8  
DEVICE=enp0s8

BOOTPROTO=static  
ONBOOT=yes

IPADDR=192.168.1.19  
NETMASK=255.255.255.0  
GATEWAY=192.168.1.254
```

### Annexe 2 :
```
# dhcpd.conf

# option definitions common to all supported networks
option domain-name "tp5.b1";

default-lease-time 600;
max-lease-time 7200;

# If this DHCP server is the official DHCP server for the local
# network, the authoritative directive should be uncommented.
authoritative;

# Use this to send dhcp log messages to a different log file (you also
# have to hack syslog.conf to complete the redirection).
log-facility local7;

subnet 10.5.2.0 netmask 255.255.255.0 {
  range 10.5.2.50 10.5.2.70;
  option domain-name "tp5.b1";
  option routers 10.5.2.254;
  option broadcast-address 10.5.2.255;
}
```


### Annexe 3 :
```
# Servers to synchronize with
# Replace XXX with needed server names
server XXX
server XXX
server XXX
server XXX

# Record the rate at which the system clock gains/losses time.
driftfile /var/lib/chrony/drift

# Allow the system clock to be stepped in the first three updates
# if its offset is larger than 1 second.
makestep 1.0 3

# Enable kernel synchronization of the real-time clock (RTC).
rtcsync

# Allow NTP client access from local network.
allow all

# Serve time even if not synchronized to a time source.
local stratum 10

# Specify file containing keys for NTP authentication.
keyfile /etc/chrony.keys

# Specify directory for log files.
logdir /var/log/chrony

# Select which information is logged.
log measurements statistics tracking
```

### Annexe 4 :
```
# Server to syncrhonize with
server server1.tp6.b1 prefer

initstepslew 20 server1.tp6.b1

# Allow NTP client access from local network.
# allow all

# Record the rate at which the system clock gains/losses time.
driftfile /var/lib/chrony/drift

# Allow the system clock to be stepped in the first three updates
# if its offset is larger than 1 second.
makestep 1.0 3

# Enable kernel synchronization of the real-time clock (RTC).
rtcsync

# Serve time even if not synchronized to a time source.
local stratum 10

# Specify file containing keys for NTP authentication.
keyfile /etc/chrony.keys

# Specify directory for log files.
logdir /var/log/chrony
# Select which information is logged.
log measurements statistics tracking
```