---
markmap:
  colorFreezeLevel: 2
  maxWidth: 700


---

# BPG

## Caractéristiques

 - La métrique de BGP est de type "distance-vector" et correspond au nombre d'AS traversées. L'attribut de chemins "AS_PATH" qui contient les AS traversées permet (mais il n'est pas le seul) à BGP de choisir
- pas de hello packets donc nécessité de déclarer l'IP du voisin
- utilisation du port TCP 179 pour les échanges entre routeur BGP. 
- bit 'don't fragment' activé donc besoin du mécanisme de Path MTU.
- A besoin d'un IGP poP = Adjacence . Une route par défaut ne suffit pas.
- La redistribution des routes de l'IGP dans BGP est possible. Le contraire: routes BGP dans l'IGP est beaucoup plus risquée.  


## Sessions

- eBGP
  - TTL à 1  (le routeur averti doit être en frontal du notre)
  - Le routeur averti positionne le NEXT_HOP pour cet AS à l'IP source du routeur établissant la connexion
  - Le routeur averti ajoute son ASN dans l'AS_PATH
  - Le routeur averti vérifie que son AS  
  - Distance administrative 20.
- iBGP 
  - Fait pour le transit (TTL à 255)
  - On ne peut pas redistribuer les routes BGP dans l'IGP ( trop de route - pas d'équivalence des métriques IGP, pas de traduction des attributs dans le "langage" de l'IGP)
  - Nécessité de faire du full mesh ou un "route reflector" si il y a plusieurs routeurs
  - Distance administrative 200
  - Les routes apprises par iBGP ne sont pas retransmises dans l'IGP. 

 ## MP-BGP (multiprotocol BGP)

- AFI (Address Family Identifier): IPv4 IPv6
- SAFI (Subsequent Address Family Identifier): Unicast ou Multicast
- Utilisation d'attributs (Path Attributes) NLRI afin de transporter ces informations.

## AS Number (ASN)

- privés: 64512-65535
- publiques: le reste ils sont assignés par IANA 

## PATH VECTOR protocol avec un mécanisme de prévention des boucles (un AS_PATH contenant l'AS auquel appartient le routeur est mis de côté )

## BGP messages

- OPEN (établissement de l'adjacence)
- UPDATE (échange, retraits de routes)
- NOTIFICATION (message d'erreurs)
- KEEPALIVE (vérification que le peer est vivant)

## BGP neighbor state

- Idle (démarrage de la session connexion au voisin)
- Connect (après 3WayHandshake -> reset -> message OPEN envoyé par le routeur à l'IP la plus haute)
- Active ( l'autre routeur répond par un message Open vers OpenSet)
- OpenSent (sans erreur migration vers OpenConfirm)
- OpenConfirm (BGP attend des messages KEEPALIVE ou NOTIFICATION)
- Established (échange de routes via le message UPDATE)

## Agrégation de routes avec BGP

- Afin de réduire la taille des tables de routages
- Améliorer la stabilité en cachant les "route flaps". 


## RFC

- RFC 1105 Juin 1989
- RFC 1654 Juillet 1994
- RFC 2283 February 1998

## Commandes

```ios
    router bgp  665100
    bgp log-neighbor-changes
    # 
    no synchronisation
    # permet d'indiquer qui pilote le dialogue (état Connect)
    bgp router-id 1.1.1.1 
    aggregate address 192.168.0.0 255.255.0.0 summary-only # agrégation
    # as-set conserve les attributs initiaux 
    aggregate address 10.202.0.0 255.255.0.0 as-set summary-only 
    bgp redistribute internal
    network 192.168.1.0 mask 255.255.255.0
    network 192.168.2.0 mask 255.255.255.0
    neighbor 10.12.1.1 remote-as 65200
```
```ios
# spécificités ibgp

Router bgp 65000
  neighbor 192.168.100.1 remote-as 65000 

# indique au routeur d'utiliser toute interface
# opérationnelle pour établir les connexions TCP 
# tant que Lo0 est active et
# configurée avec une adresse IP)

  neighbor 192.168.100.1 update-source loopback 0

# Lorsqu'un routeur EBGP reçoit des routes d'un voisin
# EBGP, il transmet les routes aux routeurs IBGP avec
# l'adresse source du voisin. Mais le routeur ibg ne sais pas
# joindre ce routeur qui en dehors de l'IGP. 
# Le next-hop-self remplace l'IP du voisin
# par celle du routeur EGP de bordure    

  neighbor 192.168.100.1 next-hop-self

```

```ios
sh run | s bgp
sh run | section bgp  
show bgp ipv4 summary
show ip bgp
show ip bgp neighbor
show ip bgp neighbors | include BGP
# hard reset ne pas faire en prod 
clear ip bgp *
# soft reset
clear ip bgp 192.168.100.1 soft in|out
debug ip tcp transactions
debug ip bgp
```
## synchronisation: usecase transitaire
```ios
Router bgp 65000
   no synchronisation
```
  - Un routeur BGP apprend un réseau d'un autre peer BGP. La route apprise est transmise via iBGP à un autre routeur. 

## Attributs

- LOCAL_PREFERENCE: Attribut positionné sur les routeurs BGP de bordures
- Intérêt minimiser les distances
    -
    - AS_PATH (inter domaines)
    - MED (intra domaines)
    - eBGP avant iBGP
    - Coût IGP