---
markmap:
  colorFreezeLevel: 2
  maxWidth: 800
---

# Commandes Linux pour voir les AS traversées et plus encore
- mtr
```bash
  mtr --tcp --port 443 --show-ips --mpls --aslookup --curses www.umontpellier.fr
  mtr -P 443 -T  -b -e -z -t 194.199.227.220
  mtr  -P 443 -T -c 2  -b -e -z -t -j 194.199.227.220 | from json|flatten|get hubs|flatten
```
- traceroute
```bash
traceroute -A -n -e -i eth0 8.8.8.8
```

# Commandes Cisco



## Configuration BGP basique
```ios
# 65100 est l'AS dans lequel est notre routeur
router bgp  65100
    # par défaut on est prévenu des changements sur le voisin
    bgp log-neighbor-changes
    # Commande "synchronisation": un routeur BGP de bordure annonce une route à son voisin BGP que si cette route est aussi connue de son IGP.
    # Le but est d'éviter de perdre du trafic dans une AS de transit dans laquelle un routeur qui ne serait pas iBGP deviendrait un trou noir pour le trafic en transit
    # Mais tous les routeurs de nos jours font du iBGP et dialogue entre eux via un "Route Reflector"  ou on utilise des routeurs MPLS qui peuvent transmettre du trafic sans table de routage.   
    # En conséquence la synchronisation est désactivée par défaut sur un Cisco:
    no synchronisation
    # Sans lookpback lors de la sélection du meilleur chemin un critère est le plus haut router-id.
    # Si la loopback est positionné c'est elle qu sert d'id. A défaut c'est l'IP la plus élevée qui sert d'id. le router-id apporte peu de choses. 
    bgp router-id 1.1.1.1 
    
    # déclaration du voisin BGP et de son AS
    neighbor 10.12.1.1 remote-as 65200
    # Network permet de déclarer les réseaux à annoncer. Contrairement à OSPF cette déclaration n'est pas liée à une interface, c'est juste une déclaration...
    network 192.168.1.0 mask 255.255.255.0
    network 192.168.2.0 mask 255.255.255.0      
    # soft-reconfiguration permet d'accéder aux routes reçues d'un voisin BGP
    # avec la commande "sh ip bgp neighbors 1.1.1.1 received-routes"
    neighbor 10.12.1.1 soft-reconfiguration inbound
```
## Configuration BGP avec une LoopBack

Comme avec le protocoel OSPF annoncer le protocole à un voisin depuis une IP de loopback nous protège d'une panne matérielle et évite de casser la session BGP entre les deux routeurs. 

```ios
# Notre loopback
interface Loopback2
 ip address 2.2.2.2 255.255.255.255
!
router bgp 65535
   bgp log-neighbor-changes

   # Les routes apprises par iBGP ne sont pas retransmises pas défaut dans l'IGP 
   # La commande suivante permet de le faire.  
   bgp redistribute-internal

   network 11.0.0.0 mask 255.255.255.0
   #
   neighbor 1.1.1.1 remote-as 65531

   # Le TTL des paquets eBGP lors de l'établissement d'une session est de 1.
   # Contrairement à ce qu'on peut penser le TTL n'a pas besoin d'être de 2 car il n'est décrémenté qu'en sortie d'un routeur et pas d'une **interface** ! Néanmoins la connexion BGP
   # ne s'ouvre pas car un routeur Cisco ne peut pas se connecter à une interface dans un autre réseau. la commande suivante lui permet donc de by-passer cette restriction en
   # permettant de se connecter à un réseau situé après le réseau directement connecté du voisin BGP. Cette commande laisse le TTL à 1.

   neighbor 1.1.1.1 disable-connected-check 

   # On peut aussi utiliser la commande suivante qui elle va modifier le TTL à "2" et permettre la connexion bgp. Elle fonctionne avec eBGP et est nécessaire avec iBGP 
   # car dans ce cas on a plusieurs sauts et le TTL est décrémenté.

   neighbor 1.1.1.1 ebgp-multihop 2

 

   # On indique avec update-source au routeur d'utiliser toutes les interfaces
   # opérationnelles pour établir les connexions  BGP TCP avec le voisin. 
   # tant que Loopback1 est "up" et est configurée avec une adresse IP
   
   neighbor 1.1.1.1 update-source Loopback1
   # L'AS number est le même ici que celui de notre routeur. Le "voisin" étant dans notre AS on a donc une session iBGP avec lui.
   neighbor 5.5.5.5 remote-as 65535
   neighbor 5.5.5.5 update-source Loopback1

   # Quand une route est transmise à un routeur iBGP
   # le next-hop est un attribut BGP obligatoire
   # En faisant un "lookup" dessus on obtient l'
   # interface de sortie du routeur.
   # Quand une route BGP est transmise d'un routeur eBGP à un
   # routeur iBGP le next_hop reste inchangé et donc le routeur iBGP ne sait pas comment atteindre l'IP de ce NextHop.
   # Le next-hop-self permet que l'IP du routeur source iBGP devienne le next-hop à atteindre 
   neighbor 5.5.5.5 next-hop-self

# Cette route statique obligatoire afin d'atteindre la loopback qui
# est derrière l'interface physique du routeur (Voir la solution IGP ci-dessous)
ip route 1.1.1.1 255.255.255.255 11.0.0.100

```

