# TP2 : Ethernet, IP, et ARP

# I. Setup IP

Le lab, il vous faut deux machine : 

- les deux machines doivent être connectées physiquement
- vous devez choisir vous-mêmes les IPs à attribuer sur les interfaces réseau, les contraintes :
  - IPs privées (évidemment n_n)
  - dans un réseau qui peut contenir au moins 38 adresses IP (il faut donc choisir un masque adapté)
  - oui c'est random, on s'exerce c'est tout, p'tit jog en se levant
  - le masque choisi doit être le plus grand possible (le plus proche de 32 possible) afin que le réseau soit le plus petit possible

🌞 **Mettez en place une configuration réseau fonctionnelle entre les deux machines**

🐈‍⬛
```
pierre in ~/Ynov/TP-Leo/tp-r2 on main λ networksetup -setmanual AX88179A 10.42.0.2 255.255.255.192 1
0.42.0.1

pierre in ~/Ynov/TP-Leo/tp-r2 on main λ ifconfig
en7: flags=8863<UP,BROADCAST,SMART,RUNNING,SIMPLEX,MULTICAST> mtu 1500
    options=400<CHANNEL_IO>
    ether a0:ce:c8:ee:d4:14
    inet6 fe80::14a1:9159:dc3c:fbe%en7 prefixlen 64 secured scopeid 0x17
    inet 10.42.0.2 netmask 0xffffffc0 broadcast 10.42.0.63
    nd6 options=201<PERFORMNUD,DAD>
    media: autoselect (1000baseT <full-duplex>)
    status: active

$ ip a 
  [...]
  inet 10.42.0.1/26 brd 10.42.0.63
  [...]
```

- vous renseignerez dans le compte rendu :
  - les deux IPs choisies, en précisant le masque
  - l'adresse de réseau
  - l'adresse de broadcast
- vous renseignerez aussi les commandes utilisées pour définir les adresses IP *via* la ligne de commande

🐈‍⬛ **On a choisit les IP 10.42.0.1 et .2, /26 en masque pour avoir 64 - 2 = 62 adresses disponible.**

l'adresse mac du routeur d'ynov: 00:c0:e7:e0:4:4e
```
pierre in ~/Ynov/TP-Leo/tp-r2 on main λ arp -a -i en0 | grep 254
? (10.33.19.254) at 0:c0:e7:e0:4:4e on en0 ifscope [ethernet]
```

> Rappel : tout doit être fait *via* la ligne de commandes. Faites-vous du bien, et utilisez Powershell plutôt que l'antique cmd sous Windows svp.

🌞 **Prouvez que la connexion est fonctionnelle entre les deux machines**

- un `ping` suffit !

🐈‍⬛
```
pierre in ~/Ynov/TP-Leo/tp-r2 on main λ ping 10.42.0.1
PING 10.42.0.1 (10.42.0.1): 56 data bytes
64 bytes from 10.42.0.1: icmp_seq=0 ttl=64 time=0.720 ms
64 bytes from 10.42.0.1: icmp_seq=1 ttl=64 time=0.794 ms
```

🌞 **Wireshark it**

