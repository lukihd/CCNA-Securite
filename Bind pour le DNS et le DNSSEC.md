# Bind pour le DNS et le DNSSEC



[TOC]

## Pourquoi utiliser Bind

Nous avons choisis d’utiliser Bind 9 pour notre infrastructure car il est le plus utilisé pour la gestion de service DNS (Domain Name System) et DNSSEC (Domain Name System Security Extensions). De plus on retrouve beaucoup de ressource sur ce sujet. Le debug est donc simplifier et en cas de problème la réactivité n’en sera qu’améliorée.

## Bind DNS

###Installation

D’abord on installe Bind

````bash
yum install bind bind-utils -y
````

Il y a plusieurs prérequis pour utiliser Bind :

- trusted-recursion : Les adresses IP ou sous-réseaux que vous souhaitez autoriser en entrée pour effectuer des recherches.
- forwarders : Spécifier 2 serveurs DNS (exemple ceux de google)
- zonefile : Faire un fichier pour les zones d’action du service



Nous avons utiliser cette configuration :

```bash
- trusted-recursion depuis le réseau 10.1.0.0/24
- forwarders : 8.8.8.8 & 8.8.4.4
- zonefile :  on va utiliser le nom starcrypt.com
```



Une fois cette étape terminée on passe à la configuration.

###Configuration

On doit donc se diriger dans le fichier de configuration `/etc/named.conf` 

```
cp /etc/named.conf /etc/named.conf.orig
```

Dans notre fichier de config on va ajouter les mentions suivantes

```bash
acl "trusted-recursion" {
        localhost;
        localnets;
        #on ajoute notre réseau
        10.1.0.0/24;
};
[...]
option {
    [...]
    forwarders {
    	# on ajoute les serveurs DNS de google comme forwarders
        8.8.8.8;
        8.8.4.4;
    }
}
[...]
zone "starcrypt.com" {
    type master;
    file "dynamic/starcrypt.com"; # chemin vers notre config
    allow-transfer { 10.1.0.254; };
    notify yes;
};

zone "10.1.0.in-addr.arpa" in {
        type master;
        file "dynamic/10.0.1.in-addr.arpa.zone";
        allow-transfer { 10.1.0.254; };
        notify yes;
};

```

Une fois le fichier configuré nous avons créer le fichier de la zone “starcrypt.com”

``````bash
nano /var/named/dynamic/db.starcrypt.com
$TTL 300                ; 5 minutes 
@       IN      SOA     r1.starcrypt.com. admin.starcrypt.com. (
                  1     ; Serial
               3600     ; Refresh
                300     ; Retry
            1814400     ; Expire
                300 )   ; Negative Cache TTL

; name servers - NS records
    IN      NS      r1.starcrypt.com.

; name servers - A records
r1.starcrypt.com.     IN     A     10.1.0.254

; All other A records
web.starcrypt.com.    IN     A     10.1.0.1
``````

Ensuite on met en place le reverse dns

```
nano /var/named/dynamic/10.1.0.in-addr.arpa.zone
$ORIGIN 10.1.0.in-addr.arpa.
$TTL 86400              ; 1 day
@       IN      SOA     r1.starcrypt.com. admin.starcrypt.com. (
                  1     ; Serial
               7200     ; refresh (2 hous)
               7200     ; retry (2 hours)
            2419200     ; expire (5 weeks 6 days 16 hours)
              86400  )  ; minimum (1 day)

10.1.0.in-addr.arpa. IN NS r1.starcrypt.com.

254     IN     PTR     r1.starcrypt.com.

1       IN     PTR     www.starcrypt.com.
```

on vérifie la config avec la commande `named-checkconf` et  ensuite on dit à bind de se lancer au démarrage.

``` bash
systemctl enable named
systemctl restart named
```

Notre DNS est configuré on peut passer à l’étape suivante, le DNSSEC.

## Bind DNSSEC

### Configuration

Pour le DNSSEC pas besoin d’installation puisqu’il est déjà compris dans Bind il suffit d’ajouter les options

```  
dnssec-enable yes;
dnssec-validation yes;
dnssec-lookaside auto;
```

Dans la partie `options {}` du fichier de config `named.conf`

On crée en suite une ZSK (Zone Signing Key) avec la commande 

```
dnssec-keygen -a NSEC3RSASHA1 -b 2048 -n ZONE sytarcrypt.com
```

Puis une KSK ( Key Signing Key ) avec la commande 

```
dnssec-keygen -f KSK -a NSEC3RSASHA1 -b 4096 -n ZONE starcrypt.com
```

On ajoute ensuite ces clés dans le fichier de zone puis on utilise la commande dnssec-signzone. D’après le tuto que j’ai suivis apres le -3 on doit mettre ca mais je sais pas trop pourquoi.

```bash
dnssec-signzone -3 $(head -c 1000 /dev/random | sha1sum | cut -b 1-16) -A -N INCREMENT -o starcrypt.com -t starcrypt.com.zone
# et ca nous sort ça si il n'y a pas d'erreur
Verifying the zone using the following algorithms: NSEC3RSASHA1.
Zone signing complete:
Algorithm: NSEC3RSASHA1: KSKs: 1 active, 0 stand-by, 0 revoked
                        ZSKs: 1 active, 0 stand-by, 0 revoked
example.com.zone.signed
Signatures generated:                       14
Signatures retained:                         0
Signatures dropped:                          0
Signatures successfully verified:            0
Signatures unsuccessfully verified:          0
Signing time in seconds:                 0.046
Signatures per second:                 298.310
Runtime in seconds:                      0.056
```

Cette commande va créer un fichier `starcrypt.com.zone.signed` on va donc modifier le fichier de zone précédemment mis dans la conf du DNS par celui-ci.

```
nano /etc/named.conf
[...]
zone "starcrypt.com" {
    type master;
    file "dynamic/starcrypt.com.zone.signed"; # chemin vers notre config dnssec
    allow-transfer { 10.1.0.254; };
    notify yes;
};
```

On sauvegarde et relance bind. Notre serveur DNSSEC est donc opérationnel.