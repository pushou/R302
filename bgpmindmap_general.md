---
markmap:
  colorFreezeLevel: 2
  maxWidth: 700
---

# BPG
## Définitions 

- Un "Autonomous System" est un ensemble de réseaux gérés par une seule entité. Par exemple "RENATER" qui fédère les réseaux des universités françaises.
- Un ISP est une entreprise dont le métier est de connecter d'autres entreprises à l'internet.
  
    - Les "ISP" de tiers 1 structurent l'internet.
    - Les "tiers 2" vendent et achètent du transit.
    - Les "tiers 3" ne fournissent pas de transit. Ils peuvent être "multi-homés"(deux chemins ou plus vers l'internet) afin d'assurer la haute disponibilité des accès et avoir les chemins les plus rapides vers une ressource prisée comme un "GAFAM".

- Les points d'échanges sont des lieux physiques où s'interconnectent les "ISP" afin de faciliter les échanges de données (France XI par exemple).
- Un "Point Of Presence" ("POP") est un lieu physique permettant l'échange de données pour l'internet avec des entités locales.

## Relations entre AS

- Le "transit" est le service vendu par un "ISP" (Internet Service Provider) pour donner accès à Internet.
- Le "peering" est l’échange réciproque de flux souvent dans un but économique (voir <https://www.peeringdb.com/net/433> comme exemple).
  - Un "peering" n'est pas transitif : ce n'est pas parce que A à un "peer" avec B et avec C que C a un "peering" avec B.
  - Un "peering" ne donne pas accès à tous les réseaux de l'"AS" auquel on est connecté.

## Caractéristiques de BGP

- La métrique de BGP est de type "distance-vector" et correspond au nombre d'"AS" traversés. L'attribut de chemins "AS_PATH" qui contient les AS traversés permet (mais il n'est pas le seul critère) à BGP de choisir le meilleur chemin: c'est le plus court "AS-PATH" qui est retenu comme chemin.
-  BGP dispose d'un mécanisme de prévention des boucles : si un "AS_PATH" contient l'"AS" auquel appartient le routeur ce vecteur est mise de côté lors du choix de la meilleure route.
- Il n'y a pas de paquets "Hello" pour l'auto-découverte des "voisins" entre eux et donc il est nécessaire de déclarer l'IP du voisin pour communiquer.
- Le temps de convergence de BGP est plus long que celui d'un IGP. En contrepartie, il gère des centaines de milliers de routes ce que ne peut faire un IGP.
- Le port TCP 179 est utilisé pour les échanges entre routeurs BGP. 
- Le bit "don't fragment" est activé et est utilisé par le mécanisme de "Path MTU".
- BGP a besoin d'un IGP pour partager les adresses internes à l'AS. Une route par défaut ne suffit pas.
- La table BGP est appelée "Loc-RIB table" (faire "show ip bgp") et est distincte de la RIB (faire "show ip route"). En conséquence,  il faut vérifier les deux tables : si la route exacte n'existe pas dans la "RIB BGP" ne transmettra pas la route à un autre AS 
- Les routes apprises dans iBGP sont redistribuées dans eBGP et vice-versa. Mais il n'y a pas de redistribution des routes d'un routeur iBGP vers un routeur iBGP.

## Deux règles pour BGP :

- **Principes de synchronisation** : Les routes apprises via iBGP doivent être également dans la table de routage de votre IGP (OSPF,…) avant d’être annoncées à d’autres voisins d’AS différents. Cette règle a pour but de ne pas accepter de transit qu'un routeur hors du réseau iBGP ne pourrait pas router conduisant à devenir un "black hole". Ce mécanisme ralentit la convergence. Les routeurs supportant dans leur immense majorité iBGP, la synchronisation est donc désactivée par défaut chez Cisco.
- **Split-Horizon**: une route apprise par une session iBGP n'est jamais transmise à un autre voisin iBGP afin d'éviter les boucles. On est donc obligé de faire du "full-mesh" entre tous les routeurs iBGP afin que ces routeurs iBGP se connaissent.



## Sessions

- **eBGP**
    - C'est une session BGP entre deux routeurs appartenant à des AS différents.
    - Le TTL du paquet BGP est de 1 (le routeur averti doit être en frontal du nôtre et donc à une distance de 1 suffit et sécurise les échanges)
    - Le routeur averti vérifie que son AS n'est pas dans l'"AS_PATH" reçu. 
    - Le routeur averti positionne le "NEXT-HOP" pour cette AS à l'IP source du routeur source établissant la connexion.
    - Le routeur averti ajoute son ASN dans l'"AS_PATH".
    - La distance administrative est de 20 pour BGP.
  
- **iBGP** 
    - Il est fait pour communiquer entre routeurs BGP dans un même AS (TTL des paquets > 1).
    - L'attribut AS_PATH n'est pas modifié par iBGP, il n'y a donc pas de détection possible des boucles avec iBGP.
    - Dans le cas d'un AS de transit les routeurs traversés qui n'implémentent qu'un IGP ne connaissent 
      pas les routes issues de BGP. Il faut donc soit des routes statiques, soit du "tunelling" MPLS, soit du fullmesh iBGP ("route reflector"), soit redistribuer les routes BGP dans l'IGP mais ...
      - Il n'est pas conseiller de redistribuer les routes BGP dans l'IGP : il peut y avoir beaucoup de routes BGP...trop pour un IGP qui n'est pas taillé pour cela. Il n'y a pas non plus d'équivalence des métriques IGP ni de traductions possibles des attributs BGP dans le "langage" de l'IGP. 
      - La redistribution des routes de l'IGP dans BGP n'est pas souhaitable non plus : il vaut mieux que tous les routeurs internes à l'AS supportent iBGP avec une topologie "full mesh" qui peut être facilitée par un "RR" (réflecteur de route).
    - L'attribut "NEXT_HOP" reçu lors de l'échange BGP indique quel est la prochaine IP pour atteindre un réseau. L'attribut "NEXT_HOP" n'est pas mis à jour lors sa transmission via iBGP à un routeur. Ce dernier se retrouve avec un "NEXT_HOP" qui est celui du voisinBGP du premier routeur et il lui faut donc un chemin pour le joindre :  
       - Soit en injectant ce "NEXT_HOP"  dans l'IGP de l'AS (déclaration Network dans le routeur eBGP de bordure de notre AS).
       - Soit en utilisant la commande "next-hop-self" qui permet de modifier le "NEXT_HOP" avec l'adresse du routeur.
    - La distance administrative de iBGP est de 200. 
   
## MP-BGP (multiprotocol BGP)

- Protocole BGP permet de travailler avec plusieurs familles et sous familles IP.
Les notions d'"Address Family Identifier" (**AFI**) et de "Subsequent Address Family Identifier" (**SAFI**) correspondent au format des préfixes transportés:
    - Les adresses IPv4 sont identifiées en AFI => IPv4 (1) et SAFI = unicast ou multicast (1).
    - Les adresses IPv6 sont identifiées en AFI => IPv4 (2) et SAFI = unicast ou multicast (1).
    - Les adresses VPN IPv4 sont identifiées en AFI = IPv4 (1), SAFI = VPN (128). Une des particularités des routes VPN est d'avoir un label associé.
- On utilise l'attribut "Path Attributes NLRI" afin de transporter ces informations.
      
## AS Number (**ASN**)

- privés: 64512-65535
- publics: le reste ... les ASN sont assignés par le "IANA"

## Messages BGP échangés sur le port tcp 179

- OPEN (établissement de l'adjacence).
- UPDATE (échange, retraits de routes).
- NOTIFICATION (message d'erreurs).
- KEEPALIVE (vérification que le "voisin" est vivant toutes les 60 secondes, au bout de 180 secondes toutes les routes provenant de ce "voisin" seront supprimées).
  
## Les différents états de la relation entre deux voisins BGP ("neighbor state")

- Idle (démarrage de la session connexion au voisin)
- Connect (après 3WayHandshake -> reset -> message OPEN envoyé par le routeur).
- Active (l'autre routeur répond par un message Open vers OpenSet)
- OpenSent (sans erreurs migration vers OpenConfirm)
- OpenConfirm (BGP attend des messages KEEPALIVE ou NOTIFICATION)
- Established (échange de routes via le message UPDATE)

## Intérêt de l'agrégation de préfixes avec BGP

- Afin de réduire la taille des tables de routages.
- Améliorer la stabilité en cachant les "route flaps". 

## RFC

- RFC 1105 juin 1989.
- RFC 1654 juillet 1994.
- RFC 2283 février 1998.

