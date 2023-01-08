---
markmap:
  colorFreezeLevel: 2
  maxWidth: 700
---

# BPG
## Définitions 

- Un "Autonomous System" est un ensemble de réseaux géré par une seule entité.
- ISP: Entreprises dont le métier est de connecter d'autres entreprises à l'internet.
  
    - Les ISP de tiers 1 structurent  l'internet
    - Les tiers 2 vendent et achètent du transit.
    - Les tiers 3 ne fournissent pas de transit. Ils peuvent être "multi-homés"(deux chemins ou plus vers l'internet) afin d'assurer la haute disponibilité des accès et avoir les chemins les plus rapides vers une ressource prisée.

- Les points d'échanges sont des lieux physiques ou s'interconnectent les ISP afin de faciliter les échanges de données (France XI par exemple).
- Un "Point Of Presence" Pop est un lieu physique permettant l'échange de données pour l'internet avec des entités locales.

## Relations entre AS

- Le Transit est le service vendu par un ISP pour donner accès à Internet.
- Le peering est l’échange réciproque de flux souvent dans un but économique.
  - Un peering n'est pas transitif.
  - Il ne donne pas accès à tous les réseaux de l'AS auquel on est connecté.
- voir <https://www.peeringdb.com/net/433> comme exemple.
- Un peer est un routeur d'un autre AS avec lequel on établit une relation de voisinage BGP.


## Caractéristiques de BGP

- La métrique de BGP est de type "distance-vector" et correspond au nombre d'AS traversés. L'attribut de chemins "AS_PATH" qui contient les AS traversés permet (mais  il n'est pas le seul) à BGP de choisir le meilleur chemin.
- Pas de paquets pour l'auto-découverte donc nécessité de déclarer l'IP du voisin pour communiquer.
- Le temps de convergence de BGP est plus long que celui d'un IGP.
- Utilisation du port TCP 179 pour les échanges entre routeur BGP. 
- Bit 'don't fragment' activé dont a besoin le mécanisme de "Path MTU" pour émettre les paquets.
- BGP a besoin d'un IGP. Une route par défaut ne suffit pas.
- Deux règles pour BGP :
   - Principes de synchronisation : Les routes apprises via IBGP doivent être également
apprises par un IGP (OSPF,…) avant d’être annoncées à d’autres peers d’AS différents.
   - La redistribution des routes de l'IGP dans BGP est possible mais distribuer des routes BGP dans l'IGP est beaucoup plus risquée.  
- La table BGP est appelée "Loc-RIB table" et est distincte de la RIB.
- Les routes apprises dans iBGP sont redistribuées dans eBGP et vice-versa. Mais il n'y a pas de redistribution des routes d'un routeur iBGP vers un routeur iBGP.

## Sessions

- eBGP
  - C'est une session BGP entre deux routeurs appartenant à des AS différents.
  - le TTL du paquet BGP est de 1 (le routeur averti doit être en frontal du nôtre et donc à une distance de 1 suffit)
  - Le routeur averti vérifie que son AS n'est pas dans l'AS_PATH reçu. 
  - Le routeur averti positionne le NEXT_HOP pour cet AS à l'IP source du routeur source établissant la connexion.
  - Le routeur averti ajoute son ASN dans l'AS_PATH.
  - Distance administrative 20 pour BGP.
  
- iBGP 
  - Il est fait pour communiquer entre routeurs BGP dans un même AS (TTL à 255).
  - L'attribut AS_PATH n'est pas modifié par iBGP, il n'y a donc pas de détection possible des boucles.
  - Split-Horizon: une route apprise par une session iBGP n'est jamais transmise à un autre voisin iBGP. On est donc
    obligé de faire du "full-mesh" entre tous les routeurs iBGP.
    dans l'IGP pour les transmettre au routeur eBGP de bordure
  - Dans le cas ou l'AS sert de transit les routeurs traversés qui n'implémentent qu'un IGP ne connaissent 
    pas les routes issues de BGP. Il faut donc soit des routes statiques, soit du tunelling MPLS, soit du fullmesh iBGP (route reflector), soit redistribuer les routes BGP dans l'IGP mais...
  - Il n'est pas conseiller de redistribuer les routes BGP dans l'IGP (Il peut y avoir beaucoup de routes BGP...
    trop pour un IGP qui n'est pas taillé pour cela. Il n'y a pas non plus d'équivalence des métriques IGP ni 
    de traductions possibles des attributs BGP dans le "langage" de l'IGP.
  - Distance administrative 200.
  
## MP-BGP (multiprotocol BGP)

- Protocole BGP permet de travailler avec plusieurs familles et sous familles IP.
   - AFI (Address Family Identifier): IPv4 IPv6
   - SAFI (Subsequent Address Family Identifier): Unicast ou Multicast
   - Utilisation d'attributs (Path Attributes) NLRI afin de transporter ces informations.
  
## AS Number (ASN)

- privés: 64512-65535
- publics: le reste ... les ASN sont assignés par IANA. 

## PATH VECTOR protocol avec un mécanisme de prévention des boucles (un AS_PATH contenant l'AS auquel appartient le routeur est mis de côté)

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

- RFC 1105 juin 1989
- RFC 1654 juillet 1994
- RFC 2283 février 1998

