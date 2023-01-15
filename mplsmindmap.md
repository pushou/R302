---
markmap:
  colorFreezeLevel: 2
  maxWidth: 700
---

# MPLS

## Caractéristiques et intérêts


- MPLS permet de faire du "Traffic engineering" (partage de charge, de la QoS, VPN).
- Le réseau MPLS est opéré par une seule entité. Seul le passage au SDN (Software Defined Networking) permet de piloter plusieurs fournisseurs de réseaux afin choisir le meilleur chemin.


## Les VRF sont nécessaires pour MPLS

 - Un routeur physique peut gérer plusieurs tables de routages : en général une par client. Ce sont les VRF (Virtual Routing & Forwarding).
 - Chaque VRF correspond à une table de routage (une pour chaque client du FAI). 
 - Les VRF permettent à plusieurs clients d'utiliser le même plan d'adressage privé sans craindre l'"overlaps" des réseaux. 
 - Les "VRF lite" sont des VRF sans MPLS. Elles sont utiles sur des réseaux de petites à moyennes tailles.

## Définitions 

- MPLS c'est de la commutation de paquets: il n'y a pas du parcours des tables de routage pour trouver le meilleur chemin. Mais si la commutation de label était plus rapide que le routage ce n'est plus le cas actuellement. Ce n'est donc plus un des raisons qui a fait le succès de MPLS.
- C'est une technologie "de couches entre 2 et 3" (2,5).
- Elle est basée sur la labellisation :une étiquette est ajoutée au paquet.
- Le paquet est transmis par le routeur suivant en fonction de ce label qui lui-même en appose un. En conséquence, un label a une signification locale et le même label peut apparaître plusieurs fois.
- MPLS transporte de l'IP, de l'Ethernet ou de l'ATM de façon agnostique dans un **circuit virtuel (VC)**.

 
## Description des en-têtes des paquets MPLS

- 012345678901234567890123456789  
 |-------LABEL---------|TC|S|TTL|
  Label : 20 premiers bits sont des entiers courts
  TC : 3 bits utilisés pour la QoS
  Stack : indique la fin de l'empilement

## Les différents types de routeurs MPLS

- **LER (Label Edge Router)** : ce sont les points d'entrée (ingress) et de sortie (egress) du réseau MPLS. l'ingress LER reçoit le paquet, détermine son étiquette. À la sortie du réseau l'egress LER enlève les labels. Ils effectuent des opérations complexes.
- **LSR (Label Switch Router)** ou "transit node" : Routeur commutateur de labels MPLS. 
  - Chaque LSR possède une table de label propre.
  - Chaque LSR assigne des labels à ses FEC.
  - Les labels sont assignés et échangés entre LSR adjacents.
  - Le VPN et l'ingénierie de trafic nécessitent des échanges de labels entre voisins distants.
- Les LER sont aussi appelés PE (Provider Edge) et les LSR des P (Provider) dans la terminologie Cisco. Les CE (Customer Edge) sont les routeurs des clients en frontal des LER ou PE.
- Terminologie Cisco
  CE -> PE  →P  P ... →E -> CE
  avec la terminologie Officielle
  CE -> ingress LER -> LSR - LSR -egress LER.

## FEC ou classe d'équivalence

- FEC ("Forwarding Equivalent Class" ou "Classe d'équivalence") : on réunit les paquets ayant les mêmes caractéristiques (IP|source ou IP|réseau de destination ou autres...) dans une même FEC. Une FEC est souvent identifiée par le réseau de destination.
- Les couples de labels sortants et rentrants matérialisent pour chaque FEC un chemin ou **circuit virtuel** des paquets. C'est le LSP (**Label Switching Path**). Il y a un LSP différent pour l'aller et pour le retour.
- Une FEC peut être travaillée manuellement en fonction de critères plus précis (ports , adresse sources...) pour différencier certains flux, par exemple pour la QoS. En mode automatique il faut que l'ensemble des LSR échangent les labels.

