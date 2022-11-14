# I. Exploration locale en solo

## 1. Affichage d'informations sur la pile TCP/IP locale

### En ligne de commande

En utilisant la ligne de commande (CLI) de votre OS :

```
pierre in ~/Ynov/TP-Leo on main ● λ ifconfig
[...]
en0: flags=8863<UP,BROADCAST,SMART,RUNNING,SIMPLEX,MULTICAST> mtu 1500
	options=6463<RXCSUM,TXCSUM,TSO4,TSO6,CHANNEL_IO,PARTIAL_CSUM,ZEROINVERT_CSUM>
	ether f8:4d:89:67:c3:19
	inet6 fe80::1cb3:5ee2:115d:2dcb%en0 prefixlen 64 secured scopeid 0xd
	inet 10.33.19.92 netmask 0xfffffc00 broadcast 10.33.19.255
	nd6 options=201<PERFORMNUD,DAD>
	media: autoselect
	status: active
[...]
```

- utilisez une commande pour connaître l'adresse IP de la [passerelle](../../cours/lexique.md#passerelle-ou-gateway) de votre carte WiFi
```
pierre in ~/Ynov/TP-Leo on main ● λ netstat -nr
Routing tables

Internet:
Destination        Gateway            Flags           Netif Expire
default            10.33.19.254       UGScg             en0
10.33.16/22        link#13            UCS               en0      !
[...]
```

### En graphique (GUI : Graphical User Interface)

En utilisant l'interface graphique de votre OS :  

- trouvez l'IP, la MAC et la [gateway](../../cours/lexique.md#passerelle-ou-gateway) pour l'interface WiFi de votre PC

![](./Interface%20ip%20wifi.png)
![](./adresse%20mac.png)

### Questions

- 🌞 à quoi sert la [gateway](../../cours/lexique.md#passerelle-ou-gateway) dans le réseau d'YNOV ?

Acceder a internet cad d'autre LAN

## 2. Modifications des informations

### A. Modification d'adresse IP (part 1)  

🌞 Utilisez l'interface graphique de votre OS pour **changer d'adresse IP** :

![](./changement%20ip.png)

🌞 **Il est possible que vous perdiez l'accès internet.** Que ce soit le cas ou non, expliquez pourquoi c'est possible de perdre son accès internet en faisant cette opération.

Si quelqu'un possède déjà l'ip il récupèrera les informations a ma place et personne ne pourra me répondre.

### B. Table ARP

🌞 Exploration de la table ARP

```
pierre in ~/Ynov/TP-Leo on main ● λ netstat -nr
Routing tables

Internet:
Destination        Gateway            Flags           Netif Expire
default            10.33.19.254       UGScg             en0
10.33.16/22        link#13            UCS               en0      !
10.33.16.2         72:85:b9:a6:4e:d2  UHLWI             en0    861
10.33.16.8         f6:47:ac:9b:1:2    UHLWI             en0   1128
10.33.16.10        e8:f4:8:8c:cc:a6   UHLWI             en0   1070
10.33.16.12        c6:20:1c:b0:d:47   UHLWI             en0      !
10.33.16.15        82:35:a3:36:c4:26  UHLWI             en0    189
```

pour une raison qui m'échappe il donne pas l'adresse mac de la gateway 🤡


🌞 Et si on remplissait un peu la table ?

- envoyez des ping vers des IP du même réseau que vous. Lesquelles ? menfou, random. Envoyez des ping vers au moins 3-4 machines.
- affichez votre table ARP
- listez les adresses MAC associées aux adresses IP que vous avez ping

```
arp -a
[...]
? (10.33.19.32) at (incomplete) on en0 ifscope [ethernet]
? (10.33.19.67) at (incomplete) on en0 ifscope [ethernet]
? (10.33.19.87) at (incomplete) on en0 ifscope [ethernet]
[...]
```

# II. Exploration locale en duo

Owkay. Vous savez à ce stade :

- afficher les informations IP de votre machine
- modifier les informations IP de votre machine
- c'est un premier pas vers la maîtrise de votre outil de travail

On va maintenant répéter un peu ces opérations, mais en créant un réseau local de toutes pièces : entre deux PCs connectés avec un câble RJ45.

## 1. Prérequis

- deux PCs avec ports RJ45
- un câble RJ45
- **firewalls désactivés** sur les deux PCs

## 2. Câblage

Ok c'est la partie tendue. Prenez un câble. Branchez-le des deux côtés. **Bap.**

## Création du réseau (oupa)

Cette étape peut paraître cruciale. En réalité, elle n'existe pas à proprement parlé. On ne peut pas "créer" un réseau. Si une machine possède une carte réseau, et si cette carte réseau porte une adresse IP, alors cette adresse IP se trouve dans un réseau (l'adresse de réseau). Ainsi, le réseau existe. De fait.  

