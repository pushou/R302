---
markmap:
  colorFreezeLevel: 2
  maxWidth: 700
---

# Commandes Cisco

- [topo transit](topo-transit.png)
- [avant-après_nexthop_self](nexthop-self.png)
 


```ios
# configuration classique
interface loopbak
router bgp  665100
    bgp log-neighbor-changes
    # "synchronisation" permet de ne pas annoncer une 
    # route BGP tant que l'IGP ne la connais pas
    no synchronisation
    # permet d'indiquer qui pilote le dialogue (état Connect)
    bgp router-id 1.1.1.1 
    # déclaration des réseaux à annoncer
    network 192.168.1.0 mask 255.255.255.0
    network 192.168.2.0 mask 255.255.255.0
    # déclaration du voisin BGP et de son AS
    neighbor 10.12.1.1 remote-as 65200
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
   # afin d'atteindre la loopback 
   neighbor 1.1.1.1 ebgp-multihop 2

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

- Commandes de debug
```ios
sh run | section bgp  
sh run | s bgp
show bgp ipv4 summary
show ip bgp
show ip bgp neighbor
show ip bgp neighbors | include BGP
# hard reset ne pas faire en prod 
clear ip bgp *
# soft reset avec l'adresse du voisin
clear ip bgp 192.168.100.1 soft in|out
debug ip tcp transactions
debug ip bgp
```

- biblio 
    - Exemples config eBGP/iBGP Cisco: 
      <https://www.cisco.com/c/en/us/support/docs/ip/border-gateway-protocol-bgp/13751-23.html>
    - BGP fundamental Cisco: 
      <https://www.ciscopress.com/articles/article.asp?p=2756480&seqNum=7>
    - Configure iBGP:
      <https://www.mustbegeek.com/configure-ibgp-in-cisco-ios-router>