## Traffic Engineering

- Le "traffic engineering" permet d'avoir des contraintes additionnelles sur le meilleur chemin (par exemple pour avoir le plus court chemin **et** avec la meilleure bande passante).
- Le protocole RSVP permet de réserver de la bande passante.
- Chaque chemin de commutation de label (Label Switching Path) peut être associé à une réservation de bande passante. Cette réservation peut être calculée de façon automatique ("auto bandwidth").
- "MPLS Fast Reroute" améliore la convergence lors d'un incident. MPLS-FR a pour but de pré-calculer des LSP de secours et pré-implémenter la FIB pour gagner du temps.


## Dynamique du "switching MPLS"

  1. Le paquet entre sur le réseau.
  2. Le LER détermine une FEC pour le paquet et appose une étiquette contenant un label de sortie (opération **PUSH**).
  3. Le LER transmet le paquet au LSR.
  4. Le LSR modifie l'étiquette (opération **SWAP**) : le Label de sortie devient le label d'entrée et un nouveau label de sortie est apposé.
  5. Le paquet est transmis de LER en LER avec plusieurs **swap**.
  6. Le paquet arrive au LSR de sortie qui dépile l'étiquette (opération **POP**) et transmet le paquet IP qui y était contenu au réseau du client.


## LDP: Protocole de Distribution de Labels

- LDP transporte les informations nécessaires à la construction des tables de labels sur les LSR.
- Les LSR connaissent les autres routeurs grâce à un IGP.

## Le modèle VPN MPLS 


- Les VPN MPLS sont de niveau 2 (Layer 2 Pseudowire) ou 3 (L3VPN).
- Le "MultiProtocol BGP" (MP-BGP) est une extension au protocole de routage BGP qui permet à différentes familles d’adresses d’être distribuées en parallèle entre routeurs (en l'occurrence pour les L3VPN : adresses VPN IPv4/v6 avec labels MPLS). Les routeurs d'un réseau MPLS communiquent ainsi en i-MP-BGP (interior MultiProtocol BGP). 
- Les PE et CE échangent des informations au travers de protocoles comme BGP, OSPF ou des routes statiques. Ils reçoivent des annonces de routes.


## Route Distinguisher et Route Target (source wikipédia)

- Une VRF a un "**Route Distinguisher**" unique. Le format de RD est souvent "Numéro de l'AS :Numéro Assigné". Le numéro assigné dépend de l'opérateur:
    -  Exemple pour le VRF client_A :
    IP VRF client_A rd 65535:1
    - Le préfixe suivant : 192.168.0.0 de la VRF client_A prendra alors cette forme :
    65535:1:192.168.0.0
- Le **route target (RT)** est une information supplémentaire associée à tout préfixe d'une VRF lors de son export vers BGP. Cette valeur peut être égale au RD (route distinguisher) ou pas. Elle est utilisée pour distinguer et filtrer les préfixes reçus en MP-BGP afin de déterminer vers quelles VRF locales ils doivent être importés. Du point de vue BGP il s'agit d'une communauté BGP étendue (64 bits). Un préfixe peut recevoir un ou plusieurs RT. 

## MPLS et traceroute

- La décrémentation du TTL n'est pas demandée lors de la traversée d'un routeur par MPLS  ce qui est problématique pour faire un traceroute ... et donc voir le chemin emprunté par les paquets afin de diagnostiquer d'éventuels problèmes.
- On peut néanmoins l'implémenter et faire du traceroute avec un réseau MPLS.

## La sécurité avec MPLS

- La sécurité du réseau MPLS repose sur :
  - L'indépendance des flux.
  - Le pilotage et la sécurité du réseau qui sont assurées par un seul fournisseur. 
- On peut néanmoins s'interroger sur le chiffrement des données qui est absent de MPLS et sur le fait que c'est le niveau de sécurité du fournisseur qui détermine le niveau de sécurité du réseau.
  