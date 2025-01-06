
# TP 1 : Configuration réseau entre deux machines

## Étape 1 : Déterminer les adresses MAC

### Commande :
Montrez l'adresse Mac :
```bash


show
```

### Résultats :
- **PC2** :
  ```
  NAME   IP/MASK              GATEWAY           MAC                LPORT  RHOST:PORT
  PC2    0.0.0.0/0            0.0.0.0           00:50:79:66:68:01  20004  127.0.0.1:20005
         fe80::250:79ff:fe66:6801/64
  ```
- **PC1** :
  ```
  NAME   IP/MASK              GATEWAY           MAC                LPORT  RHOST:PORT
  PC1    0.0.0.0/0            0.0.0.0           00:50:79:66:68:00  20002  127.0.0.1:20003
         fe80::250:79ff:fe66:6800/64
  ```

---

## Étape 2 : Définir une adresse IP statique

### Commande pour afficher les options disponibles :
afficher les options de configuration :
```bash
ip
```

### Options disponibles :
```
ip ARG ... [OPTION]
  Configure the current VPC's IP settings
    ARG ...:
    address [mask] [gateway]
    address [gateway] [mask]
                   Set the VPC's IP, default gateway IP, and network mask
                   Default IPv4 mask is /24, IPv6 is /64.
                   Example: ip 10.1.1.70/26 10.1.1.65
                   Sets the VPC's IP to 10.1.1.70, the gateway to 10.1.1.65,
                   and the netmask to 255.255.255.192.
    auto           Attempt to obtain IPv6 address, mask, and gateway using SLAAC
    dhcp [OPTION]  Attempt to obtain IPv4 address, mask, gateway, DNS via DHCP
          -d         Show DHCP packet decode
          -r         Renew DHCP lease
          -x         Release DHCP lease
    dns ip         Set DNS server IP, delete if IP is '0'
    dns6 ipv6      Set DNS server IPv6, delete if IPv6 is '0'
    domain NAME    Set local domain name to NAME
```

### Configuration des adresses IP statiques :
- **PC1** :
  ```bash
  ip 10.10.0.2/24
  ```
  Résultat :
  ```
  Checking for duplicate address...
  PC1 : 10.10.0.2 255.255.255.0
  ```

- **PC2** :
  ```bash
  ip 10.10.0.1
  ```
  Résultat :
  ```
  Checking for duplicate address...
  PC2 : 10.10.0.1 255.255.255.0
  ```

---

## Étape 3 : Vérifier la configuration

### Commande :
```bash

save

show ip
```

### Résultats :
- **PC1** :
  ```
  NAME        : PC1[1]
  IP/MASK     : 10.10.0.2/24
  GATEWAY     : 0.0.0.0
  DNS         :
  MAC         : 00:50:79:66:68:00
  LPORT       : 20002
  RHOST:PORT  : 127.0.0.1:20003
  MTU         : 1500
  ```

- **PC2** :
  ```
  NAME        : PC2[1]
  IP/MASK     : 10.10.0.1/24
  GATEWAY     : 0.0.0.0
  DNS         :
  MAC         : 00:50:79:66:68:01
  LPORT       : 20004
  RHOST:PORT  : 127.0.0.1:20005
  MTU         : 1500
  ```

---

## Étape 4 : Tester la connectivité (Ping)

### Depuis **PC2**, ping vers **PC1** :
Commande :
```bash
ping 10.10.0.2
```

Résultat :
```
84 bytes from 10.10.0.2 icmp_seq=1 ttl=64 time=2.542 ms
84 bytes from 10.10.0.2 icmp_seq=2 ttl=64 time=4.370 ms
84 bytes from 10.10.0.2 icmp_seq=3 ttl=64 time=1.748 ms
84 bytes from 10.10.0.2 icmp_seq=4 ttl=64 time=2.509 ms
84 bytes from 10.10.0.2 icmp_seq=5 ttl=64 time=3.984 ms
```

### Depuis **PC1**, ping vers **PC2** :
Commande :
```bash
ping 10.10.0.1
```

Résultat :
```
84 bytes from 10.10.0.1 icmp_seq=1 ttl=64 time=1.236 ms
84 bytes from 10.10.0.1 icmp_seq=2 ttl=64 time=3.580 ms
84 bytes from 10.10.0.1 icmp_seq=3 ttl=64 time=9.462 ms
84 bytes from 10.10.0.1 icmp_seq=4 ttl=64 time=2.940 ms
84 bytes from 10.10.0.1 icmp_seq=5 ttl=64 time=2.440 ms
```

---

# Analyse avec Wireshark

## Protocole Utilisé
- **ICMP**


# Commandes ARP

#### PC2
```bash
PC2> show arp
00:50:79:66:68:00  10.10.0.2 expires in 113 seconds
```

#### PC1
```bash
PC1> show arp
00:50:79:66:68:01  10.10.0.1 expires in 89 seconds
```

