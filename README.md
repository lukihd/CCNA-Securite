# CCNA-Sécurité

Nous avons choisi ce Tp sur la sécurité car nous sommes intéréssé tous les deux par la sécurité informatique.
Ce Tp nous a permis de consolider nos connaissances mais aussi d'apprendre, de découvrir des outils permettant de ssécuriser unne infra.

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

![schéma infra](https://github.com/lukihd/CCNA-Securite/blob/master/Annexes/sch%C3%A9ma.png)

## Mise en place de l'infra