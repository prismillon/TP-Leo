# Sujet Réseau et Infra

On va utiliser GNS3 dans ce TP pour se rapprocher d'un cas réel. On va focus sur l'aspect routing/switching, avec du matériel Cisco. On va aussi mettre en place des VLANs.

![best memes from cisco doc](./pics/the-best-memes-come-from-cisco-documentation.jpg)

# Sommaire

- [Sujet Réseau et Infra](#sujet-réseau-et-infra)
- [Sommaire](#sommaire)
- [0. Prérequis](#0-prérequis)
  - [Checklist VM Linux](#checklist-vm-linux)
- [I. Dumb switch](#i-dumb-switch)
  - [1. Topologie 1](#1-topologie-1)
  - [2. Adressage topologie 1](#2-adressage-topologie-1)
  - [3. Setup topologie 1](#3-setup-topologie-1)
- [II. VLAN](#ii-vlan)
  - [1. Topologie 2](#1-topologie-2)
  - [2. Adressage topologie 2](#2-adressage-topologie-2)
    - [3. Setup topologie 2](#3-setup-topologie-2)
- [III. Routing](#iii-routing)
  - [1. Topologie 3](#1-topologie-3)
  - [2. Adressage topologie 3](#2-adressage-topologie-3)
  - [3. Setup topologie 3](#3-setup-topologie-3)
- [IV. NAT](#iv-nat)
  - [1. Topologie 4](#1-topologie-4)
  - [2. Adressage topologie 4](#2-adressage-topologie-4)
  - [3. Setup topologie 4](#3-setup-topologie-4)
- [V. Add a building](#v-add-a-building)
  - [1. Topologie 5](#1-topologie-5)
  - [2. Adressage topologie 5](#2-adressage-topologie-5)
  - [3. Setup topologie 5](#3-setup-topologie-5)

# 0. Prérequis

➜ GNS3 installé et prêt à l'emploi

- [GNS3VM](https://www.gns3.com/software/download-vm) fonctionnelle
- de quoi faire tourner un switch Cisco
  - [IOU2 L2 dispo ici](http://dl.nextadmin.net/dl/EVE-NG-image/iol/bin/i86bi_linux_l2-adventerprisek9-ms.SSA.high_iron_20180510.bin)
- de quoi faire tourner un routeur Cisco
  - [image d'un 3745 dispo ici](http://dl.nextadmin.net/dl/EVE-NG-image/dynamips/c3725-adventerprisek9-mz.124-15.T14.image)

➜ Les clients seront soit :

- VMs Rocky Linux
- VPCS
  - c'est un truc de GNS pour simuler un client du réseau
  - quand on veut juste un truc capable de faire des pings et rien de plus, c'est parfait
  - ça consomme R en ressources

> Faites bien attention aux logos des machines sur les schémas, et vous verrez clairement quand il faut un VPCS ou une VM.

➜ **Vous ne créerez aucune machine virtuelle au début. Vous les créerez au fur et à mesure que le TP vous le demande.** A chaque fois qu'une nouvelle machine devra être créée, vous trouverez l'emoji 🖥️ avec son nom.

## Checklist VM Linux

A chaque machine déployée, vous **DEVREZ** vérifier la 📝**checklist**📝 :

- [x] IP locale, statique ou dynamique
- [x] hostname défini
- [x] firewall actif, qui ne laisse passer que le strict nécessaire
- [x] SSH fonctionnel
- [x] résolution de nom
  - vers internet, quand vous aurez le routeur en place

**Les éléments de la 📝checklist📝 sont STRICTEMENT OBLIGATOIRES à réaliser mais ne doivent PAS figurer dans le rendu.**

# I. Dumb switch

## 1. Topologie 1

![Topologie 1](./pics/topo1.png)

## 2. Adressage topologie 1

| Node  | IP            |
|-------|---------------|
| `pc1` | `10.5.1.1/24` |
| `pc2` | `10.5.1.2/24` |

## 3. Setup topologie 1

🌞 **Commençons simple**

- définissez les IPs statiques sur les deux VPCS
- `ping` un VPCS depuis l'autre

```
pc1> ping 10.5.10.2

84 bytes from 10.5.10.2 icmp_seq=1 ttl=64 time=0.852 ms
84 bytes from 10.5.10.2 icmp_seq=2 ttl=64 time=0.770 ms
84 bytes from 10.5.10.2 icmp_seq=3 ttl=64 time=0.586 ms
84 bytes from 10.5.10.2 icmp_seq=4 ttl=64 time=0.601 ms
```

> Jusque là, ça devrait aller. Noter qu'on a fait aucune conf sur le switch. Tant qu'on ne fait rien, c'est une bête multiprise.

# II. VLAN

**Le but dans cette partie va être de tester un peu les *VLANs*.**

On va rajouter **un troisième client** qui, bien que dans le même réseau, sera **isolé des autres grâce aux *VLANs***.

**Les *VLANs* sont une configuration à effectuer sur les *switches*.** C'est les *switches* qui effectuent le blocage.

Le principe est simple :

- déclaration du VLAN sur tous les switches
  - un VLAN a forcément un ID (un entier)
  - bonne pratique, on lui met un nom
- sur chaque switch, on définit le VLAN associé à chaque port
  - genre "sur le port 35, c'est un client du VLAN 20 qui est branché"

![VLAN FOR EVERYONE](./pics/get_a_vlan.jpg)

## 1. Topologie 2

![Topologie 2](./pics/topo2.png)

## 2. Adressage topologie 2

| Node  | IP             | VLAN |
|-------|----------------|------|
| `pc1` | `10.5.10.1/24` | 10   |
| `pc2` | `10.5.10.2/24` | 10   |
| `pc3` | `10.5.10.3/24` | 20   |

### 3. Setup topologie 2

🌞 **Adressage**

- définissez les IPs statiques sur tous les VPCS
- vérifiez avec des `ping` que tout le monde se ping

```
pc1> ip 10.5.10.1/24 10.5.10.254
pc2> ip 10.5.10.2/24 10.5.10.254
pc3> ip 10.5.10.3/24 10.5.10.254
```
```
pc3> ping 10.5.10.1

84 bytes from 10.5.10.1 icmp_seq=1 ttl=64 time=0.427 ms
84 bytes from 10.5.10.1 icmp_seq=2 ttl=64 time=1.133 ms
84 bytes from 10.5.10.1 icmp_seq=3 ttl=64 time=0.677 ms
84 bytes from 10.5.10.1 icmp_seq=4 ttl=64 time=0.761 ms

pc3> ping 10.5.10.2

84 bytes from 10.5.10.2 icmp_seq=1 ttl=64 time=1.133 ms
84 bytes from 10.5.10.2 icmp_seq=2 ttl=64 time=0.135 ms
84 bytes from 10.5.10.2 icmp_seq=3 ttl=64 time=0.175 ms
84 bytes from 10.5.10.2 icmp_seq=4 ttl=64 time=0.677 ms
```

🌞 **Configuration des VLANs**

- référez-vous [à la section VLAN du mémo Cisco](../../cours/memo/memo_cisco.md#8-vlan)
- déclaration des VLANs sur le switch `sw1`
- ajout des ports du switches dans le bon VLAN (voir [le tableau d'adressage de la topo 2 juste au dessus](#2-adressage-topologie-2))
  - ici, tous les ports sont en mode *access* : ils pointent vers des clients du réseau

```
conf t
vlan 10
name clients
exit
vlan 20
name admins
exit
vlan 30
name servers
exit
```

```
interface ethernet0/0
sw ac vlan 10
exit
interface ethernet0/1
sw ac vlan 10
exit
interface ethernet0/2
sw ac vlan 20
exit
```

🌞 **Vérif**

- `pc1` et `pc2` doivent toujours pouvoir se ping
- `pc3` ne ping plus personne

```
pc1> ping 10.5.10.2

84 bytes from 10.5.10.2 icmp_seq=1 ttl=64 time=0.347 ms
84 bytes from 10.5.10.2 icmp_seq=2 ttl=64 time=1.038 ms
84 bytes from 10.5.10.2 icmp_seq=3 ttl=64 time=0.598 ms
84 bytes from 10.5.10.2 icmp_seq=4 ttl=64 time=0.557 ms
```

```
pc3> ping 10.5.10.1

host (10.5.10.1) not reachable

pc3> ping 10.5.10.2

host (10.5.10.2) not reachable
```

# III. Routing

Dans cette partie, on va donner un peu de sens aux VLANs :

- un pour les serveurs du réseau
  - on simulera ça avec un p'tit serveur web
- un pour les admins du réseau
- un pour les autres random clients du réseau

Cela dit, il faut que tout ce beau monde puisse se ping, au moins joindre le réseau des serveurs, pour accéder au super site-web.

**Bien que bloqué au niveau du switch à cause des VLANs, le trafic pourra passer d'un VLAN à l'autre grâce à un routeur.**

Il assurera son job de routeur traditionnel : router entre deux réseaux. Sauf qu'en plus, il gérera le changement de VLAN à la volée.

## 1. Topologie 3

![Topologie 3](./pics/topo3.png)

## 2. Adressage topologie 3

Les réseaux et leurs VLANs associés :

| Réseau    | Adresse        | VLAN associé |
|-----------|----------------|--------------|
| `clients` | `10.5.10.0/24` | 10           |
| `admins`  | `10.5.20.0/24` | 20           |
| `servers` | `10.5.30.0/24` | 30           |

> **Question de bonne pratique** : on fait apparaître le numéro du VLAN dans l'adresse du réseau concerné. En effet, souvent, à un VLAN donné est associé un réseau donné. Par exemple le VLAN **20** correspond au réseau 10.5.**20**.0/24.

L'adresse des machines au sein de ces réseaux :

| Node               | `clients`        | `admins`         | `servers`        |
|--------------------|------------------|------------------|------------------|
| `pc1.clients.tp5`  | `10.5.10.1/24`   | x                | x                |
| `pc2.clients.tp5`  | `10.5.10.2/24`   | x                | x                |
| `adm1.admins.tp5`  | x                | `10.5.20.1/24`   | x                |
| `web1.servers.tp5` | x                | x                | `10.5.30.1/24`   |
| `r1`               | `10.5.10.254/24` | `10.5.20.254/24` | `10.5.30.254/24` |

## 3. Setup topologie 3

🖥️ VM `web1.servers.tp5`, déroulez la [Checklist VM Linux](#checklist-vm-linux) dessus

🌞 **Adressage**

- définissez les IPs statiques sur toutes les machines **sauf le *routeur***

```
adm1>ip 10.5.20.1/24 10.5.20.254

vim /etc/sysconfig/network-scripts/ifcfg-enp0s3
DEVICE=enp0s3

BOOTPROTO=static
ONBOOT=yes

IPADDR=10.5.30.1
NETMASK=255.255.255.0

GATEWAY=10.5.30.254
DNS1=1.1.1.1
```

🌞 **Configuration des VLANs**

- référez-vous au [mémo Cisco](../../cours/memo/memo_cisco.md#8-vlan)
- déclaration des VLANs sur le switch `sw1`
- ajout des ports du switches dans le bon VLAN (voir [le tableau d'adressage de la topo 2 juste au dessus](#2-adressage-topologie-2))
- il faudra ajouter le port qui pointe vers le *routeur* comme un *trunk* : c'est un port entre deux équipements réseau (un *switch* et un *routeur*)

```
interface ethernet0/2
sw ac vlan 20
exit
interface ethernet0/3
sw ac vlan 30
exit

interface ethernet3/3
sw trunk encapsulation dot1Q
sw mode trunk
sw trunk allowed vlan 10,20,30
no shutdown
exit
```

---

➜ **Pour le *routeur***

- référez-vous au [mémo Cisco](../../cours/memo/memo_cisco.md)
- ici, on va avoir besoin d'un truc très courant pour un *routeur* : qu'il porte plusieurs IP sur une unique interface
  - avec Cisco, on crée des "sous-interfaces" sur une interface
  - et on attribue une IP à chacune de ces sous-interfaces
- en plus de ça, il faudra l'informer que, pour chaque interface, elle doit être dans un VLAN spécifique

Pour ce faire, un exemple. On attribue deux IPs `192.168.1.254/24` VLAN 10 et `192.168.2.254` VLAN 20 à un *routeur*. L'interface concernée sur le *routeur* est `fastEthernet 0/0` :

```cisco
# conf t

(config)# interface fastEthernet 0/0.10
R1(config-subif)# encapsulation dot1Q 10
R1(config-subif)# ip addr 192.168.1.254 255.255.255.0 
R1(config-subif)# exit

(config)# interface fastEthernet 0/0.20
R1(config-subif)# encapsulation dot1Q 20
R1(config-subif)# ip addr 192.168.2.254 255.255.255.0 
R1(config-subif)# exit
```

🌞 **Config du *routeur***

- attribuez ses IPs au *routeur*
  - 3 sous-interfaces, chacune avec son IP et un VLAN associé

```
interface FastEthernet0/0.10
 encapsulation dot1Q 10
 ip address 10.5.10.254 255.255.255.0
!
interface FastEthernet0/0.20
 encapsulation dot1Q 20
 ip address 10.5.20.254 255.255.255.0
!
interface FastEthernet0/0.30
 encapsulation dot1Q 30
 ip address 10.5.30.254 255.255.255.0
!
```

🌞 **Vérif**

- tout le monde doit pouvoir ping le routeur sur l'IP qui est dans son réseau
- en ajoutant une route vers les réseaux, ils peuvent se ping entre eux
  - ajoutez une route par défaut sur les VPCS
  - ajoutez une route par défaut sur la machine virtuelle
  - testez des `ping` entre les réseaux

```
pc1> sh ip

NAME        : pc1[1]
IP/MASK     : 10.5.10.1/24
GATEWAY     : 10.5.10.254
DNS         : 8.8.8.8

pc2> sh ip

NAME        : pc2[1]
IP/MASK     : 10.5.10.2/24
GATEWAY     : 10.5.10.254
DNS         : 8.8.8.8

adm1> sh ip

NAME        : adm1[1]
IP/MASK     : 10.5.20.1/24
GATEWAY     : 10.5.20.254
DNS         : 8.8.8.8

[root@web ~]# ping 10.5.10.1

84 bytes from 10.5.10.1 icmp_seq=1 ttl=64 time=22.0 ms
84 bytes from 10.5.10.1 icmp_seq=2 ttl=64 time=15.6 ms
84 bytes from 10.5.10.1 icmp_seq=3 ttl=64 time=16.4 ms
84 bytes from 10.5.10.1 icmp_seq=4 ttl=64 time=25.2 ms
```

# IV. NAT

On va ajouter une fonctionnalité au routeur : le NAT.

On va le connecter à internet (simulation du fait d'avoir une IP publique) et il va faire du NAT pour permettre à toutes les machines du réseau d'avoir un accès internet.

![Yellow cable](./pics/yellow-cable.png)

## 1. Topologie 4

![Topologie 3](./pics/topo4.png)

## 2. Adressage topologie 4

Les réseaux et leurs VLANs associés :

| Réseau    | Adresse        | VLAN associé |
|-----------|----------------|--------------|
| `clients` | `10.5.10.0/24` | 10           |
| `admins`  | `10.5.20.0/24` | 20           |
| `servers` | `10.5.30.0/24` | 30           |

L'adresse des machines au sein de ces réseaux :

| Node               | `clients`        | `admins`         | `servers`        |
|--------------------|------------------|------------------|------------------|
| `pc1.clients.tp5`  | `10.5.10.1/24`   | x                | x                |
| `pc2.clients.tp5`  | `10.5.10.2/24`   | x                | x                |
| `adm1.admins.tp5`  | x                | `10.5.20.1/24`   | x                |
| `web1.servers.tp5` | x                | x                | `10.5.30.1/24`   |
| `r1`               | `10.5.10.254/24` | `10.5.20.254/24` | `10.5.30.254/24` |

## 3. Setup topologie 4

🌞 **Ajoutez le noeud Cloud à la topo**

- branchez à `eth1` côté Cloud
- côté routeur, il faudra récupérer un IP en DHCP (voir [le mémo Cisco](../../cours/memo/memo_cisco.md))
- vous devriez pouvoir `ping 1.1.1.1`

```
interface fastEthernet0/1
ip dhcp
no shutdown
exit

R1#ping 1.1.1.1

Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 1.1.1.1, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 16/28/36 ms
```

🌞 **Configurez le NAT**

- référez-vous [à la section NAT du mémo Cisco](../../cours/memo/memo_cisco.md#7-configuration-dun-nat-simple)

```
interface FastEthernet0/0.10
 encapsulation dot1Q 10
 ip address 10.5.10.254 255.255.255.0
 ip nat inside
 ip virtual-reassembly
!
interface FastEthernet0/0.20
 encapsulation dot1Q 20
 ip address 10.5.20.254 255.255.255.0
 ip nat inside
 ip virtual-reassembly
!
interface FastEthernet0/0.30
 encapsulation dot1Q 30
 ip address 10.5.30.254 255.255.255.0
 ip nat inside
 ip virtual-reassembly
!
interface FastEthernet0/1
 ip address dhcp
 ip nat outside
 ip virtual-reassembly
 duplex auto
 speed auto
```

🌞 **Test**

- ajoutez une route par défaut (si c'est pas déjà fait)
  - sur les VPCS
  - sur la machine Linux
- configurez l'utilisation d'un DNS
  - sur les VPCS
  - sur la machine Linux
- vérifiez un `ping` vers un nom de domaine

```
ip dns 1.1.1.1
pc1> ping google.com
google.com resolved to 216.58.198.206

84 bytes from 216.58.198.206 icmp_seq=1 ttl=127 time=41.070 ms
84 bytes from 216.58.198.206 icmp_seq=2 ttl=127 time=35.204 ms
84 bytes from 216.58.198.206 icmp_seq=3 ttl=127 time=35.383 ms
84 bytes from 216.58.198.206 icmp_seq=4 ttl=127 time=38.443 ms
```


# V. Add a building

On a acheté un nouveau bâtiment, faut tirer et configurer un nouveau switch jusque là-bas.

On va en profiter pour setup un serveur DHCP pour les clients qui s'y trouvent.

## 1. Topologie 5

![Topo 5](./pics/topo5.png)

## 2. Adressage topologie 5

Les réseaux et leurs VLANs associés :

| Réseau    | Adresse        | VLAN associé |
|-----------|----------------|--------------|
| `clients` | `10.5.10.0/24` | 10           |
| `admins`  | `10.5.20.0/24` | 20           |
| `servers` | `10.5.30.0/24` | 30           |

L'adresse des machines au sein de ces réseaux :

| Node                | `clients`        | `admins`         | `servers`        |
|---------------------|------------------|------------------|------------------|
| `pc1.clients.tp5`   | `10.5.10.1/24`   | x                | x                |
| `pc2.clients.tp5`   | `10.5.10.2/24`   | x                | x                |
| `pc3.clients.tp5`   | DHCP             | x                | x                |
| `pc4.clients.tp5`   | DHCP             | x                | x                |
| `pc5.clients.tp5`   | DHCP             | x                | x                |
| `dhcp1.clients.tp5` | `10.5.10.253/24` | x                | x                |
| `adm1.admins.tp5`   | x                | `10.5.20.1/24`   | x                |
| `web1.servers.tp5`  | x                | x                | `10.5.30.1/24`   |
| `r1`                | `10.5.10.254/24` | `10.5.20.254/24` | `10.5.30.254/24` |

## 3. Setup topologie 5

Vous pouvez partir de la topologie 4. 

🌞  **Vous devez me rendre le `show running-config` de tous les équipements**

- de tous les équipements réseau
  - le routeur
  - les 3 switches

```
sw1:
interface Ethernet3/1
 switchport trunk allowed vlan 10,20,30
 switchport trunk encapsulation dot1q
 switchport mode trunk
!
interface Ethernet3/2
 switchport trunk allowed vlan 10,20,30
 switchport trunk encapsulation dot1q
 switchport mode trunk
!
interface Ethernet3/3
 switchport trunk allowed vlan 10,20,30
 switchport trunk encapsulation dot1q
 switchport mode trunk
!

sw2:
interface Ethernet0/0
 switchport access vlan 10
!
interface Ethernet0/1
 switchport access vlan 10
!
interface Ethernet0/2
 switchport access vlan 20
!
interface Ethernet0/3
 switchport access vlan 30
!
interface Ethernet3/3
 switchport trunk allowed vlan 10,20,30
 switchport trunk encapsulation dot1q
 switchport mode trunk
!

sw3:
interface Ethernet0/0
 switchport access vlan 10
!
interface Ethernet0/1
 switchport access vlan 10
!
interface Ethernet0/2
 switchport access vlan 10
!
interface Ethernet0/3
 switchport access vlan 10
!
interface Ethernet3/3
 switchport trunk allowed vlan 10
 switchport trunk encapsulation dot1q
 switchport mode trunk
!

r1:
interface FastEthernet0/0
 no ip address
 duplex auto
 speed auto
!
interface FastEthernet0/0.10
 encapsulation dot1Q 10
 ip address 10.5.10.254 255.255.255.0
 ip nat inside
 ip virtual-reassembly
!
interface FastEthernet0/0.20
 encapsulation dot1Q 20
 ip address 10.5.20.254 255.255.255.0
 ip nat inside
 ip virtual-reassembly
!
interface FastEthernet0/0.30
 encapsulation dot1Q 30
 ip address 10.5.30.254 255.255.255.0
 ip nat inside
 ip virtual-reassembly
!
interface FastEthernet0/1
 ip address dhcp
 ip nat outside
 ip virtual-reassembly
 duplex auto
 speed auto
!
```

> N'oubliez pas les VLANs sur tous les switches.

🖥️ **VM `dhcp1.client1.tp5`**, déroulez la [Checklist VM Linux](#checklist-vm-linux) dessus

🌞  **Mettre en place un serveur DHCP dans le nouveau bâtiment**

- il doit distribuer des IPs aux clients dans le réseau `clients` qui sont branchés au même switch que lui
- sans aucune action manuelle, les clients doivent...
  - avoir une IP dans le réseau `clients`
  - avoir un accès au réseau `servers`
  - avoir un accès WAN
  - avoir de la résolution DNS

```
[root@dhcp1 ~]# cat /etc/dhcp/dhcpd.conf

  default-lease-time 900;
  max-lease-time 10800;
  ddns-update-style none;
  authoritative;
  subnet 10.5.10.0 netmask 255.255.255.0 {
    range 10.5.10.3 10.5.10.252;
    option routers 10.5.10.254;
    option subnet-mask 255.255.255.0;
    option domain-name-servers 1.1.1.1;

  }
```

> Réutiliser les serveurs DHCP qu'on a monté dans les autres TPs.

🌞  **Vérification**

- un client récupère une IP en DHCP
- il peut ping le serveur Web
- il peut ping `8.8.8.8`
- il peut ping `google.com`

```
pc4> ip dhcp
DORA IP 10.5.10.4/24 GW 10.5.10.254

pc4> ping 10.5.30.1

84 bytes from 10.5.30.1 icmp_seq=1 ttl=63 time=19.769 ms
84 bytes from 10.5.30.1 icmp_seq=2 ttl=63 time=15.713 ms
84 bytes from 10.5.30.1 icmp_seq=3 ttl=63 time=24.467 ms
84 bytes from 10.5.30.1 icmp_seq=4 ttl=63 time=18.041 ms

pc4> ping 8.8.8.8

84 bytes from 8.8.8.8 icmp_seq=1 ttl=127 time=51.602 ms
84 bytes from 8.8.8.8 icmp_seq=2 ttl=127 time=42.587 ms
84 bytes from 8.8.8.8 icmp_seq=3 ttl=127 time=33.321 ms
84 bytes from 8.8.8.8 icmp_seq=4 ttl=127 time=35.495 ms

pc4> ping google.com
google.com resolved to 142.250.178.142

84 bytes from 142.250.178.142 icmp_seq=1 ttl=127 time=39.714 ms
84 bytes from 142.250.178.142 icmp_seq=2 ttl=127 time=42.855 ms
84 bytes from 142.250.178.142 icmp_seq=3 ttl=127 time=35.271 ms
84 bytes from 142.250.178.142 icmp_seq=4 ttl=127 time=34.294 ms
```

> Faites ça sur n'importe quel VPCS que vous venez d'ajouter : `pc3` ou `pc4` ou `pc5`.

![i know cisco](./pics/i_know.jpeg)