---

## Ajout d'un switch

### Déterminer l'adresse MAC
```bash
VPCS> show
NAME   IP/MASK              GATEWAY           MAC                LPORT  RHOST:PORT
VPCS1  0.0.0.0/0            0.0.0.0           00:50:79:66:68:02  20006  127.0.0.1:20007
       fe80::250:79ff:fe66:6802/64
```

### Déterminer une IP fixe pour la machine 3
```bash
VPCS> ip 10.10.0.3/24
Checking for duplicate address...
VPCS : 10.10.0.3 255.255.255.0
```

#### Sauvegarder et vérifier l'IP
```bash
save
show ip
```

---

## Pings entre les machines

### Node 1 à Node 2
```bash
PC1> ping 10.10.0.2
84 bytes from 10.10.0.2 icmp_seq=1 ttl=64 time=1.946 ms
84 bytes from 10.10.0.2 icmp_seq=2 ttl=64 time=4.524 ms
84 bytes from 10.10.0.2 icmp_seq=3 ttl=64 time=1.576 ms
84 bytes from 10.10.0.2 icmp_seq=4 ttl=64 time=2.329 ms
84 bytes from 10.10.0.2 icmp_seq=5 ttl=64 time=7.600 ms
```

### Node 2 à Node 3
```bash
PC2> ping 10.10.0.3
84 bytes from 10.10.0.3 icmp_seq=1 ttl=64 time=2.387 ms
84 bytes from 10.10.0.3 icmp_seq=2 ttl=64 time=2.069 ms
84 bytes from 10.10.0.3 icmp_seq=3 ttl=64 time=3.710 ms
84 bytes from 10.10.0.3 icmp_seq=4 ttl=64 time=3.875 ms
84 bytes from 10.10.0.3 icmp_seq=5 ttl=64 time=3.394 ms
```

### Node 1 à Node 3
```bash
PC1> ping 10.10.0.3
84 bytes from 10.10.0.3 icmp_seq=1 ttl=64 time=4.130 ms
84 bytes from 10.10.0.3 icmp_seq=2 ttl=64 time=3.244 ms
84 bytes from 10.10.0.3 icmp_seq=3 ttl=64 time=3.716 ms
84 bytes from 10.10.0.3 icmp_seq=4 ttl=64 time=1.685 ms
84 bytes from 10.10.0.3 icmp_seq=5 ttl=64 time=5.272 ms
```

---

## Fichier Wireshark
Le fichier de capture Wireshark peut être téléchargé ici :  
[Tram Ping Gns3.pcapng](/mnt/data/Tram Ping Gns3.pcapng)


# Donner un accès Internet au DHCP

## Vérification de la connexion Internet
J'ai ajouté une carte réseau, et pour vérifier la connectivité Internet :

```bash
ping google.com
```

Exemple de sortie :
```
PING google.com 

56(84) bytes of data.
64 bytes from mad01s26-in-f174.1e100.net (216.58.214.174): icmp_seq=1 ttl=63 time=37.0 ms
64 bytes from par10s42-in-f14.1e100.net (216.58.214.174): icmp_seq=2 ttl=63 time=29.4 ms
```

---

## Mise à jour de la VM
Pour mettre à jour la machine virtuelle, exécutez :
```bash
sudo dnf update -y
```

---

## Configuration statique de l'IP
1. Vérifiez les interfaces réseau disponibles :
   ```bash
   ip a
   ```

2. Éditez le fichier de configuration réseau pour l'interface `enp0s3` :
   ```bash
   vi /etc/sysconfig/network-scripts/ifcfg-enp0s3
   ```

   Ajoutez ou modifiez les lignes suivantes :
   ```
   DEVICE=enp0s3
   NAME=ok
   ONBOOT=yes
   BOOTPROTO=static
   IPADDR=10.1.1.10
   NETMASK=255.255.255.0
   ```

3. Rechargez les configurations réseau :
   ```bash
   sudo nmcli connection reload
   ```

4. Activez la connexion :
   ```bash
   sudo nmcli connection up ok 
   ```

5. Vérifiez que l'IP statique est appliquée :
   ```bash
   ip a
   ```

---

## Configuration du DHCP

1. Installez le serveur DHCP :
   ```bash
   sudo dnf -y install dhcp-server
   ```

2. Éditez le fichier de configuration DHCP :
   ```bash
   vi /etc/dhcp/dhcpd.conf
   ```

   Contenu du fichier :
   ```
   authoritative;
   subnet 10.1.1.0 netmask 255.255.255.0 {
       range 10.1.1.10 10.1.1.50;
       option broadcast-address 10.1.1.1;
       option routers 10.1.1.1;
   }
   ```

3. Activez et démarrez le service DHCP :
   ```bash
   systemctl enable --now dhcpd
   ```

---





### PC3