## Configuration BGP avancée 

Cisco considère que les voisins BGP sont IPv4... ce qui n'est pas forcément vrai.
Il vaut donc mieux l'éviter en activant spécifiquement les protocoles utilisés par chaque voisin.
```ios
interface Loopback3
 ip address 3.3.3.3 255.255.255.255
 ip ospf 1 area 0
!
interface FastEthernet0/0
 ip address 10.0.23.1 255.255.255.252
 duplex auto
 speed auto
!
interface FastEthernet0/1
 ip address 35.0.0.2 255.255.255.252
 duplex auto
 speed auto
!
# C'est OSPF qui permet au voisins BGP de dialoguer de loopback à loopback 
router ospf 1
 log-adjacency-changes
 passive-interface FastEthernet0/1
 network 3.3.3.3 0.0.0.0 area 0
 network 10.0.23.0 0.0.0.255 area 0
!
router bgp 300
# Un voisin IPV4 n'est plus actif par défaut avec cette commande:
 no bgp default ipv4-unicast
 bgp log-neighbor-changes
 neighbor 1.1.1.1 remote-as 300
 neighbor 1.1.1.1 update-source Loopback3
 neighbor 35.0.0.1 remote-as 200
 !
 address-family ipv4
  # activations explicites des "voisins"
  neighbor 1.1.1.1 activate
  neighbor 35.0.0.1 activate
  neighbor 1.1.1.1 soft-reconfiguration inbound
  neighbor 35.0.0.1 soft-reconfiguration inbound
  no auto-summary
  no synchronization
  network 35.0.0.0 mask 255.255.255.252
 exit-address-family
```

## Agrégation de routes

L'agrégation des routes est un enjeu important pour la rapidité de l'internet.

- as-set conserve les attributs initiaux des routes vers les réseaux
  
```ios
aggregate address 192.168.0.0 255.255.0.0 summary-only 
aggregate address 10.202.0.0 255.255.0.0 as-set summary-only 
```
  
## injection de la route par défaut

On ne peut annoncer une route à un voisin BGP seulement si cette route est présente dans la table de routage (RIB). L'astuce est d'annoncer une route statique qui ne sera jamais sélectionnée afin que la route soit présente dans la table de routage et puisse être diffuser par BGP comme ici sur une route par défaut.

```ios
router bgp 65535
   network 0.0.0.0

ip default route 0.0.0.0 0.0.0.0 null0
```


```ios
router bgp 65535
   # oblige la création artificielle de la route et l'injecte dans la RIB BGP, que la 
   # route soit présente ou pas dans la table de routage de l'IGP
   default-information originate
```

