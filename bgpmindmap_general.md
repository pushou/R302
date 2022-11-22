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
- La table BGP est appelée Loc-Rib table


## Sessions

- eBGP
  - TTL à 1  (le routeur averti doit être en frontal du notre)
  - Le routeur averti positionne le NEXT_HOP pour cet AS à l'IP source du routeur établissant la connexion
  - Le routeur averti ajoute son ASN dans l'AS_PATH
  - Le routeur averti vérifie que son AS  
  - Distance administrative 20.
- iBGP 
  - Fait pour le transit (TTL à 255)
  - On ne peut pas redistribuer les routes BGP dans l'IGP (C'est trop de route - Il n'y a pas 
    d'équivalence des métriques IGP ni de traduction des attributs BGP dans le "langage" de l'IGP.
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

