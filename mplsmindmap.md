---
markmap:
  colorFreezeLevel: 2
  maxWidth: 700
---

# MPLS

## Intérêts

- Au début la commutation de label était plus rapide que le routage mais ce n'est plus le cas actuellement.
- MPLS permet de faire du "Traffic engineering" (partage de charge, de la QoS, du VPN.)
- Les VPN MPLS sont de niveau 2 (Layer 2 Pseudowire) ou 3(L3VPN).
- Le réseau MPLS est opéré par une seule entité. Seul le passage au  SDN (Software Defined Networking) permet de piloter plusieurs fournisseurs de réseaux afin choisir le meilleur chemin. 



## Les VRF sont nécessaires pour MPLS

 - Un routeur physique peut gérer plusieurs tables de routages: une par client. Ce sont les VRF(Virtual Routing & Forwarding).
 - Chaque VRF correspond à une table de routage (une pour chaque client du FAI). 
 - Les VRF permettent à plusieurs clients d'utiliser le même plan d'adressage privé sans craindre l'"overlaps" des réseaux.  

## Définitions 

- MPLS c'est de la commutation de paquets pas du parcours des tables de routage pour trouver le meilleur chemin.
- Technologie de couche entre 2 et 3 (2,5).
- Basée sur la labellisation: une étiquette est ajoutée au paquet.
- Le paquet et transmis par le routeur suivant en fonction de ce label qui lui-même en appose un. En conséquence un label a une signification locale et on peut donc avoir un label apparaissant plusieurs fois.
- MPLS transporte de l'IP, de l'éthernet ou de l'ATM de façon agnostique dans un circuit virtuel (VC).

 
## Caractéristiques des paquets

- 012345678901234567890123456789  
 |-------LABEL---------|TC|S|TTL|
  Label: 20 premiers bits entier court
  TC: 3 bits utilisé pour la QoS
  Stack: indique la fin de l'empilement

## Routeurs

- LER(Label Edge Router) : Points d'entrée (ingress) et de sortie (egress) du réseau MPLS. l' ingress LER reçoit le paquet , détermine son étiquette.  A la sortie du réseau l'egress LER enlève les labels. Ils effectuent des opérations complexes.
- LSR(Label Switch Router) ou "transit node" : Routeur commutateur de labels MPLS. 
  - Chaque LSR possède une table de label propre.
  - Chaque LSR assignes des labels à ses FEC.
  - Les labels sont assignés et échangés entre LSR adjacents.
  - Le VPN et le TE nécessitent des échanges de labels entre voisins distants.
- Les LER sont aussi appelés PE (Provider Edge) et les LSR des P (Provider) dans la terminologie Cisco. Les CE (Customer Edge) sont les routeurs des clients en frontal des LER ou PE.
- Terminologie Cisco
  CE -> PE  -> P -> P ... -> PE -> CE
  avec la terminologie Officielle
  CE -> ingress LER -> LSR - LSR

## FEC ou classe d'équivalence.

- FEC(Forwarding Equivalent Class= Classe d'équivalence): on réunit les paquets ayant les mêmes caractéristiques (IP|source source ou IP|réseau de destination ou ...) dans une même FEC. Une FEC  est identifiée par le réseau de destination.
- Les couples de labels sortants et rentrants matérialisent pour chaque FEC un chemin ou **circuit virtuel** des paquets ou LSP(Label Switching Path). Il y a un LSP pour l'aller et un autre pour le retour.
- Une FEC peut être travaillée manuellement en fonction de critères plus précis (ports , adresse sources...) pour différencier certains flux, par exemple pour la QoS. En mode automatique il faut que l'ensemble des LSR échangent les labels.

## Traffic Engineering

- Le "traffic engineering" permet d'avoir des contraintes additionnelles sur le meilleur chemin (par exemple on veut le plus court chemin avec la meilleure bande passante).
- Le protocole RSVP, permet de réserver de la bande passante.
- Chaque chemin de commutation de label (Label Switching Path) peut être associé à une réservation de bande passante. Cette réservation peut être calculée de façon automatique (auto bandwidth) 
- "MPLS Fast Reroute" améliore la convergence lors d'un incident. MPLS FR a pour but de pré-calculer des LSP de secours et pré-implémenter la FIB pour gagner du temps.
- "MPLS Protection Schemes"

## Dynamique du switching MPLS

  1. Le paquet entre sur le réseau.
  2. Le LER détermine une FEC pour le paquet et appose une étiquette contenant un label de sortie (opération **PUSH**).
  3. Le LER transmet le paquet au LSR.
  4. Le LSR modifie l'étiquette (opération **SWAP**): le Label de sortie devient le label d'entrée et un un nouveau label de sortie est apposé)
  5. Le paquet est transmis de LER en LER.
  6. Le paquet arrive au LSR de sortie qui dépile l'étiquette(opération **POP**)et transmet le paquet IP qui y était contenu. 

## LDP: Protocole de Distribution de Label

- LDP transporte les informations nécessaires à la construction des tables de labels sur les LSR.
- Les LSR connaissent les autres routeurs grâce à un IGP.

 

## Le modèle VPN MPLS 

- "VRF lite" ce sont des VRF sans MPLS.
- Le protocole MP-BGP permet de transporter des routes de plusieurs familles d'adresse 
- Les PE et CE échanges des informations au travers de protocoles comme BGP, OSPF ou des routes statiques. Ils reçoivent des annonce de routes
- Les PE translatent les routes en VPN-IPV4:
     - Ajout du SOO (Site of origin).
     - Ajout du "Route Target"

## Route Distinguisher Target

- Une VRF a un "Route Distinguisher" unique sur le PE. Le format de RD est souvent "Numéro de l'AS: Numéro Assigné". Le numéro assigné dépend de l'opérateur 
- Les VRF sont raccordés entre elles via des RouteTarget importées et exportées sur les PE.

## MPLS et traceroute

- Le mécanisme  de décrémentation du TTL n'est pas demandé par MPLS ce qui est problématique pour un traceroute... et donc voir le chemin emprunté par les paquets.
- On peut néanmoins l'implémenter...

## La sécurité avec MPLS

 - La sécurité du réseau MPLS repose sur:
    - L'indépendance des flux.
    - Le pilotage et la sécurité du réseau est assurée par un seul Provider. 
- On peut néanmoins s'interroger sur dle chiffrement des données qui est absent de MPLS et sur le fait que c'est le niveau de sécurité du fournisseur qui détermine le niveau de sécurité. 
  
