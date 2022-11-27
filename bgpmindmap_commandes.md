---
markmap:
  colorFreezeLevel: 2
  maxWidth: 800
---

# Commandes Cisco

- [topo transit](topo-transit.png)
- [avant-après_nexthop_self](nexthop-self.png)
 


```ios
# configuration classique
interface loopbak
router bgp  665100
    bgp log-neighbor-changes
    
    # "synchronisation" permet de n'annoncer une route via BGP que si la route est connue de l'IGP
    # Permet d'échapper au principe de synchronisation
    no synchronisation
    # permet d'indiquer qui pilote le dialogue (état Connect)
    bgp router-id 1.1.1.1 
    # déclaration des réseaux à annoncer
    network 192.168.1.0 mask 255.255.255.0
    network 192.168.2.0 mask 255.255.255.0
    # déclaration du voisin BGP et de son AS
    neighbor 10.12.1.1 remote-as 65200
    # permet d'accéder aux résultats de 
    # sh ip bgp neighbors 1.1.1.1 received-routes
    neighbor 10.12.1.1 soft-reconfiguration inbound
```

```ios
# configuration à base de loopbacks pour eBGP et iBGP
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

   # le TTL du "packet" passe à 2 
   # afin d'atteindre la loopback on change le TTL du paquet BGP à deux
   neighbor 1.1.1.1 ebgp-multihop 2
   # Une autre façon de faire désactiver le besoin de BGP d'avoir une 
   # connexion directe avec le routeur voisin 
   neighbor 1.1.1.1 disable-connected-check

   # indique au routeur d'utiliser toute interface
   # opérationnelle pour établir les connexions TCP 
   # tant que Lo2 est active et
   # configurée avec une adresse IP
   # = plus de stabilité et de résilience
   neighbor 1.1.1.1 update-source Loopback2
   # L'AS number est le même ici ce qui permet
   # de qualifier cette session comme une session iBGP
   neighbor 5.5.5.5 remote-as 65535
   neighbor 5.5.5.5 update-source Loopback2

   # Quand une route est transmise à un routeur iBGP
   # le next-hop est un attribut BGP obligatoire
   # En faisant un "lookup" dessus on obtient l'
   # interface de sortie du routeur.
   # Quand une route BGP est transmise d'un routeur eBGP à un
   # routeur iBGP le next_hop reste inchangé.
   # Le next-hop-self devient l'IP du routeur source iBGP 
   neighbor 5.5.5.5 next-hop-self

# route statique obligatoire afin d'atteindre la loopback qui
# est derrière l'interface physique du routeur
ip route 1.1.1.1 255.255.255.255 11.0.0.100
```

```ios
    # agrégation de routes
    aggregate address 192.168.0.0 255.255.0.0 summary-only 
    # as-set conserve les attributs initiaux 
    aggregate address 10.202.0.0 255.255.0.0 as-set summary-only 
```

- Injection de la route par défaut.
  
```ios
router bgp 65535
   # Injecte la route par défaut dans BGP seulement si la route par défaut est présente dans la table de routage, 
   # ou apprise d'un autre protocole
   network 0.0.0.0
# command enables the router to advertise the default route because the router thinks that 0.0.0.0
# is directly connected via Null0 - si c'est une stub AS il faut le faire pointer vers l'IP du voisin BGP de l'ISP.
ip default route 0.0.0.0 0.0.0.0 null0
```

```ios
router bgp 65535
   # oblige la création artificielle de la route et l'injecte dans la RIB BGP, que la route soit présente ou pas
   # dans la table de routage
   default-information originate
```
```ios
router bgp 65535
   # idem que précédemment mais on averti qu'un seul routeur BGP 
   neighbor 5.5.5.5 default-information originate
```

```ios
# pour voir l'injection c'est la table bgp rien dans la RIB
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

- Commandes de debug:
  
```ios

sh run | section bgp  
sh run | s bgp

show ip bgp
show ip bgp neighbor
show ip bgp neighbors | include BGP

# nécessite "neighbor 5.5.5.5 soft-reconfiguration inbound"
# liste les réseaux reçus du voisin
sh ip bgp neighbors 1.1.1.1 received-routes

# liste les réseaux réellement appris du voisin
sh ip bgp neighbors 1.1.1.1 routes

# liste les réseaux annoncés au voisin
sh ip bgp neighbors 1.1.1.1 advertised-routes

# hard reset ne pas faire en prod 
clear ip bgp *

# soft reset avec l'adresse du voisin
clear ip bgp 192.168.100.1 soft in|out

debug ip tcp transactions
debug ip bgp
debug ip packet
u all # pour arrêter
```

- Tables BGP
   lors des updates BGP et avec les routes locales
  - Quand un message BP UPDATE arrive, l'information est mise dans la table BGP.
  - L'algorithme BGP cherche le meilleur chemin
  - La RIB du routeur est mise à jour et les voisins aussi.
  - la version de la table est incrémentée pour chaque changement de **meilleur chemin**
  
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


- Churn

- biblio 
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