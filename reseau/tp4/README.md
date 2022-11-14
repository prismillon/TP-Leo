# TP4 : TCP, UDP et services réseau

Dans ce TP on va explorer un peu les protocoles TCP et UDP. On va aussi mettre en place des services qui font appel à ces protocoles.

# 0. Prérequis

➜ Pour ce TP, on va se servir de VMs Rocky Linux. 1Go RAM c'est large large. Vous pouvez redescendre la mémoire vidéo aussi.  

➜ Les firewalls de vos VMs doivent **toujours** être actifs (et donc correctement configurés).

➜ **Si vous voyez le p'tit pote 🦈 c'est qu'il y a un PCAP à produire et à mettre dans votre dépôt git de rendu.**

# I. First steps

Faites-vous un petit top 5 des applications que vous utilisez sur votre PC souvent, des applications qui utilisent le réseau : un site que vous visitez souvent, un jeu en ligne, Spotify, j'sais po moi, n'importe.

🌞 **Déterminez, pour ces 5 applications, si c'est du TCP ou de l'UDP**

- avec Wireshark, on va faire les chirurgiens réseau
- déterminez, pour chaque application :
  - IP et port du serveur auquel vous vous connectez
  - le port local que vous ouvrez pour vous connecter

- Discord:
    - ip:port discord 162.159.128.233:443
    - mon port: 55028
    ```
    pierre in ~/Ynov/TP-Leo/tp-r4 on main λ lsof -nP -i TCP | grep Discord
    Discord   65176 pierre   24u  IPv4 0x82a5ff8966c79111      0t0  TCP 10.33.16.106:50667->162.159.133.234:443 (ESTABLISHED)
    Discord   65176 pierre   25u  IPv4 0x82a5ff8966c8e6b1      0t0  TCP 10.33.16.106:50680->35.186.224.47:443 (ESTABLISHED)
    Discord   65179 pierre   51u  IPv4 0x82a5ff8966c71c19      0t0  TCP 127.0.0.1:6463 (LISTEN)
    ```
    - [🦈 discord](./discord.pcapng)

- spotify:
    - ip:port 35.186.224.25:443
    - mon port: 51752
    ```
    pierre in ~/Ynov/TP-Leo/tp-r4 on main λ lsof -nP -i TCP | grep Spotify
    Spotify   65198 pierre   28u  IPv4 0x82a5ff8966c766b1      0t0  TCP *:50670 (LISTEN)
    Spotify   65198 pierre   50u  IPv4 0x82a5ff8966c7c679      0t0  TCP 10.33.16.106:50682->104.199.65.124:4070 (ESTABLISHED)
    Spotify   65198 pierre   53u  IPv4 0x82a5ff8966c7e6b1      0t0  TCP *:57621 (LISTEN)
    Spotify   65198 pierre  112u  IPv4 0x82a5ff8966c99111      0t0  TCP 10.33.16.106:50687->35.186.224.47:443 (ESTABLISHED)
    Spotify   65209 pierre   20u  IPv4 0x82a5ff8966c7d111      0t0  TCP 10.33.16.106:50685->35.186.224.25:443 (ESTABLISHED)
    Spotify   65209 pierre   22u  IPv4 0x82a5ff8966c8dc19      0t0  TCP 10.33.16.106:50679->34.98.74.57:443 (ESTABLISHED)
    Spotify   65209 pierre   23u  IPv4 0x82a5ff8966c88679      0t0  TCP 10.33.16.106:50708->142.250.201.163:443 (ESTABLISHED)
    Spotify   65209 pierre   25u  IPv4 0x82a5ff8966c7dc19      0t0  TCP 10.33.16.106:50671->35.188.42.15:443 (CLOSE_WAIT)
    Spotify   65209 pierre   27u  IPv4 0x82a5ff8966c7b149      0t0  TCP 10.33.16.106:50681->34.98.74.57:443 (ESTABLISHED)
    Spotify   65209 pierre   29u  IPv4 0x82a5ff8966c866b1      0t0  TCP 10.33.16.106:50683->104.18.42.171:443 (ESTABLISHED)
    Spotify   65209 pierre   33u  IPv4 0x82a5ff8966c80679      0t0  TCP 10.33.16.106:50677->172.64.145.85:443 (ESTABLISHED)
    Spotify   65209 pierre   34u  IPv4 0x82a5ff8966c97be1      0t0  TCP 10.33.16.106:50688->151.101.122.248:443 (ESTABLISHED)
    Spotify   65209 pierre   36u  IPv4 0x82a5ff8966c98679      0t0  TCP 10.33.16.106:50689->151.101.122.248:443 (ESTABLISHED)
    Spotify   65209 pierre   38u  IPv4 0x82a5ff8966c95c19      0t0  TCP 10.33.16.106:50690->151.101.122.248:443 (ESTABLISHED)
    Spotify   65209 pierre   39u  IPv4 0x82a5ff8966c81111      0t0  TCP 10.33.16.106:50676->104.18.42.171:443 (ESTABLISHED)
    Spotify   65209 pierre   42u  IPv4 0x82a5ff8966c79c19      0t0  TCP 10.33.16.106:50686->35.186.224.47:443 (ESTABLISHED)
    Spotify   65209 pierre   43u  IPv4 0x82a5ff8966c966b1      0t0  TCP 10.33.16.106:50691->151.101.122.248:443 (ESTABLISHED)
    ```
    - [🦈 spotify](./spotify.pcapng)

