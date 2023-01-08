---
markmap:
  colorFreezeLevel: 2
  maxWidth: 800
---
# Commandes Linux (AS+LABEL)


```bash
  #mtr
  mtr --tcp --port 443 --show-ips --mpls --aslookup --curses www.umontpellier.fr
  mtr -P 443 -T  -b -e -z -t 194.199.227.220
  mtr  -P 443 -T -c 2  -b -e -z -t -j 194.199.227.220 | from json|flatten|get hubs|flatten
```

```bash
# traceroute
traceroute -A -n -e -i eth0 8.8.8.8
```
# Commandes Cisco

## mpls config

- AutoConfig LDP
  ```ios
  router ospf 1
      mpls ldp autoconfig
  ```

- ``` ios
  # cef améliore les performances (distributed c'est mieux) 
  conf t
     ip cef distributed
  ```

- ```ios
  interface FastEthernet2/0
         mpls ip
  # En complément
  ```

## Debug mpls

- ```ios
  sh mpls forwarding-table  
  Local  Outgoing    Prefix            Bytes tag  Outgoing   Next Hop    
  tag    tag or VC   or Tunnel Id      switched   interface              
  16     Pop tag     10.0.24.0/30      0          Fa2/0      10.0.12.2    
  17     Pop tag     10.0.23.0/30      0          Fa2/0      10.0.12.2    
  18     Pop tag     2.2.2.2/32        0          Fa2/0      10.0.12.2    
  19     16          3.3.3.3/32        0          Fa2/0      10.0.12.2    
  20     17          4.4.4.4/32        0          Fa2/0      10.0.12.2    
  21     Untagged    5.0.0.0/8[V]      0          Fa1/0      192.168.1.5  
  22     Aggregate   192.168.1.0/24[V] 0                                  
  23     Untagged    6.0.0.0/8[V]      0          Fa3/0      192.168.1.6  
  24     Aggregate   192.168.1.0/24[V] 0                                  
  25     Untagged    11.11.11.11/32[V] 0          Fa0/0      192.168.1.11 
  26     Aggregate   192.168.1.0/24[V] 0    

- ```ios
  R1#show tag-switching tdp neighbor 
    Peer LDP Ident: 2.2.2.2:0; Local LDP Ident 1.1.1.1:0
	TCP connection: 2.2.2.2.62837 - 1.1.1.1.646
	State: Oper; Msgs sent/rcvd: 141/142; Downstream
	Up time: 01:55:58
	LDP discovery sources:
	 FastEthernet2/0, Src IP addr: 10.0.12.2
        Addresses bound to peer LDP Ident:
          10.0.12.2       2.2.2.2         10.0.23.2       10.0.24.2     
  ```

- ```ios
  R1#sh bgp vpnv4 unicast all summary 
  BGP router identifier 1.1.1.1, local AS number 200
  BGP table version is 33, main routing table version 33
  15 network entries using 2055 bytes of memory
  20 path entries using 1360 bytes of memory
  17/14 BGP path/bestpath attribute entries using 2108 bytes of memory
  7 BGP extended community entries using 360 bytes of memory
  0 BGP route-map cache entries using 0 bytes of memory
  0 BGP filter-list cache entries using 0 bytes of memory
  BGP using 5883 total bytes of memory
  BGP activity 15/0 prefixes, 22/2 paths, scan interval 15 secs
 
  Neighbor        V    AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
  3.3.3.3         4   200      71      73       33    0    0  00:58:42        7
  4.4.4.4         4   200      71      74       33    0    0  00:58:39        7
  ```

- ```ios
  # Véfifier la correspondance label fec
  R1#show tag-switching tdp neighbor 
    Peer LDP Ident: 2.2.2.2:0; Local LDP Ident 1.1.1.1:0
	 TCP connection: 2.2.2.2.62837 - 1.1.1.1.646
	 State: Oper; Msgs sent/rcvd: 141/142; Downstream
	 Up time: 01:55:58
	LDP discovery sources:
	 FastEthernet2/0, Src IP addr: 10.0.12.2
        Addresses bound to peer LDP Ident:
          10.0.12.2       2.2.2.2         10.0.23.2       10.0.24.2     
  ```

- ```ios
  # vérifiez la forwarding table
  R3#show tag-switching forwarding-table 
  Local  Outgoing    Prefix            Bytes tag  Outgoing   Next Hop    
  tag    tag or VC   or Tunnel Id      switched   interface              
  16     Pop tag     10.0.24.0/30      0          Fa0/0      10.0.23.2    
  17     Pop tag     2.2.2.2/32        0          Fa0/0      10.0.23.2    
  18     17          4.4.4.4/32        0          Fa0/0      10.0.23.2    
  19     Pop tag     10.0.12.0/30      0          Fa0/0      10.0.23.2    
  20     18          1.1.1.1/32        0          Fa0/0      10.0.23.2    
  21     Untagged    7.0.0.0/8[V]      0          Fa1/0      192.168.3.7  
  22     Aggregate   192.168.3.0/24[V] 0                                  
  23     Untagged    9.9.9.9/32[V]     0          Fa2/0      192.168.3.9  
  24     Untagged    10.10.10.10/32[V] 0          Fa2/0      192.168.3.9  
  25     Aggregate   192.168.3.0/24[V] 0                                  
  26     Untagged    192.168.4.0/24[V] 0          Fa2/0      192.168.3.9  
  27     Untagged    192.168.90.0/24[V]   \                                        0          Fa2/0      192.168.3.9     
  ```