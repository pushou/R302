---
markmap:
  colorFreezeLevel: 2
  maxWidth: 700


---

# BPG

## Caractéristiques

- La métrique de BGP est de type "distance-vector" et correspond au nombre d'AS traversées. L'attribut de chemins "AS_PATH" qui contient les AS traversées permet (mais il n'est pas le seul) à BGP de choisir
- pas de hello paquets donc nécessité de déclarer l'IP du voisin
- utilisation du port TCP 179 pour les échanges entre routeur BGP. 
- bit 'don't fragment' activé donc besoin du mécanisme de Path MTU.
- A besoin d'un IGP poP = Adjacence . Une route par défaut ne suffit pas.
- La redistribution des routes de l'IGP dans BGP est possible. Le contraire: routes BGP dans l'IGP est beaucoup plus risquée.  
- La table BGP est appelée Loc-RIB table donc distincte de la RIB.


## Sessions

- eBGP
  - TTL à 1  (le routeur averti doit être en frontal du notre)
  - Le routeur averti positionne le NEXT_HOP pour cet AS à l'IP source du routeur établissant la connexion
  - Le routeur averti ajoute son ASN dans l'AS_PATH
  - Le routeur averti vérifie que son AS  
  - Distance administrative 20.
- iBGP 
  - Il est fait pour communiquer entre routeurs BGP dans une même AS (TTL à 255). 
  - Dans le cas ou l'AS sert de transit les routeurs traversés qui n'implémentent qu'un IGP ne connaissent 
    pas les routes issues de BGP. Il faut donc soit des routes statiques, soit du tunelling MPLS, soit du fullmesh iBGP (route reflector), soit redistribuer les routes BGP dans l'IGP mais...
  - Il n'est pas conseiller de redistribuer les routes BGP dans l'IGP (Il peut y avoir beaucoup de routes BGP...
    trop pour un IGP qui n'est pas taillé pour cela. Il n'y a pas non plus d'équivalence des métriques IGP ni 
    de traductions possibles des attributs BGP dans le "langage" de l'IGP.
  - Distance administrative 200.
  
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