- Ankama launcher:
    - ip:port 18.66.218.84:443
    - mon port: 50491
    ```
    pierre in ~/Ynov/TP-Leo/tp-r4 on main λ lsof -nP -i TCP | grep Ankama
    Ankama    64713 pierre   60u  IPv4 0x82a5ff8966c89111      0t0  TCP 10.33.16.106:50558->99.81.239.126:6337 (ESTABLISHED)
    Ankama    64713 pierre   61u  IPv4 0x82a5ff8966c77149      0t0  TCP 127.0.0.1:26116 (LISTEN)
    Ankama    64713 pierre   62u  IPv4 0x82a5ff8966cafbe1      0t0  TCP 127.0.0.1:26117 (LISTEN)
    Ankama    64713 pierre   63u  IPv4 0x82a5ff8966c7a6b1      0t0  TCP 10.33.16.106:50655->18.64.79.12:443 (ESTABLISHED)
    Ankama    64713 pierre   64u  IPv4 0x82a5ff8966c73be1      0t0  TCP 10.33.16.106:50712->18.64.79.74:443 (ESTABLISHED)
    Ankama    64713 pierre   65u  IPv4 0x82a5ff8966c83149      0t0  TCP 10.33.16.106:50664->18.64.79.92:443 (ESTABLISHED)
    Ankama    64713 pierre   66u  IPv4 0x82a5ff8966c85c19      0t0  TCP 10.33.16.106:50654->18.64.79.12:443 (ESTABLISHED)
    Ankama    64713 pierre   67u  IPv4 0x82a5ff896951a6b1      0t0  TCP 10.33.16.106:50808->52.222.158.33:443 (ESTABLISHED)
    Ankama    64713 pierre   70u  IPv4 0x82a5ff8969519c19      0t0  TCP 10.33.16.106:50809->52.222.158.33:443 (ESTABLISHED)
    Ankama    64720 pierre   22u  IPv4 0x82a5ff8966c74679      0t0  TCP 10.33.16.106:50491->18.66.218.84:443 (ESTABLISHED)
    Ankama    64720 pierre   23u  IPv4 0x82a5ff8966cadc19      0t0  TCP 10.33.16.106:50547->104.16.71.20:443 (ESTABLISHED)
    Ankama    64720 pierre   24u  IPv4 0x82a5ff8966c7f149      0t0  TCP 10.33.16.106:50546->104.18.42.171:443 (ESTABLISHED)
    Ankama    64720 pierre   31u  IPv4 0x82a5ff8966c7fbe1      0t0  TCP 10.33.16.106:50714->13.249.9.32:443 (ESTABLISHED)
    ```
    - [🦈 ankama launcher](./ankama.pcapng)