```ios
router bgp 65535
   # idem que précédemment mais on averti qu'un seul routeur BGP 
   neighbor 5.5.5.5 default-information originate
```



```ios
# pour voir l'injection c'est la table bgp
R1(config)#do sh ip bgp
BGP table version is 23, local router ID is 1.1.1.1
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal, 
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter, 
              x best-external, a additional-path, c RIB-compressed, 
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 *>  0.0.0.0          192.168.1.1              0         32768 i
 *>  2.2.2.2/32       10.11.0.2                0         32768 ?
```

## Commandes de debug:
  
### running
```ios

sh run | section bgp  
sh run | s bgp
```
### show ip bgp
```ios
show ip bgp neighbor
show ip bgp neighbors | include BGP

# nécessite "neighbor 5.5.5.5 soft-reconfiguration inbound"
# liste les réseaux reçus du voisin
sh ip bgp neighbors 1.1.1.1 received-routes

# liste les réseaux réellement appris du voisin
sh ip bgp neighbors 1.1.1.1 routes

# liste les réseaux annoncés au voisin
sh ip bgp neighbors 1.1.1.1 advertised-routes

# Préfixe le plus long
show ip bgp longer-prefixes

# hard reset ne pas faire en prod 
clear ip bgp *

# soft reset avec l'adresse du voisin
clear ip bgp 192.168.100.1 soft in|out

debug ip tcp transactions
debug ip bgp
debug ip packet
u all # pour arrêter
```

```ios
show ip bgp ipv4 unicast summary 
...
BGP router identifier 2.2.2.2, local AS number 65535
BGP table version is 7, main routing table version 7
6 network entries using 864 bytes of memory
7 path entries using 560 bytes of memory
3/3 BGP path/bestpath attribute entries using 408 bytes of memory
2 BGP AS-PATH entries using 48 bytes of memory
0 BGP route-map cache entries using 0 bytes of memory
0 BGP filter-list cache entries using 0 bytes of memory
BGP using 1880 total bytes of memory
BGP activity 6/0 prefixes, 7/0 paths, scan interval 60 secs

Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
1.1.1.1         4        65531     108     118        7    0    0 01:43:01        3
5.5.5.5         4        65535     122     120        7    0    0 01:42:46        3
```

## Cinématique des updates BGP et avec les routes locales

 - Quand un message BGP UPDATE arrive, l'information est mise dans la table BGP.
 - L'algorithme BGP cherche le meilleur chemin.
 - La RIB du routeur est mise à jour et les voisins aussi.
 - la version de la table est incrémentée pour chaque changement de **meilleur chemin**
  

# topologie
- [topo transit](topo-transit.png)

# nexthop
- [avant-après_nexthop_self](nexthop-self.png)
 
# Utilitaires

- "Looking Glass" :<https://www.cogentco.com/en/looking-glass>
- "Routing Information Service" :<https://ris-live.ripe.net/>
- "Julia Evans Tools to explore BGP": <https://jvns.ca/blog/2021/10/05/tools-to-look-at-bgp-routes/>
# Biblio 
  - Exemples config eBGP/iBGP Cisco: 
    <https://www.cisco.com/c/en/us/support/docs/ip/border-gateway-protocol-bgp/13751-23.html>
  - BGP fundamental Cisco: 
    <https://www.ciscopress.com/articles/article.asp?p=2756480&seqNum=7>
  - Configure iBGP:
    <https://www.mustbegeek.com/configure-ibgp-in-cisco-ios-router>
  - Understand the BGP table
    <https://networkingwithfish.com/understanding-the-bgp-table-version-part-1-introduction-to-bgp-table-version/>
    <https://networkingwithfish.com/understanding-the-bgp-table-version-part-2-bgp-table-version-in-action/>
    <https://networkingwithfish.com/understanding-the-bgp-table-version-part-3-bgp-table-version-troubleshooting/>
  - null interface
    <https://www.cisco.com/c/fr_ca/support/docs/ip/ip-routed-protocols/14956-route-to-null-interface.html>