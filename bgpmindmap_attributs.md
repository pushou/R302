---
markmap:
  colorFreezeLevel: 4
  maxWidth: 500


---
# Attributs BGP
## BGP supportent 4 types d'attributs (pas tous reconnus par les constructeurs)
 
### **Well-Known Mandatory**

- **AS_PATH** : Il contient la liste des AS pour arriver à une destination, incrémenté par l'identifiant de l'AS courante.
- **NEXT_HOP** : C'est l'adresse IP du voisin qui a envoyé la mise à jour. Les paquets seront routés à cette adresse pour entrer dans l'AS. 
- **ORIGIN** : Elle contient une information sur l'origine de la route.
  - **i** : L'origine est une déclaration à l'aide de la commande Network. 
  - **?** : L'origine est inconnue, ou vient d'une redistribution de routes.
  
### **Well-Known Discretionary** (attributs reconnus par tous mais qui ne font pas forcément partie de la mise à jour des routes).
  - LOCAL_PREFERENCE : Il indique aux routeurs BGP d'une même AS quel est le chemin préféré pour une destination donnée.
    - Un chemin avec une préférence locale plus élevée est plus prioritaire.
    - La valeur par défaut chez Cisco est de 100.
    - Cette valeur est configurée et envoyée aux routeurs iBGP mais pas aux voisins eBGP.
    - Le routeur ayant la plus  grand valeur LOCAL_PREF est choisi comme routeur de sortie.
  - ATOMIC_AGGREGATE_ID : il indique que la route annoncée a été agrégée et permet de tracer l'origine de l'agrégation (une route contient l'ID du routeur, de son AS ainsi que le supernet).
  - Cluster List

### **Optional Transitive**  (Attribut créé par un constructeur et pas forcément reconnu par d'autres. Transitif, BGP l'accepte et le transmet même si son implémentation ne le reconnaît pas).

    - COMMUNITY peut être utilisé pour filtrer des routes (identique à un TAG).  
    - Par exemple une communauté "No_advertise" indique que les routes tagguées ainsi ne doivent pas être annoncées dans l'AS. La communauté No_Export indique que les routes ne doivent pas être transmises à un autre peer eBPG.

### **Optional Nontransitive**

    - L'attribut **MED**:  (Multiple Exit Discriminator) donne des indices aux routeurs extérieurs sur le chemin préféré pour contacter un AS qui a plusieurs points d'entrée.
        - Un MED petit est préféré à un grand MED ! Le MED est envoyé aux pairs eBGP, qui informent leurs peer IBGP.
        - Les routeurs IBGP à l'intérieur de l'AS peuvent utiliser le MED, mais ils ne peuvent pas l'envoyer à un routeur eBGP.
    
    - L'attribut **WEIGHT** est propriétaire de CISCO. 
        - Il est utilisé quand un routeur à plusieurs peers vers un AS donné. 
        - Il est local au routeur et n'est jamais propagé lors d'un update BGP.
        - Sa valeur va de 0 à 65535. Le chemin ayant le poids le plus fort est préféré. 


## Algorithme de sélection du meilleur chemin avec BGP

  1. Si on a plusieurs routes vers un réseau, on choisit le plus long chemin qui correspond à la destination (exemple: entre 194.199.0.0/16 et 194.199.227.0/24 on choisit le second chemin car c'est une route plus spécifique).
  2. Si les routes sont identiques on choisit la distance administrative la plus faible. Elle est de 20 pour eBGP et seules les routes connectées, les routes statiques les résumés de routes EIGRP sont plus prioritaires. 
- 3.Algorithme de choix en fonction des attributs du plus important au moins important :
  - WEIGHT (la plus élevée)
  - LOCAL_PREF (la plus élevée)
  - Réseaux annoncés avec les commandes "network" ou "aggregate" ou "redistribute".
  - Métriques IGP.
  - AS_PATH le plus court
  - Les chemins dont l'ORIGINE est un IGP sont préférés à ceux dont l'origine est EGP , eux-mêmes préférés aux chemins avec une origine incomplète ou inconnue.
  - Chemins avec la valeur la MED plus basse.
  - Chemins appris par eBGP préférés à ceux choisis par iBGP.
  - Chemins avec le plus petit "router ID".
  - Chemins qui comportent le moins de sauts iBGP.
  - Chemins qui viennent du voisin avec l'adresse IP la plus basse.