- dofus (dl)
    - ip:port  18.64.79.12:443
    - mes ports: 50642, 50644, 50643, 50650, 50649
    ```
    pierre in ~/Ynov/TP-Leo/tp-r4 on main λ lsof -nP -i TCP | grep Ankama
    Ankama    64713 pierre   60u  IPv4 0x82a5ff8966c89111      0t0  TCP 10.33.16.106:50558->99.81.239.126:6337 (ESTABLISHED)
    Ankama    64713 pierre   61u  IPv4 0x82a5ff8966c77149      0t0  TCP 127.0.0.1:26116 (LISTEN)
    Ankama    64713 pierre   62u  IPv4 0x82a5ff8966cafbe1      0t0  TCP 127.0.0.1:26117 (LISTEN)
    Ankama    64713 pierre   63u  IPv4 0x82a5ff8966c7a6b1      0t0  TCP 10.33.16.106:50655->18.64.79.12:443 (ESTABLISHED)
    Ankama    64713 pierre   64u  IPv4 0x82a5ff8966c73be1      0t0  TCP 10.33.16.106:50712->18.64.79.74:443 (ESTABLISHED)
    Ankama    64713 pierre   65u  IPv4 0x82a5ff8966c83149      0t0  TCP 10.33.16.106:50664->18.64.79.92:443 (ESTABLISHED)
    Ankama    64713 pierre   66u  IPv4 0x82a5ff8966c85c19      0t0  TCP 10.33.16.106:50654->18.64.79.12:443 (ESTABLISHED)
    Ankama    64713 pierre   67u  IPv4 0x82a5ff896951a6b1      0t0  TCP 10.33.16.106:50808->52.222.158.33:443 (ESTABLISHED)
    Ankama    64713 pierre   70u  IPv4 0x82a5ff8969519c19      0t0  TCP 10.33.16.106:50809->52.222.158.33:443 (ESTABLISHED)
    Ankama    64720 pierre   22u  IPv4 0x82a5ff8966c74679      0t0  TCP 10.33.16.106:50491->18.66.218.84:443 (ESTABLISHED)
    Ankama    64720 pierre   23u  IPv4 0x82a5ff8966cadc19      0t0  TCP 10.33.16.106:50547->104.16.71.20:443 (ESTABLISHED)
    Ankama    64720 pierre   24u  IPv4 0x82a5ff8966c7f149      0t0  TCP 10.33.16.106:50546->104.18.42.171:443 (ESTABLISHED)
    Ankama    64720 pierre   31u  IPv4 0x82a5ff8966c7fbe1      0t0  TCP 10.33.16.106:50714->13.249.9.32:443 (ESTABLISHED)
    ```
    - [🦈 dofus](./dofus.pcapng)   

- mail:
    - ip:port 108.177.15.109:993
    - mes ports 50890, 50891, 50892, 50893, 50897
    ```
    pierre in ~/Ynov/TP-Leo/tp-r4 on main λ lsof -nP -i TCP | grep Mail
    Mail      65485 pierre   56u  IPv4 0x82a5ff8966c75111      0t0  TCP 10.33.16.106:50890->17.42.251.56:993 (ESTABLISHED)
    Mail      65485 pierre   57u  IPv4 0x82a5ff8966c85111      0t0  TCP 10.33.16.106:50891->108.177.15.109:993 (ESTABLISHED)
    Mail      65485 pierre   58u  IPv4 0x82a5ff8966c8fbe1      0t0  TCP 10.33.16.106:50892->108.177.15.109:993 (ESTABLISHED)
    Mail      65485 pierre   59u  IPv4 0x82a5ff8966c91111      0t0  TCP 10.33.16.106:50893->108.177.15.109:993 (ESTABLISHED)
    Mail      65485 pierre   68u  IPv4 0x82a5ff8966c91111      0t0  TCP 10.33.16.106:50893->108.177.15.109:993 (ESTABLISHED)
    Mail      65485 pierre   70u  IPv4 0x82a5ff8966c8fbe1      0t0  TCP 10.33.16.106:50892->108.177.15.109:993 (ESTABLISHED)
    Mail      65485 pierre   73u  IPv4 0x82a5ff8966c85111      0t0  TCP 10.33.16.106:50891->108.177.15.109:993 (ESTABLISHED)
    Mail      65485 pierre   77u  IPv4 0x82a5ff8966c75111      0t0  TCP 10.33.16.106:50890->17.42.251.56:993 (ESTABLISHED)
    Mail      65485 pierre   99u  IPv4 0x82a5ff8966cafbe1      0t0  TCP 10.33.16.106:50897->17.42.251.56:993 (ESTABLISHED)
    Mail      65485 pierre  100u  IPv4 0x82a5ff8966cafbe1      0t0  TCP 10.33.16.106:50897->17.42.251.56:993 (ESTABLISHED)
    ```
    - [🦈 mail](./mail.pcapng)