```bash
PC3> dhcp
DDORA IP 10.1.1.11/24 GW 10.1.1.1

PC3> ip show
Invalid address

PC3> show ip

NAME        : PC3[1]
IP/MASK     : 10.1.1.11/24
GATEWAY     : 10.1.1.1
DNS         :
DHCP SERVER : 10.1.1.10
DHCP LEASE  : 43078, 43200/21600/37800
MAC         : 00:50:79:66:68:02
LPORT       : 20011
RHOST:PORT  : 127.0.0.1:20012
MTU         : 1500
```

### PC1

```bash
PC1> dhcp
DDORA IP 10.1.1.12/24 GW 10.1.1.1

PC1> show ip

NAME        : PC1[1]
IP/MASK     : 10.1.1.12/24
GATEWAY     : 10.1.1.1
DNS         :
DHCP SERVER : 10.1.1.10
DHCP LEASE  : 43195, 43200/21600/37800
MAC         : 00:50:79:66:68:00
LPORT       : 20009
RHOST:PORT  : 127.0.0.1:20010
MTU         : 1500
```

### PC2

```bash
PC2> dhcp
DDORA IP 10.1.1.13/24 GW 10.1.1.1

PC2> show ip

NAME        : PC2[1]
IP/MASK     : 10.1.1.13/24
GATEWAY     : 10.1.1.1
DNS         :
DHCP SERVER : 10.1.1.10
DHCP LEASE  : 43189, 43200/21600/37800
MAC         : 00:50:79:66:68:01
LPORT       : 20007
RHOST:PORT  : 127.0.0.1:20008
MTU         : 1500
```

# DHCP Spoofing avec dnsmasq Setup et Test


## Installation et configuration dnsmasq

### Installation

```bash
dnf install -y dnsmasq
```

### Configuration

configuration file `/etc/dnsmasq.conf`:

```bash
port=0 #
dhcp-range=10.1.1.210,10.1.1.250,255.255.255.0,12h
dhcp-authoritative
interface=enp0s3
```

### Restart dnsmasq

```bash
systemctl restart dnsmasq
systemctl status dnsmasq
```

## Disable Legitimate DHCP

Stop le  DHCP server legit:

```bash
systemctl stop dhcpd
sudo systemctl disable dhcpd
```

## Firewall Rules

Regle Firewall pour dnsmasq:

```bash
firewall-cmd --add-service=dhcp --permanent
firewall-cmd --reload
```

## Test 

### DHCP Request on PC4

```bash
PC4> dhcp
DDORA IP 10.1.1.248/24 GW 10.1.1.1

PC4> show ip

NAME        : PC4[1]
IP/MASK     : 10.1.1.248/24
GATEWAY     : 10.1.1.1
DNS         :
DHCP SERVER : 10.1.1.1
DHCP LEASE  : 43192, 43200/21600/37800
MAC         : 00:50:79:66:68:03
LPORT       : 20016
RHOST:PORT  : 127.0.0.1:20017
MTU         : 1500
```

## Race !!

### RACE1

```bash
RACE1> dhcp
DORA IP 10.1.1.15/24 GW 10.1.1.1

RACE1> show ip

NAME        : RACE1[1]
IP/MASK     : 10.1.1.15/24
GATEWAY     : 10.1.1.1
DNS         :
DHCP SERVER : 10.1.1.10
DHCP LEASE  : 43054, 43072/21536/37688
MAC         : 00:50:79:66:68:04
LPORT       : 20016
RHOST:PORT  : 127.0.0.1:20017
MTU         : 1500
```

### RACE2

```bash
RACE2> dhcp
DDORA IP 10.1.1.250/24 GW 10.1.1.1

RACE2> show ip

NAME        : RACE2[1]
IP/MASK     : 10.1.1.250/24
GATEWAY     : 10.1.1.1
DNS         :
DHCP SERVER : 10.1.1.1
DHCP LEASE  : 43040, 43200/21600/37800
MAC         : 00:50:79:66:68:05
LPORT       : 20022
RHOST:PORT  : 127.0.0.1:20023
MTU         : 1500
```

### RACE3

```bash
RACE3> dhcp
DDORA IP 10.1.1.210/24 GW 10.1.1.1

RACE3> show ip

NAME        : RACE3[1]
IP/MASK     : 10.1.1.210/24
GATEWAY     : 10.1.1.1
DNS         :
DHCP SERVER : 10.1.1.1
DHCP LEASE  : 43016, 43200/21600/37800
MAC         : 00:50:79:66:68:06
LPORT       : 20024
RHOST:PORT  : 127.0.0.1:20025
MTU         : 1500
```

## WIRESHARK DHCP Races

```bash
RACE1> dhcp
DORA IP 10.1.1.15/24 GW 10.1.1.1

RACE1> dhcp
DORA IP 10.1.1.211/24 GW 10.1.1.1

RACE1>
```
