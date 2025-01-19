
# Test de réseau avec LAN, VLAN, NAT et DNS

## Partie 1: Ping entre clients du LAN1 et du LAN2

### Configuration IP de PC1
```
NAME        : PC1[1]
IP/MASK     : 10.3.1.1/24
GATEWAY     : 10.3.1.254
DNS         :
MAC         : 00:50:79:66:68:00
LPORT       : 10004
RHOST:PORT  : 127.0.0.1:10005
MTU:        : 1500
```

### Ping de PC1 vers 10.3.2.1
```
84 bytes from 10.3.2.1 icmp_seq=1 ttl=62 time=29.002 ms
84 bytes from 10.3.2.1 icmp_seq=2 ttl=62 time=32.048 ms
84 bytes from 10.3.2.1 icmp_seq=3 ttl=62 time=41.000 ms
84 bytes from 10.3.2.1 icmp_seq=4 ttl=62 time=28.534 ms
84 bytes from 10.3.2.1 icmp_seq=5 ttl=62 time=35.602 ms
```

### Adresses MAC des routeurs (R2)
```
Protocol  Address          Age (min)  Hardware Addr   Type   Interface
Internet  10.3.12.1              59   c401.0575.0001  ARPA   FastEthernet0/0
Internet  10.3.12.2               -   c402.05e4.0000  ARPA   FastEthernet0/0
Internet  10.3.2.2               38   0050.7966.6801  ARPA   FastEthernet0/1
Internet  10.3.2.1                5   0050.7966.6800  ARPA   FastEthernet0/1
Internet  10.3.2.254              -   c402.05e4.0001  ARPA   FastEthernet0/1
```

### Accès Internet via R1
#### Depuis R1
```
R1#ping 8.8.8.8
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 24/56/92 ms
```
#### Depuis LAN1
```
PC1> ping 8.8.8.8
84 bytes from 8.8.8.8 icmp_seq=1 ttl=113 time=30.484 ms
84 bytes from 8.8.8.8 icmp_seq=2 ttl=113 time=30.273 ms
84 bytes from 8.8.8.8 icmp_seq=3 ttl=113 time=31.482 ms
84 bytes from 8.8.8.8 icmp_seq=4 ttl=113 time=22.627 ms
84 bytes from 8.8.8.8 icmp_seq=5 ttl=113 time=32.959 ms
```
#### Depuis LAN2
```
PC3> ping 8.8.8.8
84 bytes from 8.8.8.8 icmp_seq=1 ttl=112 time=41.154 ms
84 bytes from 8.8.8.8 icmp_seq=2 ttl=112 time=40.512 ms
84 bytes from 8.8.8.8 icmp_seq=3 ttl=112 time=44.176 ms
84 bytes from 8.8.8.8 icmp_seq=4 ttl=112 time=33.617 ms
84 bytes from 8.8.8.8 icmp_seq=5 ttl=112 time=35.611 ms
```

---

## Partie 2: VLAN

### Test ping entre PC1 et PC2 (même VLAN)
```
PC1> ping 10.3.1.2
84 bytes from 10.3.1.2 icmp_seq=1 ttl=64 time=2.778 ms
84 bytes from 10.3.1.2 icmp_seq=2 ttl=64 time=2.868 ms
84 bytes from 10.3.1.2 icmp_seq=3 ttl=64 time=3.087 ms
84 bytes from 10.3.1.2 icmp_seq=4 ttl=64 time=2.719 ms
84 bytes from 10.3.1.2 icmp_seq=5 ttl=64 time=2.738 ms
```

### Test ping depuis R1
```
R1#ping 10.3.1.1
!!!!!
R1#ping 10.3.2.1
!!!!!
R1#ping 10.3.1.2
!!!!!
```

### Ping entre VLAN 1 et VLAN 2
```
PC1> ping 10.3.2.1
84 bytes from 10.3.2.1 icmp_seq=1 ttl=63 time=15.255 ms
84 bytes from 10.3.2.1 icmp_seq=2 ttl=63 time=15.914 ms
84 bytes from 10.3.2.1 icmp_seq=3 ttl=63 time=20.622 ms
```

---

## Partie 3: DHCP et DNS

### Obtention d’adresse IP par DHCP
```
PC4> dhcp
DORA IP 10.3.2.10/24 GW 10.3.2.254
```
### Configuration obtenue
```
NAME        : PC4[1]
IP/MASK     : 10.3.2.10/24
GATEWAY     : 10.3.2.254
DNS         : 1.1.1.1
DHCP SERVER : 10.3.2.253
```
### Ping d’une adresse DNS publique
```
PC4> ping 1.1.1.1
84 bytes from 1.1.1.1 icmp_seq=1 ttl=59 time=20.809 ms
```

### Test DNS résolu
```
PC4> ping efrei.fr
efrei.fr resolved to 51.255.68.208
84 bytes from 51.255.68.208 icmp_seq=1 ttl=59 time=43.690 ms
```

---

## Partie 4: Requêtes avancées et simulation d’attaque

### Requête AXFR
```
[root@localhost ~]$ dig axfr @dns.tp3.b2 tp3.b2
; <<>> DiG 9.16.23-RH <<>> axfr @dns.tp3.b2 tp3.b2
; (1 server found)
tp3.b2. 86400 IN SOA dns.tp3.b2. admin.tp3.b2. 2019061800 3600 1800 604800 86400
tp3.b2. 86400 IN NS dns.tp3.b2. 
coolsite.tp3.b2. 86400 IN A 10.3.3.4 
...
```

### Simulation DNS spoof avec Scapy
```python
from scapy.all import *

answer = sr1(IP(dst="dns.tp3.b2", src="10.3.2.10")/UDP(dport=53)/DNS(rd=1,qd=DNSQR(qname="tp3.b2", qtype="AXFR")),verbose=0)
print(answer[DNS].summary())
```

### TCP RST
```
[root@localhost root]# python3 rst.py
ip src >> 10.3.3.1
ip dst >> 10.3.3.2
port src >> 56100
port dst >> 22
Seq nb >> 2819881537
Ack nb >> 3424035478
```

### Capture résultats
```
[ 3029.781295 ] device enp0s3 entered promiscuous mode
[ 3029.781295 ] device enp0s3 left promiscuous mode
```