> Dès qu'on se connecte à un serveur, notre PC ouvre un port random. Une fois la connexion TCP ou UDP établie, entre le port de notre PC et le port du serveur qui est en écoute, on parle de tunnel TCP ou de tunnel UDP.


> Aussi, TCP ou UDP ? Comment le client sait ? Il sait parce que le serveur a décidé ce qui était le mieux pour tel ou tel type de trafic (un jeu, une page web, etc.) et que le logiciel client est codé pour utiliser TCP ou UDP en conséquence.

🌞 **Demandez l'avis à votre OS**

- votre OS est responsable de l'ouverture des ports, et de placer un programme en "écoute" sur un port
- il est aussi responsable de l'ouverture d'un port quand une application demande à se connecter à distance vers un serveur
- bref il voit tout quoi
- utilisez la commande adaptée à votre OS pour repérer, dans la liste de toutes les connexions réseau établies, la connexion que vous voyez dans Wireshark, pour chacune des 5 applications

**Il faudra ajouter des options adaptées aux commandes pour y voir clair. Pour rappel, vous cherchez des connexions TCP ou UDP.**

🦈🦈🦈🦈🦈 **Bah ouais, captures Wireshark à l'appui évidemment.** Une capture pour chaque application, qui met bien en évidence le trafic en question.

# II. Mise en place

Allumez une VM Linux pour la suite.

## 1. SSH

Connectez-vous en SSH à votre VM.

🌞 **Examinez le trafic dans Wireshark**

- donnez un sens aux infos devant vos yeux, capturez un peu de trafic, et coupez la capture, sélectionnez une trame random et regardez dedans, vous laissez pas brainfuck par Wireshark n_n
- **déterminez si SSH utilise TCP ou UDP**
  - pareil réfléchissez-y deux minutes, logique qu'on utilise pas UDP non ?
- **repérez le *3-Way Handshake* à l'établissement de la connexion**
  - c'est le `SYN` `SYNACK` `ACK`
    [🦈 SYN](./synssh.pcapng)
- **repérez le FIN FINACK à la fin d'une connexion**
    - entre le *3-way handshake* et l'échange `FIN`, c'est juste une bouillie de caca chiffré, dans un tunnel TCP
    [🦈 FIN](./finssh.pcapng)

🌞 **Demandez aux OS**

- repérez, avec une commande adaptée, la connexion SSH depuis votre machine
- ET repérez la connexion SSH depuis votre VM

```
pierre in ~/Ynov/TP-Leo/tp-r4 on main λ lsof -nP -i TCP | grep ssh
ssh       66177 pierre    3u  IPv4 0x82a5ff8966c7bbe1      0t0  TCP 10.37.129.1:50990->10.37.129.2:22 (ESTABLISHED)
```

```
[user@localhost ~]$ ss -npt
State   Recv-Q   Send-Q     Local Address:Port     Peer Address:Port    Process
ESTAB   0        52           10.37.129.2:22        10.37.129.1:50986
```

[🦈 ssh complet](./fullssh.pcapng)

🦈 **Je veux une capture clean avec le 3-way handshake, un peu de trafic au milieu et une fin de connexion**

## 2. NFS

Allumez une deuxième VM Linux pour cette partie.

Vous allez installer un serveur NFS. Un serveur NFS c'est juste un programme qui écoute sur un port (comme toujours en fait, oèoèoè) et qui propose aux clients d'accéder à des dossiers à travers le réseau.

Une de vos VMs portera donc le serveur NFS, et l'autre utilisera un dossier à travers le réseau.

🌞 **Mettez en place un petit serveur NFS sur l'une des deux VMs**

- j'vais pas ré-écrire la roue, google it, ou [go ici](https://www.server-world.info/en/note?os=Rocky_Linux_8&p=nfs&f=1)
- partagez un dossier que vous avez créé au préalable dans `/srv`
- vérifiez que vous accédez à ce dossier avec l'autre machine : [le client NFS](https://www.server-world.info/en/note?os=Rocky_Linux_8&p=nfs&f=2)