- `ping` ça envoie des paquets de type ICMP (c'est pas de l'IP, c'est un de ses frères)
  - les paquets ICMP sont encapsulés dans des trames Ethernet, comme les paquets IP
  - il existe plusieurs types de paquets ICMP, qui servent à faire des trucs différents
- **déterminez, grâce à Wireshark, quel type de paquet ICMP est envoyé par `ping`**
  - pour le ping que vous envoyez
  - et le pong que vous recevez en retour

🐈‍⬛ **le type des pings est "8 (echo (ping) request)" d'après la tram wireshark**

> Vous trouverez sur [la page Wikipedia de ICMP](https://en.wikipedia.org/wiki/Internet_Control_Message_Protocol) un tableau qui répertorie tous les types ICMP et leur utilité

🦈 **PCAP qui contient les paquets ICMP qui vous ont permis d'identifier les types ICMP**

[ping type pcap file](./ping-type.pcapng)

# II. ARP my bro

ARP permet, pour rappel, de résoudre la situation suivante :

- pour communiquer avec quelqu'un dans un LAN, il **FAUT** connaître son adresse MAC
- on admet un PC1 et un PC2 dans le même LAN :
  - PC1 veut joindre PC2
  - PC1 et PC2 ont une IP correctement définie
  - PC1 a besoin de connaître la MAC de PC2 pour lui envoyer des messages
  - **dans cette situation, PC1 va utilise le protocole ARP pour connaître la MAC de PC2**
  - une fois que PC1 connaît la mac de PC2, il l'enregistre dans sa **table ARP**

🌞 **Check the ARP table**

🐈‍⬛
```
pierre in ~/Ynov/TP-Leo/tp-r2 on main λ arp -a | grep en7
? (10.42.0.1) at 8:97:98:d4:fb:50 on en7 ifscope [ethernet]
? (239.255.255.250) at 1:0:5e:7f:ff:fa on en7 ifscope permanent [ethernet]
```

Sa MAC est: **8:97:98:d4:fb:50** (il manque le 0 au début mais j'imagine que macos ne l'affiche pas si c'est un 0 au début),
puisqu'il est la gateway de notre réseau, c'est aussi la mac de la gateway.

- utilisez une commande pour afficher votre table ARP
- déterminez la MAC de votre binome depuis votre table ARP
- déterminez la MAC de la *gateway* de votre réseau

> Il peut être utile de ré-effectuer des `ping` avant d'afficher la table ARP. En effet : les infos stockées dans la table ARP ne sont stockées que temporairement. Ce laps de temps est de l'ordre de ~60 secondes sur la plupart de nos machines.

🌞 **Manipuler la table ARP**

🐈‍⬛
```
pierre in ~/Ynov/TP-Leo/tp-r2 on main ● λ man arp
pierre in ~/Ynov/TP-Leo/tp-r2 on main ● λ arp -d -i en7 -a
arp: writing to routing socket: Operation not permitted
arp: writing to routing socket: Operation not permitted
pierre in ~/Ynov/TP-Leo/tp-r2 on main ● λ sudo !!
pierre in ~/Ynov/TP-Leo/tp-r2 on main ● λ sudo arp -d -i en7 -a
Password:
10.42.0.1 (10.42.0.1) deleted
239.255.255.250 (239.255.255.250) deleted
pierre in ~/Ynov/TP-Leo/tp-r2 on main ● λ arp -a -i en7
pierre in ~/Ynov/TP-Leo/tp-r2 on main ● λ ping 10.42.0.1
PING 10.42.0.1 (10.42.0.1): 56 data bytes
64 bytes from 10.42.0.1: icmp_seq=0 ttl=64 time=0.882 ms
64 bytes from 10.42.0.1: icmp_seq=1 ttl=64 time=0.670 ms
^C
--- 10.42.0.1 ping statistics ---
2 packets transmitted, 2 packets received, 0.0% packet loss
round-trip min/avg/max/stddev = 0.670/0.776/0.882/0.106 ms
pierre in ~/Ynov/TP-Leo/tp-r2 on main ● λ arp -a -i en7
? (10.42.0.1) at 8:97:98:d4:fb:50 on en7 ifscope [ethernet]
```

On voit ici qu'on a pu vider la table arp de la carte réseau en7, et ensuite qu'on a pu la re remplire grace a un ping

- utilisez une commande pour vider votre table ARP
- prouvez que ça fonctionne en l'affichant et en constatant les changements
- ré-effectuez des pings, et constatez la ré-apparition des données dans la table ARP

> Les échanges ARP sont effectuées automatiquement par votre machine lorsqu'elle essaie de joindre une machine sur le même LAN qu'elle. Si la MAC du destinataire n'est pas déjà dans la table ARP, alors un échange ARP sera déclenché.

🌞 **Wireshark it**

- vous savez maintenant comment forcer un échange ARP : il sufit de vider la table ARP et tenter de contacter quelqu'un, l'échange ARP se fait automatiquement
- mettez en évidence les deux trames ARP échangées lorsque vous essayez de contacter quelqu'un pour la "première" fois
  - déterminez, pour les deux trames, les adresses source et destination
  - déterminez à quoi correspond chacune de ces adresses

🐈‍⬛ **Mon packet envoyé contient:**
- l'adresse source est: a0:ce:c8:ee:d4:14 (mon adapateur)
- l'adresse de destination est: ff:ff:ff:ff:ff:ff (broadcast)

**Le packet reçu contient:**
- source: 08:97:98:d4:fb:50
- destination: a0:ce:c8:ee:d4:14

🦈 **PCAP qui contient les trames ARP**

[pcap échange arp](./arp_via_ping.pcapng)
> L'échange ARP est constitué de deux trames : un ARP broadcast et un ARP reply.

# II.5 Interlude hackerzz

**Chose promise chose due, on va voir les bases de l'usurpation d'identité en réseau : on va parler d'*ARP poisoning*.**

> On peut aussi trouver *ARP cache poisoning* ou encore *ARP spoofing*, ça désigne la même chose.

Le principe est simple : on va "empoisonner" la table ARP de quelqu'un d'autre.  
Plus concrètement, on va essayer d'introduire des fausses informations dans la table ARP de quelqu'un d'autre.

Entre introduire des fausses infos et usurper l'identité de quelqu'un il n'y a qu'un pas hihi.

---

Le principe de l'attaque :

- on admet Alice, Bob et Eve, tous dans un LAN, chacun leur PC
- leur configuration IP est ok, tout va bien dans le meilleur des mondes
- **Eve 'lé pa jonti** *(ou juste un agent de la CIA)* : elle aimerait s'immiscer dans les conversations de Alice et Bob
  - pour ce faire, Eve va empoisonner la table ARP de Bob, pour se faire passer pour Alice
  - elle va aussi empoisonner la table ARP d'Alice, pour se faire passer pour Bob
  - ainsi, tous les messages que s'envoient Alice et Bob seront en réalité envoyés à Eve

La place de ARP dans tout ça :

- ARP est un principe de question -> réponse (broadcast -> *reply*)
- IL SE TROUVE qu'on peut envoyer des *reply* à quelqu'un qui n'a rien demandé :)
- il faut donc simplement envoyer :
  - une trame ARP reply à Alice qui dit "l'IP de Bob se trouve à la MAC de Eve" (IP B -> MAC E)
  - une trame ARP reply à Bob qui dit "l'IP de Alice se trouve à la MAC de Eve" (IP A -> MAC E)
- ha ouais, et pour être sûr que ça reste en place, il faut SPAM sa mum, genre 1 reply chacun toutes les secondes ou truc du genre
  - bah ui ! Sinon on risque que la table ARP d'Alice ou Bob se vide naturellement, et que l'échange ARP normal survienne
  - aussi, c'est un truc possible, mais pas normal dans cette utilisation là, donc des fois bon, ça chie, DONC ON SPAM

![Am I ?](./pics/arp_snif.jpg)

---

J'peux vous aider à le mettre en place, mais **vous le faites uniquement dans un cadre privé, chez vous, ou avec des VMs**

**Je vous conseille 3 machines Linux**, Alice Bob et Eve. La commande `[arping](https://sandilands.info/sgordon/arp-spoofing-on-wired-lan)` pourra vous carry : elle permet d'envoyer manuellement des trames ARP avec le contenu de votre choix.

GLHF.

# III. DHCP you too my brooo

![YOU GET AN IP](./pics/dhcp.jpg)

*DHCP* pour *Dynamic Host Configuration Protocol* est notre p'tit pote qui nous file des IP quand on arrive dans un réseau, parce que c'est chiant de le faire à la main :)

Quand on arrive dans un réseau, notre PC contacte un serveur DHCP, et récupère généralement 3 infos :

- **1.** une IP à utiliser
- **2.** l'adresse IP de la passerelle du réseau
- **3.** l'adresse d'un serveur DNS joignable depuis ce réseau

L'échange DHCP consiste en 4 trames : DORA, que je vous laisse google vous-mêmes : D

🌞 **Wireshark it**

- identifiez les 4 trames DHCP lors d'un échange DHCP
  - mettez en évidence les adresses source et destination de chaque trame
- identifiez dans ces 4 trames les informations **1**, **2** et **3** dont on a parlé juste au dessus

🐈‍⬛ 
- 1:
    - Source: Apple_67:c3:19 (f8:4d:89:67:c3:19)
    - Destination: Broadcast (ff:ff:ff:ff:ff:ff)
- 2:
    - Source: Fiberdat_e0:04:4e (00:c0:e7:e0:04:4e)
    - Destination: Apple_67:c3:19 (f8:4d:89:67:c3:19)
- 3:
    - Source: Apple_67:c3:19 (f8:4d:89:67:c3:19)
    - Destination: Broadcast (ff:ff:ff:ff:ff:ff)
- 4:
    - Source: Fiberdat_e0:04:4e (00:c0:e7:e0:04:4e)
    - Destination: Apple_67:c3:19 (f8:4d:89:67:c3:19)
- Info a identifier:
    - Your (client) IP address: 10.33.19.92
    - Router: 10.33.19.254
    - Domain Name Server: 8.8.8.8, Domain Name Server: 8.8.4.4, Domain Name Server: 1.1.1.1

🦈 **PCAP qui contient l'échange DORA**

[pcap dhcp request file](./dhcp_request.pcapng)

> **Soucis** : l'échange DHCP ne se produit qu'à la première connexion. **Pour forcer un échange DHCP**, ça dépend de votre OS. Sur **GNU/Linux**, avec `dhclient` ça se fait bien. Sur **Windows**, le plus simple reste de définir une IP statique pourrie sur la carte réseau, se déconnecter du réseau, remettre en DHCP, se reconnecter au réseau. Sur **MacOS**, je connais peu mais Internet dit qu'c'est po si compliqué, appelez moi si besoin.

# IV. Avant-goût TCP et UDP

TCP et UDP ce sont les deux protocoles qui utilisent des ports. Si on veut accéder à un service, sur un serveur, comme un site web :

- il faut pouvoir joindre en terme d'IP le correspondant
  - on teste que ça fonctionne avec un `ping` généralement
- il faut que le serveur fasse tourner un programme qu'on appelle "service" ou "serveur"
  - le service "écoute" sur un port TCP ou UDP : il attend la connexion d'un client
- le client **connaît par avance** le port TCP ou UDP sur lequel le service écoute
- en utilisant l'IP et le port, il peut se connecter au service en utilisant un moyen adapté :
  - un navigateur web pour un site web
  - un `ncat` pour se connecter à un autre `ncat`
  - et plein d'autres, **de façon générale on parle d'un client, et d'un serveur**

---

🌞 **Wireshark it**

- déterminez à quelle IP et quel port votre PC se connecte quand vous regardez une vidéo Youtube
  - il sera sûrement plus simple de repérer le trafic Youtube en fermant tous les autres onglets et toutes les autres applications utilisant du réseau

🐈‍⬛ 
- Destination Address: 216.58.215.46 
- Destination Port: 443

🦈 **PCAP qui contient un extrait de l'échange qui vous a permis d'identifier les infos**

[pcap du bordel de youtube](./bordel_youtube.pcapng)