**Donc il suffit juste de définir une adresse IP sur une carte réseau pour que le réseau existe ! Bap.**

## 3. Modification d'adresse IP

🌞Si vos PCs ont un port RJ45 alors y'a une carte réseau Ethernet associée :

- modifiez l'IP des deux machines pour qu'elles soient dans le même réseau
  - choisissez une IP qui commence par "192.168"
  - utilisez un /30 (que deux IP disponibles)
- vérifiez à l'aide de commandes que vos changements ont pris effet
- utilisez `ping` pour tester la connectivité entre les deux machines
- affichez et consultez votre table ARP

```
pierre in ~/Ynov/TP-Leo on main ● λ ping 192.168.0.1
PING 192.168.0.1 (192.168.0.1): 56 data bytes
64 bytes from 192.168.0.1: icmp_seq=0 ttl=64 time=1.807 ms
64 bytes from 192.168.0.1: icmp_seq=1 ttl=64 time=1.534 ms
64 bytes from 192.168.0.1: icmp_seq=2 ttl=64 time=1.993 ms
64 bytes from 192.168.0.1: icmp_seq=3 ttl=64 time=2.033 ms
64 bytes from 192.168.0.1: icmp_seq=4 ttl=64 time=0.575 ms

pierre in ~/Ynov/TP-Leo on main ● λ arp -a
? (192.168.0.1) at 8:97:98:d4:fb:50 on en7 ifscope [ethernet]
```


## 4. Utilisation d'un des deux comme gateway

Ca, ça peut toujours dépann. Comme pour donner internet à une tour sans WiFi quand y'a un PC portable à côté, par exemple. 

L'idée est la suivante :

- vos PCs ont deux cartes avec des adresses IP actuellement
  - la carte WiFi, elle permet notamment d'aller sur internet, grâce au réseau YNOV
  - la carte Ethernet, qui permet actuellement de joindre votre coéquipier, grâce au réseau que vous avez créé :)
- si on fait un tit schéma tout moche, ça donne ça :

```schema
  Internet           Internet
     |                   |
    WiFi                WiFi
     |                   |
    PC 1 ---Ethernet--- PC 2
    
- internet joignable en direct par le PC 1
- internet joignable en direct par le PC 2
```

- vous allez désactiver Internet sur une des deux machines, et vous servir de l'autre machine pour accéder à internet.

```schema
  Internet           Internet
     X                   |
     X                  WiFi
     |                   |
    PC 1 ---Ethernet--- PC 2
    
- internet joignable en direct par le PC 2
- internet joignable par le PC 1, en passant par le PC 2
```