```bash
[user@localhost ~]$ sudo mount -t nfs 192.168.64.2:/srv/nfsshare /mnt
[user@localhost ~]$ df -hT /mnt
Filesystem                 Type  Size  Used Avail Use% Mounted on
192.168.64.2:/srv/nfsshare nfs4  5.6G  1.4G  4.3G  25% /mnt
[user@localhost ~]$
```

> Si besoin, comme d'hab, je peux aider à la compréhension, n'hésitez pas à m'appeler.

🌞 **Wireshark it !**

- une fois que c'est en place, utilisez `tcpdump` pour capturer du trafic NFS
- déterminez le port utilisé par le serveur

```
[user@localhost srv]$ sudo tcpdump -A -s0 port 2049
dropped privs to tcpdump
tcpdump: verbose output suppressed, use -v[v]... for full protocol decode
listening on enp0s5, link-type EN10MB (Ethernet), snapshot length 262144 bytes
01:40:23.641194 IP 192.168.64.3.qrh > localhost.localdomain.nfs: Flags [P.], seq 2656590052:2656590248, ack 2175150243, win 501, options [nop,nop,TS val 781800685 ecr 2991186232], length 196: NFS request xid 2543604821 192 getattr fh 0,2/53
E...i.@.@.....@...@......XX...(.....Gw.....
..T..I.8......TU...........................4........localhost.localdomain..................
...........................5	JGc.!.r...........?........................=........L^
.GM....H...4...	...........:
01:40:23.641313 IP localhost.localdomain.nfs > 192.168.64.3.qrh: Flags [P.], seq 1:245, ack 196, win 501, options [nop,nop,TS val 2991215847 ecr 781800685], length 244: NFS reply xid 2543604821 reply ok 240 getattr NON 3 ids 0/155862883 sz 2770455922
E..([.@.@..g..@...@.......(..XY.....]......
.JT...T.......TU...................................5....	JGc.!.r...........?...........................	...............:.........................L^
.GM....H...4.......=............0.......0.......................cGPD3.r.....cF..........cF.............=
01:40:23.641747 IP 192.168.64.3.qrh > localhost.localdomain.nfs: Flags [.], ack 245, win 501, options [nop,nop,TS val 781800698 ecr 2991215847], length 0
E..4i.@.@..e..@...@......XY...).....:}.....
..T..JT.
01:40:23.641862 IP 192.168.64.3.qrh > localhost.localdomain.nfs: Flags [P.], seq 196:504, ack 245, win 501, options [nop,nop,TS val 781800698 ecr 2991215847], length 308: NFS request xid 2560382037 304 getattr fh 0,2/53
E..hi	@.@..0..@...@......XY...)......J.....
..T..JT....0..TU...........................4........localhost.localdomain..................
...........................5	JGc.!.r...........@........................=........L^
.GM....H...4................	JGc.!.r....open id:...+........3wu.............................................tets...
...........	...........:
01:40:23.641927 IP localhost.localdomain.nfs > 192.168.64.3.qrh: Flags [P.], seq 245:345, ack 504, win 501, options [nop,nop,TS val 2991215848 ecr 781800698], length 100: NFS reply xid 2560382037 reply ok 96 getattr ERROR: Permission denied
E...[.@.@.....@...@.......)..XZ......S.....
.JT...T....`..TU...................................5....	JGc.!.r...........@...............................
01:40:23.642280 IP 192.168.64.3.qrh > localhost.localdomain.nfs: Flags [P.], seq 504:716, ack 345, win 501, options [nop,nop,TS val 781800699 ecr 2991215848], length 212: NFS request xid 2577159253 208 getattr fh 0,2/53
E...i
@.@.....@...@......XZ...)......N.....
..T..JT.......TU...........................4........localhost.localdomain..................
...........................5	JGc.!.r...........A........................=........L^
.GM....H...4........tets...
...	...........:
01:40:23.642333 IP localhost.localdomain.nfs > 192.168.64.3.qrh: Flags [P.], seq 345:445, ack 716, win 501, options [nop,nop,TS val 2991215848 ecr 781800699], length 100: NFS reply xid 2577159253 reply ok 96 getattr ERROR: No such file or directory
E...[.@.@.....@...@.......)..X[......2.....
.JT...T....`..TU...................................5....	JGc.!.r...........A................................
01:40:23.699786 IP 192.168.64.3.qrh > localhost.localdomain.nfs: Flags [.], ack 445, win 501, options [nop,nop,TS val 781800750 ecr 2991215848], length 0
E..4i.@.@..b..@...@......X[...*_....7x.....
..U..JT.
^C
8 packets captured
9 packets received by filter
0 packets dropped by kernel
```
le port utilisé est 2049, bon jle sais par ce qu'il fallait l'ouvrir mais vu que mes vm on internet et que j'ai la flemme de tout reconfig je fais pas une capture sans filtre de port sinon c'est illisible

🌞 **Demandez aux OS**

- repérez, avec un commande adaptée, la connexion NFS sur le client et sur le serveur

```
# GNU/Linux
$ ss
```

coté serveur:
```
[user@localhost srv]$ sudo ss -ntp
[sudo] password for user:
State Recv-Q Send-Q  Local Address:Port   Peer Address:Port  Process
ESTAB 0      0        192.168.64.2:2049   192.168.64.3:752
ESTAB 0      0        192.168.64.2:22     192.168.64.1:51022  users:(("sshd",pid=26262,fd=4),("sshd",pid=26258,fd=4))
``` 

coté client
```
[user@localhost mnt]$ sudo ss -ntp
[sudo] password for user:
State  Recv-Q  Send-Q   Local Address:Port   Peer Address:Port   Process
ESTAB  0       0         192.168.64.3:22     192.168.64.1:57157   users:(("sshd",pid=801,fd=4),("sshd",pid=783,fd=4))
ESTAB  0       0         192.168.64.3:752    192.168.64.2:2049
```

🦈 **Et vous me remettez une capture de trafic NFS** la plus complète possible. J'ai pas dit que je voulais le plus de trames possible, mais juste, ce qu'il faut pour avoir un max d'infos sur le trafic

[nfs pcap](./nfs.pcapng)

## 3. DNS

🌞 Utilisez une commande pour effectuer une requête DNS depuis une des VMs

- capturez le trafic avec un `tcpdump`
- déterminez le port et l'IP du serveur DNS auquel vous vous connectez

```
[user@localhost ~]$ sudo tcpdump -w /home/dns.pcap port not 22 & host google.com
[3] 1137
google.com has address 142.250.201.174
google.com has IPv6 address 2a00:1450:4007:80e::200e
google.com mail is handled by 10 smtp.google.com.
[user@localhost ~]$ dropped privs to tcpdump
tcpdump: listening on enp0s5, link-type EN10MB (Ethernet), snapshot length 262144 bytes

[user@localhost ~]$ sudo tcpdump -n -t -r dns.pcap port 53
reading from file dns.pcap, link-type EN10MB (Ethernet), snapshot length 262144
dropped privs to tcpdump
IP 192.168.64.1.domain > 192.168.64.3.52464: 53345 1/0/0 A 216.58.215.46 (44)
IP 192.168.64.3.51013 > 192.168.64.1.domain: 51135+ AAAA? google.com. (28)
IP 192.168.64.1.domain > 192.168.64.3.51013: 51135 1/0/0 AAAA 2a00:1450:4007:818::200e (56)
IP 192.168.64.3.47962 > 192.168.64.1.domain: 18404+ MX? google.com. (28)
IP 192.168.64.1.domain > 192.168.64.3.47962: 18404 1/0/0 MX smtp.google.com. 10 (49)
IP 192.168.64.3.53829 > 192.168.64.1.domain: 15474+ A? google.com. (28)
IP 192.168.64.1.domain > 192.168.64.3.53829: 15474 1/0/0 A 216.58.215.46 (44)
IP 192.168.64.3.36071 > 192.168.64.1.domain: 2277+ AAAA? google.com. (28)
IP 192.168.64.1.domain > 192.168.64.3.36071: 2277 1/0/0 AAAA 2a00:1450:4007:818::200e (56)
IP 192.168.64.3.50860 > 192.168.64.1.domain: 11693+ MX? google.com. (28)
IP 192.168.64.1.domain > 192.168.64.3.50860: 11693 1/0/0 MX smtp.google.com. 10 (49)
```

je suis pas sur d'avoir fait ce qu'il fallait mais j'ai l'impression que la resquete est passé a la machine hote (192.168.64.1.domain) et qu'elle répond la requete une fois l'avoir effectué, elle fait surement office de relais mais du coup j'imagine que la réponse c'est ma machine hote.
