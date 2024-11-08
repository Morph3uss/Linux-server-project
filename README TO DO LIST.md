Projet : Configuration d’un Serveur Linux pour un Réseau Local

Ce projet a pour but de configurer un serveur Linux pour un réseau local, avec une machine serveur et une machine client. La configuration inclut les services DHCP, DNS, un serveur web, et l'accès SSH, en utilisant des VMs sous Debian ou Ubuntu. Les différences de configuration entre Debian et Ubuntu sont indiquées pour faciliter l’adaptation.
Table des Matières

Configuration des Machines Virtuelles
    Partitionnement du Disque
    Configuration du Serveur Linux
        Installation et Configuration DHCP
        Installation et Configuration DNS (BIND9)
        Installation et Configuration du Serveur Web (Nginx)
        Configuration de SSH avec Authentification par Mot de Passe et par Clé
    Routage et NAT
    Comparaisons Debian/Ubuntu

Configuration des Machines Virtuelles
Réseau des Machines Virtuelles

   Serveur VM :
        Carte réseau 1 : Réseau interne (nommé intnet) pour la communication entre les VMs.
        Carte réseau 2 : NAT pour l'accès à Internet.
    Client VM :
        Carte réseau : Réseau interne (intnet) pour qu’elle puisse se connecter au serveur et obtenir un accès Internet via le serveur.

Remarque : Assurez-vous que les deux VMs utilisent bien le réseau interne intnet pour que la communication entre elles soit possible.
Paramètres de Base

   VM Serveur : Utilisez Debian (exemple : Debian 12) ou Ubuntu Server.
   VM Client : Utilisez Debian ou Ubuntu Desktop.

Partitionnement du Disque
Exemple de Partitionnement pour la VM Client

Les valeurs d’espace disque sont données à titre d’exemple, ajustez-les selon votre environnement :
Point de montage	Taille allouée
/	10 GB
/boot	512 MB
/home	12 GB
/var	2 GB
swap	2 GB
Total	26.84 GB

Pour configurer le partitionnement manuellement, utilisez des outils comme fdisk ou parted en ligne de commande, ou suivez l'assistant de partitionnement lors de l'installation du système.
Configuration du Serveur Linux
Installation et Configuration DHCP

Le serveur DHCP distribuera des adresses IP automatiquement aux machines du réseau interne.
    Installer le serveur DHCP :
    sudo apt install isc-dhcp-server

   Configuration du DHCP :
       Debian : Modifiez /etc/dhcp/dhcpd.conf.
        Ubuntu : Même fichier, mais vérifiez que le service est configuré pour écouter sur l’interface réseau interne.

   Exemple de configuration dans /etc/dhcp/dhcpd.conf :

   subnet 192.168.1.0 netmask 255.255.255.0 {
    range 192.168.1.100 192.168.1.150;
    option routers 192.168.1.1; # Adresse IP de la VM serveur sur le réseau interne
    option domain-name-servers 8.8.8.8, 8.8.4.4; # DNS de Google
    }

  Redémarrer le service DHCP :
    sudo systemctl restart isc-dhcp-server

Installation et Configuration DNS (BIND9)

Le serveur DNS (BIND9) gère la résolution des noms pour les ressources locales.

  Installer BIND9 :
    sudo apt install bind9 bind9utils

   Configurer le DNS :
        Dans /etc/bind/named.conf.local, ajoutez une zone pour le réseau local (ex. library.local).

   Exemple de configuration :

  zone "library.local" {
    type master;
    file "/etc/bind/db.library.local";
    };

  Créer un fichier de zone (/etc/bind/db.library.local) :
  
  $TTL 604800
    @ IN SOA library.local. admin.library.local. (
    2 ; Serial
    604800 ; Refresh
    86400 ; Retry
    2419200 ; Expire
    604800 ) ; Negative Cache TTL

  @ IN NS ns.library.local.
    ns IN A 192.168.1.10

  Redémarrer le Service DNS :
    sudo systemctl restart bind9

Installation et Configuration du Serveur Web (Nginx)

  Installer Nginx :
    sudo apt install nginx

  Configurer la Page d’Accueil :
        Créez un fichier HTML simple dans /var/www/html/index.html :
    <!DOCTYPE html> <html> <head> <title>Bibliothèque Locale</title> </head> <body> <h1>Bienvenue à la bibliothèque!</h1> <p>Page d'accueil du serveur de la bibliothèque.</p> </body> </html>

  Vérifiez l’accès au serveur web en visitant http://<adresse-IP-du-serveur>.

Configuration de SSH avec Authentification par Mot de Passe et par Clé
Méthode 1 : Authentification par Mot de Passe

   Installer SSH :
    sudo apt install openssh-server

Démarrer et vérifier SSH :
    sudo systemctl start ssh
    sudo systemctl enable ssh

Méthode 2 : Authentification par Clé (recommandée pour plus de sécurité)
   
  Générer une paire de clés sur la VM cliente :
    ssh-keygen

   Copier la clé publique vers le serveur :
    ssh-copy-id utilisateur@<IP_du_serveur>

  Désactiver l’authentification par mot de passe pour renforcer la sécurité :

    
  Dans /etc/ssh/sshd_config, définissez :

  PasswordAuthentication no

  Redémarrer SSH :
    sudo systemctl restart ssh

Routage et NAT

Pour que la VM serveur fournisse un accès Internet à la VM cliente, configurez le routage et le NAT.

   Activer le routage IP sur le serveur :
    echo "net.ipv4.ip_forward=1" | sudo tee -a /etc/sysctl.conf
    sudo sysctl -p

  Configurer le NAT avec iptables :
    sudo iptables -t nat -A POSTROUTING -o eth1 -j MASQUERADE

   Sauvegarder les règles iptables :
    sudo apt install iptables-persistent
    sudo netfilter-persistent save

Comparaisons Debian/Ubuntu
Configuration	Debian	Ubuntu
Réseau (interfaces)	/etc/network/interfaces	/etc/netplan/50-cloud-init.yaml
Service DHCP	/etc/dhcp/dhcpd.conf	Même fichier
Commande de redémarrage	systemctl restart <service>	Même commande