- pour ce faiiiiiire :
  - désactivez l'interface WiFi sur l'un des deux postes
  - s'assurer de la bonne connectivité entre les deux PCs à travers le câble RJ45
  - **sur le PC qui n'a plus internet**
    - sur la carte Ethernet, définir comme passerelle l'adresse IP de l'autre PC
  - **sur le PC qui a toujours internet**
    - sur Windows, il y a une option faite exprès (google it. "share internet connection windows 10" par exemple)
    - sur GNU/Linux, faites le en ligne de commande ou utilisez [Network Manager](https://help.ubuntu.com/community/Internet/ConnectionSharing) (souvent présent sur tous les GNU/Linux communs)
    - sur MacOS : toute façon vous avez pas de ports RJ, si ? :o (google it sinon)

---

- 🌞 pour tester la connectivité à internet on fait souvent des requêtes simples vers un serveur internet connu
  - encore une fois, un ping vers un DNS connu comme `1.1.1.1` ou `8.8.8.8` c'est parfait
- 🌞 utiliser un `traceroute` ou `tracert` pour bien voir que les requêtes passent par la passerelle choisie (l'autre le PC)

```
pierre in ~/Ynov/TP-Leo on main ● λ traceroute 1.1.1.1
traceroute to 1.1.1.1 (1.1.1.1), 64 hops max, 52 byte packets
 1  10.42.0.1 (10.42.0.1)  2.240 ms  1.728 ms  1.716 ms
 2  10.33.19.254 (10.33.19.254)  10.015 ms  6.610 ms  5.421 ms
 3  77.196.149.137 (77.196.149.137)  7.073 ms  5.043 ms  6.119 ms
 4  212.30.97.108 (212.30.97.108)  11.836 ms  12.928 ms  11.890 ms
 5  77.136.172.222 (77.136.172.222)  24.746 ms  25.119 ms  24.035 ms
 6  77.136.172.221 (77.136.172.221)  22.290 ms  24.031 ms  22.527 ms
 7  77.136.10.221 (77.136.10.221)  24.212 ms  24.861 ms  24.207 ms
 8  77.136.10.221 (77.136.10.221)  24.557 ms  24.163 ms  25.949 ms
 9  141.101.67.254 (141.101.67.254)  24.733 ms  23.722 ms  30.411 ms
10  172.71.120.2 (172.71.120.2)  32.182 ms
    172.71.124.2 (172.71.124.2)  27.189 ms
    172.71.120.2 (172.71.120.2)  28.005 ms
11  1.1.1.1 (1.1.1.1)  25.461 ms  24.462 ms  22.352 ms
```

## 5. Petit chat privé

On va créer un chat extrêmement simpliste à l'aide de `netcat` (abrégé `nc`). Il est souvent considéré comme un bon couteau-suisse quand il s'agit de faire des choses avec le réseau.

Sous GNU/Linux et MacOS vous l'avez sûrement déjà, sinon débrouillez-vous pour l'installer :). Les Windowsien, ça se passe [ici](https://eternallybored.org/misc/netcat/netcat-win32-1.11.zip) (from https://eternallybored.org/misc/netcat/).  

Une fois en possession de `netcat`, vous allez pouvoir l'utiliser en ligne de commande. Comme beaucoup de commandes sous GNU/Linux, Mac et Windows, on peut utiliser l'option `-h` (`h` pour `help`) pour avoir une aide sur comment utiliser la commande.  

Sur un Windows, ça donne un truc comme ça :

```schema
C:\Users\It4\Desktop\netcat-win32-1.11>nc.exe -h
[v1.11 NT www.vulnwatch.org/netcat/]
connect to somewhere:   nc [-options] hostname port[s] [ports] ...
listen for inbound:     nc -l -p port [options] [hostname] [port]
options:
        -d              detach from console, background mode

        -e prog         inbound program to exec [dangerous!!]
        -g gateway      source-routing hop point[s], up to 8
        -G num          source-routing pointer: 4, 8, 12, ...
        -h              this cruft
        -i secs         delay interval for lines sent, ports scanned
        -l              listen mode, for inbound connects
        -L              listen harder, re-listen on socket close
        -n              numeric-only IP addresses, no DNS
        -o file         hex dump of traffic
        -p port         local port number
        -r              randomize local and remote ports
        -s addr         local source address
        -t              answer TELNET negotiation
        -u              UDP mode
        -v              verbose [use twice to be more verbose]
        -w secs         timeout for connects and final net reads
        -z              zero-I/O mode [used for scanning]
port numbers can be individual or ranges: m-n [inclusive]
```

L'idée ici est la suivante :

- l'un de vous jouera le rôle d'un *serveur*
- l'autre sera le *client* qui se connecte au *serveur*

Précisément, on va dire à `netcat` d'*écouter sur un port*. Des ports, y'en a un nombre fixe (65536, on verra ça plus tard), et c'est juste le numéro de la porte à laquelle taper si on veut communiquer avec le serveur.

Si le serveur écoute à la porte 20000, alors le client doit demander une connexion en tapant à la porte numéro 20000, simple non ?  

Here we go :

- 🌞 **sur le PC *serveur*** avec par exemple l'IP 192.168.1.1
  - `nc.exe -l -p 8888`
    - "`netcat`, écoute sur le port numéro 8888 stp"
  - il se passe rien ? Normal, faut attendre qu'un client se connecte
- 🌞 **sur le PC *client*** avec par exemple l'IP 192.168.1.2
  - `nc.exe 192.168.1.1 8888`
    - "`netcat`, connecte toi au port 8888 de la machine 192.168.1.1 stp"
  - une fois fait, vous pouvez taper des messages dans les deux sens
- appelez-moi quand ça marche ! :)
  - si ça marche pas, essayez d'autres options de `netcat`

```
pierre in ~/Ynov/TP-Leo on main λ nc 10.42.0.1 12345
,xdn,d
jkqzfjmd
TROP MARRANT
t nul
```

---

- 🌞 pour aller un peu plus loin
  - le serveur peut préciser sur quelle IP écouter, et ne pas répondre sur les autres
  - par exemple, on écoute sur l'interface Ethernet, mais pas sur la WiFI
  - pour ce faire `nc.exe -l -p PORT_NUMBER IP_ADDRESS`
  - par exemple `nc.exe -l -p 9999 192.168.1.37`
  - on peut aussi accepter uniquement les connexions internes à la machine en écoutant sur `127.0.0.1`

```
pierre in ~/Ynov/TP-Leo on main λ nc 10.42.0.1 12345
jqhjqshkjqs
urs
montre kool

ta ligne de commande
stp
pour mon rendu
ok je t'envoie ça sur discord
nn ici
nc -lnvp 12345 -s 10.42.0.1
merci
```

## 6. Firewall

Toujours par 2.

Le but est de configurer votre firewall plutôt que de le désactiver

- Activez votre firewall
- 🌞 Autoriser les `ping`

```
pierre in ~/Ynov/TP-Leo on main λ ping 10.42.0.1
PING 10.42.0.1 (10.42.0.1): 56 data bytes
64 bytes from 10.42.0.1: icmp_seq=0 ttl=64 time=1.869 ms
64 bytes from 10.42.0.1: icmp_seq=1 ttl=64 time=2.059 ms
```

  - configurer le firewall de votre OS pour accepter le `ping`
  - aidez vous d'internet
  - on rentrera dans l'explication dans un prochain cours mais sachez que `ping` envoie un message *ICMP de type 8* (demande d'ECHO) et reçoit un message *ICMP de type 0* (réponse d'écho) en retour
- 🌞 Autoriser le traffic sur le port qu'utilise `nc`

```
pierre in ~/Ynov/TP-Leo on main λ nc 10.42.0.1 12345
allo
t'as un firewall??
OUI !!!
génial!!!!
+ Je suis sécurisé :)
ratio
k
```

  - on parle bien d'ouverture de **port** TCP et/ou UDP
  - on ne parle **PAS** d'autoriser le programme `nc`
  - choisissez arbitrairement un port entre 1024 et 20000
  - vous utiliserez ce port pour [communiquer avec `netcat`](#5-petit-chat-privé-) par groupe de 2 toujours
  - le firewall du *PC serveur* devra avoir un firewall activé et un `netcat` qui fonctionne
  
# III. Manipulations d'autres outils/protocoles côté client

## 1. DHCP

Bon ok vous savez définir des IPs à la main. Mais pour être dans le réseau YNOV, vous l'avez jamais fait.  

C'est le **serveur DHCP** d'YNOV qui vous a donné une IP.

Une fois que le serveur DHCP vous a donné une IP, vous enregistrer un fichier appelé *bail DHCP* qui contient, entre autres :

- l'IP qu'on vous a donné
- le réseau dans lequel cette IP est valable

🌞Exploration du DHCP, depuis votre PC

```
pierre in ~/Ynov/TP-Leo on main λ ipconfig getpacket en0
op = BOOTREPLY
[...]
lease_time (uint32): 0x29247
[...]
```

- afficher l'adresse IP du serveur DHCP du réseau WiFi YNOV
- cette adresse a une durée de vie limitée. C'est le principe du ***bail DHCP*** (ou *DHCP lease*). Trouver la date d'expiration de votre bail DHCP
- vous pouvez vous renseigner un peu sur le fonctionnement de DHCP dans les grandes lignes. On aura sûrement un cours là dessus :)

## 2. DNS

Le protocole DNS permet la résolution de noms de domaine vers des adresses IP. Ce protocole permet d'aller sur `google.com` plutôt que de devoir connaître et utiliser l'adresse IP du serveur de Google.  

Un **serveur DNS** est un serveur à qui l'on peut poser des questions (= effectuer des requêtes) sur un nom de domaine comme `google.com`, afin d'obtenir les adresses IP liées au nom de domaine.  

Si votre navigateur fonctionne "normalement" (il vous permet d'aller sur `google.com` par exemple) alors votre ordinateur connaît forcément l'adresse d'un serveur DNS. Et quand vous naviguez sur internet, il effectue toutes les requêtes DNS à votre place, de façon automatique.

- 🌞 trouver l'adresse IP du serveur DNS que connaît votre ordinateur

```
pierre in ~/Ynov/TP-Leo on main λ ipconfig getpacket en0
[...]
domain_name_server (ip_mult): {8.8.8.8, 8.8.4.4, 1.1.1.1}
[...]
```

- 🌞 utiliser, en ligne de commande l'outil `nslookup` (Windows, MacOS) ou `dig` (GNU/Linux, MacOS) pour faire des requêtes DNS à la main

  - faites un *lookup* (*lookup* = "dis moi à quelle IP se trouve tel nom de domaine")
    - pour `google.com`
    - pour `ynov.com`
    - interpréter les résultats de ces commandes
  - déterminer l'adresse IP du serveur à qui vous venez d'effectuer ces requêtes

```
pierre in ~/Ynov/TP-Leo on main ● λ dig google.com
[...]
;; ANSWER SECTION:
google.com.		191	IN	A	172.217.18.206
[...]

pierre in ~/Ynov/TP-Leo on main ● λ dig ynov.com
[...]
;; ANSWER SECTION:
ynov.com.		256	IN	A	104.26.11.233
ynov.com.		256	IN	A	104.26.10.233
ynov.com.		256	IN	A	172.67.74.226
[...]
```

  - faites un *reverse lookup* (= "dis moi si tu connais un nom de domaine pour telle IP")
    - pour l'adresse `78.74.21.21`
    - pour l'adresse `92.146.54.88`
    - interpréter les résultats
    - *si vous vous demandez, j'ai pris des adresses random :)*
```
pierre in ~/Ynov/TP-Leo on main ● λ dig -x 78.74.21.21
[...]
;; ANSWER SECTION:
21.21.74.78.in-addr.arpa. 3600	IN	PTR	host-78-74-21-21.homerun.telia.com.
[...]

pierre in ~/Ynov/TP-Leo on main ● λ dig -x 92.146.54.88
[...]
;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
;; QUESTION SECTION:
;88.54.146.92.in-addr.arpa.	IN	PTR

;; AUTHORITY SECTION:
92.in-addr.arpa.	3600	IN	SOA	pri.authdns.ripe.net. dns.ripe.net. 1664348702 3600 600 864000 3600
[...]
```
La première ip le serveur dns ma répondu un PTR, il a trouvé une adresse qui correspond à l'ip, par contre pour le deuxieme ce n'est pas le cas.


# IV. Wireshark

Wireshark est un outil qui permet de visualiser toutes les trames qui sortent et entrent d'une carte réseau.

Il peut :

- enregistrer le trafic réseau, pour l'analyser plus tard
- afficher le trafic réseau en temps réel

**On peut TOUT voir.**

Un peu austère aux premiers abords, une manipulation très basique permet d'avoir une très bonne compréhension de ce qu'il se passe réellement.

- téléchargez l'outil [Wireshark](https://www.wireshark.org/)
- 🌞 utilisez le pour observer les trames qui circulent entre vos deux carte Ethernet. Mettez en évidence :
  - un `ping` entre vous et la passerelle
  - un `netcat` entre vous et votre mate, branché en RJ45
  - une requête DNS. Identifiez dans la capture le serveur DNS à qui vous posez la question.
  - prenez moi des screens des trames en question
  - on va prendre l'habitude d'utiliser Wireshark souvent dans les cours, pour visualiser ce qu'il se passe

![](./wireshark-nc.png)
![](./pingwrshk.png)

# Bilan

🌞 Ce soleil est un troll. **Lisez et prenez le temps d'appréhender le texte de conclusion juste au dessus si ces notions ne vous sont pas familières.**

![](./cat.jpeg)