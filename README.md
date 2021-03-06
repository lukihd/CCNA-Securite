# CCNA-Sécurité

Nous avons choisi ce Tp sur la sécurité car nous sommes intéréssés tous les deux par la sécurité informatique.
Ce Tp nous a permis de consolider nos connaissances mais aussi d'apprendre, de découvrir des outils permettant de sécuriser une infra.

## Sommaire

- [Firewalld](#Firewalld)
- [Serveur Web](#ServeurWeb)
- [Nagios](#Monitoring)
- [Bind](https://github.com/lukihd/CCNA-Securite/blob/master/Bind%20pour%20le%20DNS%20et%20le%20DNSSEC.md)

## Sujet

Renforcer la sécurité d'une topologie simple qui comporte quelques services réseaux.

1. Infra (un petit truc suffira)
    * backbone (= accès WAN/LAN)
    * switches d'accès clients
    * clients

2. Sécurité des flux
    * firewall frontal
        * pfsense (ou opensense)
        * ou hardened CentOS
    * DMZ
    * VLANs
    * Authentification forte sur les équipements
        * SSH (routeurs, switches, VM)

3. Services réseau
    * DNS + DNSSEC
    * Serveur Web + WAF
        * apache + modsecurity
        * NGINX + naxsi
        * ou autres

4. Monitoring/Métrologie
    * Inspection de flux réseau
        * détection des flux réseau
        * visualisation
            * genre un dashboard qui affiche clairement l'état du traffic
            * il y a tel trafic qui circule dans tel lien
    * DPI (deep packet inspection)


## Choix de l'infra

![schéma infra](https://github.com/lukihd/CCNA-Securite/blob/master/Annexes/infra.png)

## Firewalld

Pour configurer et sécuriser au mieux nos clients, nous avons décidé d'utiliser firewalld qui est un pare-feu déjà par défaut sur les machines centos.

Dans le firewalld, on va configurer des zones réseaux, elles permettent d'isoler des ordinateurs et de leurs aplliquées des politiques de sécurité différentes. 

Dans notre cas, afin de sécuriser au max le client, on va utiliser la zone DMZ, qui est une zone démilitarisée. Elle joue la rôle de la zone tampon entre le réseau à protéger et un réseau hostile.
La DMZ va être formé sur la troisième carte réseau en sachant que la première est formé par le réseau externe et la seconde par le réseau interne.

A noter qu'il architeture plus sécurisée consiste à utiliser deux firewalls pour créer une DMZ. Le premier laisse passer uniquement le trafic vers la DMZ et le second autorise que le trafic entre la DMZ et el réseau interne. Cette configuration est considérée comme mieux sécurisée car une personne malveillante devra compromettre deux machines pour acéder au Lan interne. Malheuresement, nous n'avons pas rééussis à ettre en place cette architecture.

```centos@client1# sudo firewall-cmd --zone=work --change-interface=enp0s8```

## ServeurWeb 

Pour le serveur web, nous avons choisi de prendre apacha, que nous avons le plus souvent utilisé. Et en matière de sécurité, il est très bine car toutes les failles possibles ont été prises en compte.
De plus, on a utilisé un module d'Apache, ModSecurity. Il permet de sécuriser la couche applicative avant l'arrivée des requêtes sur le site hébergé.

```centos@client1# sudo yum install libapache-mod-security```

Une fois installé mod_security devient un outil de protection "générique" dans le sens où il protège contre des types d'attaques plutôt que des vulnérabilités.
Les règles présentes par défauts sur le système se cachent dans le dossier:

 ```/usr/share/modsecurity-crs```

## Monitoring

Après de nombreuses recherches, nous avons trouvé Nagios qui est un des plus populaire des sytèmes monitoring open source.
Nagios est uitle pour garder un inventaire des serveurs et pour assurer les services critiques sont opérationnels.
C'est un outil esssentiel pour tout environnement de rserveur de producion.

Tout d'abord il faut installer des librairies pour compléter l'installation par la suite:
```sudo yum install gcc glibc glibc-common gd gd-devel make net-snmp openssl-devel xinetd unzip```

Puis on installe Nagios core
```centos@router# sudo curl -L -O https://assets.nagios.com/downloads/nagioscore/releases/nagios-4.1.1.tar.gz```

```centos@router# sudo tar xvf nagios-*.tar.gz```

Il faut ensuite mettre en marche quelques commandes :
```
sudo make install
sudo make install-commandmode
sudo make install-init
sudo make install-config
sudo make install-webconf
```

Pour finit l'installation, il faut installer les plugins qui finalise l'installation de nagios:
```centos@router# sudo curl -L -O http://nagios-plugins.org/download/nagios-plugins-2.1.1.tar.gz```

Depuis cette étape impossible de dézipper le plugins, et donc impossible de configurer le nagios. 
