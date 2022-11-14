# TP1 : (re)Familiaration avec un système GNU/Linux

Dans ce TP, on va passer en revue des éléments de configurations élémentaires du système.

Vous pouvez effectuer ces actions dans la première VM. On la clonera ensuite avec toutes les configurations pré-effectuées.

Au menu :

- gestion d'utilisateurs
  - sudo
  - SSH et clés
- configuration réseau
- gestion de partitions
- gestion de services

![Heyyyy](./pics/hey.jpeg)

## Sommaire

- [TP1 : (re)Familiaration avec un système GNU/Linux](#tp1--refamiliaration-avec-un-système-gnulinux)
  - [Sommaire](#sommaire)
  - [0. Préparation de la machine](#0-préparation-de-la-machine)
  - [I. Utilisateurs](#i-utilisateurs)
    - [1. Création et configuration](#1-création-et-configuration)
    - [2. SSH](#2-ssh)
  - [II. Partitionnement](#ii-partitionnement)
    - [1. Préparation de la VM](#1-préparation-de-la-vm)
    - [2. Partitionnement](#2-partitionnement)
  - [III. Gestion de services](#iii-gestion-de-services)
  - [1. Interaction avec un service existant](#1-interaction-avec-un-service-existant)
  - [2. Création de service](#2-création-de-service)
    - [A. Unité simpliste](#a-unité-simpliste)
    - [B. Modification de l'unité](#b-modification-de-lunité)

## 0. Préparation de la machine

> **POUR RAPPEL** pour chacune des opérations, vous devez fournir dans le compte-rendu : comment réaliser l'opération ET la preuve que l'opération a été bien réalisée

🌞 **Setup de deux machines Rocky Linux configurées de façon basique.**

jui sous arm donc j'utilise utm, j'ai le droit qu'a une seule interface donc j'ai sois internet soit des ip custom :(
```
TP-Leo/linux on  main [✘?] took 2s
❯ ssh user@192.168.64.4
Activate the web console with: systemctl enable --now cockpit.socket

Last login: Mon Nov 14 11:25:59 2022 from 192.168.64.1
[user@localhost ~]$
```
```
~
❯ ssh user@192.168.64.5
Activate the web console with: systemctl enable --now cockpit.socket

Last login: Mon Nov 14 11:26:41 2022 from 192.168.64.1
[user@localhost ~]$
```

- **un accès internet (via la carte NAT)**
  - carte réseau dédiée
  - route par défaut

je peux pas avec utm :(

- **un accès à un réseau local** (les deux machines peuvent se `ping`) (via la carte Host-Only)
  - carte réseau dédiée (host-only sur VirtualBox)
  - les machines doivent posséder une IP statique sur l'interface host-only

```
[user@node1 ~]$ ping 192.168.64.5
PING 192.168.64.5 (192.168.64.5) 56(84) bytes of data.
64 bytes from 192.168.64.5: icmp_seq=1 ttl=64 time=12.1 ms
64 bytes from 192.168.64.5: icmp_seq=2 ttl=64 time=7.15 ms
^C
--- 192.168.64.5 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1004ms
rtt min/avg/max/mdev = 7.147/9.624/12.102/2.477 ms
[user@node1 ~]$
```

- **vous n'utilisez QUE `ssh` pour administrer les machines**

- **les machines doivent avoir un nom**
  - référez-vous au mémo
  - les noms que doivent posséder vos machines sont précisés dans le tableau plus bas

```
[user@localhost ~]$ sudo hostnamectl set-hostname node1.tp1.b2
```
```
[user@localhost ~]$ sudo hostnamectl set-hostname node2.tp1.b2
```

- **utiliser `1.1.1.1` comme serveur DNS**
  - référez-vous au mémo
  - vérifier avec le bon fonctionnement avec la commande `dig`
    - avec `dig`, demander une résolution du nom `ynov.com`
    - mettre en évidence la ligne qui contient la réponse : l'IP qui correspond au nom demandé
    - mettre en évidence la ligne qui contient l'adresse IP du serveur qui vous a répondu

```
[user@node1 ~]$ dig ynov.com

; <<>> DiG 9.16.23-RH <<>> ynov.com
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 58909
;; flags: qr rd ra; QUERY: 1, ANSWER: 3, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
;; QUESTION SECTION:
;ynov.com.			IN	A

;; ANSWER SECTION:
ynov.com.		14	IN	A	172.67.74.226
ynov.com.		14	IN	A	104.26.11.233
ynov.com.		14	IN	A	104.26.10.233

;; Query time: 89 msec
;; SERVER: 1.1.1.1#53(1.1.1.1)
;; WHEN: Mon Nov 14 12:38:08 CET 2022
;; MSG SIZE  rcvd: 85

[user@node1 ~]$
```

```
[user@node2 ~]$ dig ynov.com

; <<>> DiG 9.16.23-RH <<>> ynov.com
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 14619
;; flags: qr rd ra; QUERY: 1, ANSWER: 3, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
;; QUESTION SECTION:
;ynov.com.			IN	A

;; ANSWER SECTION:
ynov.com.		67	IN	A	104.26.11.233
ynov.com.		67	IN	A	172.67.74.226
ynov.com.		67	IN	A	104.26.10.233

;; Query time: 50 msec
;; SERVER: 1.1.1.1#53(1.1.1.1)
;; WHEN: Mon Nov 14 12:38:14 CET 2022
;; MSG SIZE  rcvd: 85

[user@node2 ~]$
```

- **les machines doivent pouvoir se joindre par leurs noms respectifs**
  - fichier `/etc/hosts`
  - assurez-vous du bon fonctionnement avec des `ping <NOM>`

```
[user@node1 ~]$ ping node2.tp1.b2
PING node2.tp1.b2 (192.168.64.5) 56(84) bytes of data.
64 bytes from node2.tp1.b2 (192.168.64.5): icmp_seq=1 ttl=64 time=12.1 ms
64 bytes from node2.tp1.b2 (192.168.64.5): icmp_seq=2 ttl=64 time=5.98 ms
64 bytes from node2.tp1.b2 (192.168.64.5): icmp_seq=3 ttl=64 time=3.04 ms
^C
--- node2.tp1.b2 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2009ms
rtt min/avg/max/mdev = 3.043/7.024/12.053/3.752 ms
[user@node1 ~]$
```
```
[user@node2 ~]$ ping node1.tp1.b2
PING node1.tp1.b2 (192.168.64.4) 56(84) bytes of data.
64 bytes from node1.tp1.b2 (192.168.64.4): icmp_seq=1 ttl=64 time=1.07 ms
64 bytes from node1.tp1.b2 (192.168.64.4): icmp_seq=2 ttl=64 time=2.73 ms
64 bytes from node1.tp1.b2 (192.168.64.4): icmp_seq=3 ttl=64 time=4.84 ms
^C
--- node1.tp1.b2 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2006ms
rtt min/avg/max/mdev = 1.071/2.881/4.840/1.542 ms
[user@node2 ~]$
```

- **le pare-feu est configuré pour bloquer toutes les connexions exceptées celles qui sont nécessaires**
  - commande `firewall-cmd`

```
[user@node1 ~]$ sudo firewall-cmd --list-all
public (active)
  target: default
  icmp-block-inversion: no
  interfaces: enp0s1
  sources:
  services: cockpit dhcpv6-client ssh
  ports:
  protocols:
  forward: yes
  masquerade: no
  forward-ports:
  source-ports:
  icmp-blocks:
  rich rules:
[user@node1 ~]$
```
```
[user@node2 ~]$ sudo firewall-cmd --list-all
[sudo] password for user:
public (active)
  target: default
  icmp-block-inversion: no
  interfaces: enp0s1
  sources:
  services: cockpit dhcpv6-client ssh
  ports:
  protocols:
  forward: yes
  masquerade: no
  forward-ports:
  source-ports:
  icmp-blocks:
  rich rules:
[user@node2 ~]$
```

Pour le réseau des différentes machines (ce sont les IP qui doivent figurer sur les interfaces host-only):

| Name               | IP            |
|--------------------|---------------|
| 🖥️ `node1.tp1.b2` | `10.101.1.11` |
| 🖥️ `node2.tp1.b2` | `10.101.1.12` |
| Votre hôte         | `10.101.1.1`  |

## I. Utilisateurs

[Une section dédiée aux utilisateurs est dispo dans le mémo Linux.](../../cours/memos/commandes.md#gestion-dutilisateurs).

### 1. Création et configuration

🌞 **Ajouter un utilisateur à la machine**, qui sera dédié à son administration

- précisez des options sur la commande d'ajout pour que :
  - le répertoire home de l'utilisateur soit précisé explicitement, et se trouve dans `/home`
  - le shell de l'utilisateur soit `/bin/bash`
- prouvez que vous avez correctement créé cet utilisateur
  - et aussi qu'il a le bon shell et le bon homedir

```
[user@node1 ~]$ useradd toto -m -s /bin/sh
useradd: Permission denied.
useradd: cannot lock /etc/passwd; try again later.
[user@node1 ~]$ sudo !!
sudo useradd toto -m -s /bin/sh
[sudo] password for user:
[user@node1 ~]$ sudo passwd toto
Changing password for user toto.
New password:
BAD PASSWORD: The password is shorter than 8 characters
Retype new password:
passwd: all authentication tokens updated successfully.
[user@node1 ~]$ su toto
Password:
sh-5.1$ pwd
/home/toto
sh-5.1$ whoami
toto
```

🌞 **Créer un nouveau groupe `admins`** qui contiendra les utilisateurs de la machine ayant accès aux droits de `root` *via* la commande `sudo`.

```
[user@node1 ~]$ groupadd admins
groupadd: Permission denied.
groupadd: cannot lock /etc/group; try again later.
[user@node1 ~]$ sudo !!
sudo groupadd admins
```

Pour permettre à ce groupe d'accéder aux droits `root` :

- il faut modifier le fichier `/etc/sudoers`
- on ne le modifie jamais directement à la main car en cas d'erreur de syntaxe, on pourrait bloquer notre accès aux droits administrateur
- la commande `visudo` permet d'éditer le fichier, avec un check de syntaxe avant fermeture
- ajouter une ligne basique qui permet au groupe d'avoir tous les droits (inspirez vous de la ligne avec le groupe `wheel`)

```
[user@node1 ~]$ sudo visudo

...
## Allows people in group wheel to run all commands
%wheel  ALL=(ALL)       ALL
%admins ALL=(ALL)       ALL
...
```

🌞 **Ajouter votre utilisateur à ce groupe `admins`**

> Essayez d'effectuer une commande avec `sudo` peu importe laquelle, juste pour tester que vous avez le droit d'exécuter des commandes sous l'identité de `root`. Vous pouvez aussi utiliser `sudo -l` pour voir les droits `sudo` auquel votre utilisateur courant a accès.

```
[user@node1 ~]$ sudo usermod -aG admins toto

[user@node1 ~]$ su toto
Password:
sh-5.1$ sudo -l
[sudo] password for toto:
Matching Defaults entries for toto on node1:
    !visiblepw, always_set_home,
    match_group_by_gid,
    always_query_group_plugin, env_reset,
    env_keep="COLORS DISPLAY HOSTNAME HISTSIZE
    KDEDIR LS_COLORS", env_keep+="MAIL PS1 PS2
    QTDIR USERNAME LANG LC_ADDRESS LC_CTYPE",
    env_keep+="LC_COLLATE LC_IDENTIFICATION
    LC_MEASUREMENT LC_MESSAGES",
    env_keep+="LC_MONETARY LC_NAME LC_NUMERIC
    LC_PAPER LC_TELEPHONE", env_keep+="LC_TIME
    LC_ALL LANGUAGE LINGUAS _XKB_CHARSET
    XAUTHORITY",
    secure_path=/sbin\:/bin\:/usr/sbin\:/usr/bin

User toto may run the following commands on
        node1:
    (ALL) ALL
sh-5.1$
```

---

1. Utilisateur créé et configuré
2. Groupe `admins` créé
3. Groupe `admins` ajouté au fichier `/etc/sudoers`
4. Ajout de l'utilisateur au groupe `admins`

### 2. SSH

[Une section dédiée aux clés SSH existe dans le cours.](../../cours/SSH/README.md)

Afin de se connecter à la machine de façon plus sécurisée, on va configurer un échange de clés SSH lorsque l'on se connecte à la machine.

🌞 **Pour cela...**

- il faut générer une clé sur le poste client de l'administrateur qui se connectera à distance (vous :) )
  - génération de clé depuis VOTRE poste donc
  - sur Windows, on peut le faire avec le programme `puttygen.exe` qui est livré avec `putty.exe`
- déposer la clé dans le fichier `/home/<USER>/.ssh/authorized_keys` de la machine que l'on souhaite administrer
  - vous utiliserez l'utilisateur que vous avez créé dans la partie précédente du TP
  - on peut le faire à la main
  - ou avec la commande `ssh-copy-id`

```
~/.ssh
❯ ll
.rw-r--r--  138 pierre  9 nov 20:39  config
.rw-------  411 pierre 17 nov  2021  id_ed25519
.rw-r--r--  101 pierre 17 nov  2021  id_ed25519.pub
.rw------- 2,6k pierre 16 nov  2021  id_rsa
.rw-r--r--  587 pierre 16 nov  2021  id_rsa.pub
.rw-------  12k pierre 14 nov 11:26  known_hosts
.rw-------  11k pierre 16 oct 15:23  known_hosts.old
```

🌞 **Assurez vous que la connexion SSH est fonctionnelle**, sans avoir besoin de mot de passe.

```
TP-Leo/linux on  main [✘?] took 14s
❯ ssh-copy-id toto@192.168.64.4
/usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "/Users/pierre/.ssh/id_ed25519.pub"
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
toto@192.168.64.4's password:

Number of key(s) added:        1

Now try logging into the machine, with:   "ssh 'toto@192.168.64.4'"
and check to make sure that only the key(s) you wanted were added.


TP-Leo/linux on  main [✘?] took 2s
❯ ssh toto@192.168.64.4
Last login: Mon Nov 14 15:40:50 2022 from 192.168.64.1
[toto@node1 ~]$
```

## II. Partitionnement

[Il existe une section dédiée au partitionnement dans le cours](../../cours/part/)

### 1. Préparation de la VM

⚠️ **Uniquement sur `node1.tp1.b2`.**

Ajout de deux disques durs à la machine virtuelle, de 3Go chacun.

### 2. Partitionnement

⚠️ **Uniquement sur `node1.tp1.b2`.**

🌞 **Utilisez LVM** pour...

- agréger les deux disques en un seul *volume group*
- créer 3 *logical volumes* de 1 Go chacun
- formater ces partitions en `ext4`
- monter ces partitions pour qu'elles soient accessibles aux points de montage `/mnt/part1`, `/mnt/part2` et `/mnt/part3`.

```
[toto@node1 ~]$ sudo pvs
  PV         VG Fmt  Attr PSize PFree
  /dev/vda3  rl lvm2 a--  6.41g    0
  /dev/vdb      lvm2 ---  3.00g 3.00g
  /dev/vdc      lvm2 ---  3.00g 3.00g
[toto@node1 ~]$ sudo vgcreate data /dev/vdb
  Volume group "data" successfully created
[toto@node1 ~]$ sudo vgextend data /dev/vdc
  Volume group "data" successfully extended
[toto@node1 ~]$
```

```
[toto@node1 ~]$ sudo lvcreate -L 1G data -n data1
  Logical volume "data1" created.
[toto@node1 ~]$ sudo lvcreate -L 1G data -n data2
  Logical volume "data2" created.
[toto@node1 ~]$ sudo lvcreate -L 1G data -n data3
  Logical volume "data3" created.
[toto@node1 ~]$ sudo lvs
  LV    VG   Attr       LSize   Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  data1 data -wi-a-----   1.00g
  data2 data -wi-a-----   1.00g
  data3 data -wi-a-----   1.00g
  root  rl   -wi-ao----  <5.61g
  swap  rl   -wi-ao---- 820.00m
[toto@node1 ~]$
```

```
[toto@node1 ~]$ sudo mkfs -t ext4 /dev/data/data1
mke2fs 1.46.5 (30-Dec-2021)
Discarding device blocks: done
Creating filesystem with 262144 4k blocks and 65536 inodes
Filesystem UUID: 9e4d270b-251b-4c90-9d17-480eb2d38050
Superblock backups stored on blocks:
	32768, 98304, 163840, 229376

Allocating group tables: done
Writing inode tables: done
Creating journal (8192 blocks): done
Writing superblocks and filesystem accounting information: done

[toto@node1 ~]$ sudo mkfs -t ext4 /dev/data/data2
mke2fs 1.46.5 (30-Dec-2021)
Discarding device blocks: done
Creating filesystem with 262144 4k blocks and 65536 inodes
Filesystem UUID: e56d52fd-a0ff-4392-99e5-c33dfb789d85
Superblock backups stored on blocks:
	32768, 98304, 163840, 229376

Allocating group tables: done
Writing inode tables: done
Creating journal (8192 blocks): done
Writing superblocks and filesystem accounting information: done

[toto@node1 ~]$ sudo mkfs -t ext4 /dev/data/data3
mke2fs 1.46.5 (30-Dec-2021)
Discarding device blocks: done
Creating filesystem with 262144 4k blocks and 65536 inodes
Filesystem UUID: ba89186c-580a-4032-95c4-038d9df414f9
Superblock backups stored on blocks:
	32768, 98304, 163840, 229376

Allocating group tables: done
Writing inode tables: done
Creating journal (8192 blocks): done
Writing superblocks and filesystem accounting information: done
```

```
[toto@node1 ~]$ mkdir /mnt/part1
mkdir: cannot create directory ‘/mnt/part1’: Permission denied
[toto@node1 ~]$ sudo !!
sudo mkdir /mnt/part1
[sudo] password for toto:
Sorry, try again.
[sudo] password for toto:
[toto@node1 ~]$ sudo mkdir /mnt/part2
[toto@node1 ~]$ sudo mkdir /mnt/part3
```

```
[toto@node1 ~]$ sudo mount /dev/data/data1 /mnt/part1
[toto@node1 ~]$ sudo mount /dev/data/data2 /mnt/part2
[toto@node1 ~]$ sudo mount /dev/data/data3 /mnt/part3
```

🌞 **Grâce au fichier `/etc/fstab`**, faites en sorte que cette partition soit montée automatiquement au démarrage du système.

```
/dev/data/data1         /mnt/part1              ext4    defaults        0 0
/dev/data/data2         /mnt/part2              ext4    defaults        0 0
/dev/data/data3         /mnt/part3              ext4    defaults        0 0
```

✨**Bonus** : amusez vous avec les options de montage. Quelques options intéressantes :

- `noexec`
- `ro`
- `user`
- `nosuid`
- `nodev`
- `protect`

```
/dev/data/data1		/mnt/part1		ext4	noexec		0 0
/dev/data/data2         /mnt/part2              ext4    noexec		0 0
/dev/data/data3         /mnt/part3              ext4    noexec		0 0
```

```
[toto@node1 ~]$ sudo mount -av
/                        : ignored
/boot                    : already mounted
/boot/efi                : already mounted
none                     : ignored
mount: /mnt/part1 does not contain SELinux labels.
       You just mounted a file system that supports labels which does not
       contain labels, onto an SELinux box. It is likely that confined
       applications will generate AVC messages and not be allowed access to
       this file system.  For more details see restorecon(8) and mount(8).
/mnt/part1               : successfully mounted
mount: /mnt/part2 does not contain SELinux labels.
       You just mounted a file system that supports labels which does not
       contain labels, onto an SELinux box. It is likely that confined
       applications will generate AVC messages and not be allowed access to
       this file system.  For more details see restorecon(8) and mount(8).
/mnt/part2               : successfully mounted
mount: /mnt/part3 does not contain SELinux labels.
       You just mounted a file system that supports labels which does not
       contain labels, onto an SELinux box. It is likely that confined
       applications will generate AVC messages and not be allowed access to
       this file system.  For more details see restorecon(8) and mount(8).
/mnt/part3               : successfully mounted
[toto@node1 ~]$
```

## III. Gestion de services

Au sein des systèmes GNU/Linux les plus utilisés, c'est *systemd* qui est utilisé comme gestionnaire de services (entre autres).

Pour manipuler les services entretenus par *systemd*, on utilise la commande `systemctl`.

On peut lister les unités `systemd` actives de la machine `systemctl list-units -t service`.

**Référez-vous au mémo pour voir les autres commandes `systemctl` usuelles.**

## 1. Interaction avec un service existant

⚠️ **Uniquement sur `node1.tp1.b2`.**

Parmi les services système déjà installés sur Rocky, il existe `firewalld`. Cet utilitaire est l'outil de firewalling de Rocky.

🌞 **Assurez-vous que...**

- l'unité est démarrée
- l'unitée est activée (elle se lance automatiquement au démarrage)

```
[toto@node1 ~]$ systemctl status firewalld
● firewalld.service - firewalld - dynamic firewall daemon
     Loaded: loaded (/usr/lib/systemd/system/firewalld.service; enabled; vendor preset: enabled)
     Active: active (running) since Mon 2022-11-14 16:34:29 CET; 37s ago
       Docs: man:firewalld(1)
   Main PID: 628 (firewalld)
      Tasks: 2 (limit: 4268)
     Memory: 42.9M
        CPU: 183ms
     CGroup: /system.slice/firewalld.service
             └─628 /usr/bin/python3 -s /usr/sbin/firewalld --nofork --nopid
[toto@node1 ~]$
```

```
[toto@node1 ~]$ sudo systemctl is-enabled firewalld
[sudo] password for toto:
enabled
[toto@node1 ~]$
```


## 2. Création de service

![Création de service systemd](./pics/create_service.png)

### A. Unité simpliste

⚠️ **Uniquement sur `node1.tp1.b2`.**

🌞 **Créer un fichier qui définit une unité de service** 

- le fichier `web.service`
- dans le répertoire `/etc/systemd/system`

```
[toto@node1 ~]$ sudo vim /etc/systemd/system/web.service
```

Déposer le contenu suivant :

```
[Unit]
Description=Very simple web service

[Service]
ExecStart=/usr/bin/python3 -m http.server 8888

[Install]
WantedBy=multi-user.target
```

Le but de cette unité est de lancer un serveur web sur le port 8888 de la machine. **N'oubliez pas d'ouvrir ce port dans le firewall.**

```
[toto@node1 ~]$ sudo firewall-cmd --add-port=8888/tcp --permanent
success
```

Une fois l'unité de service créée, il faut demander à *systemd* de relire les fichiers de configuration :

```bash
$ sudo systemctl daemon-reload
```

Enfin, on peut interagir avec notre unité :

```bash
$ sudo systemctl status web
$ sudo systemctl start web
$ sudo systemctl enable web
```

🌞 **Une fois le service démarré, assurez-vous que pouvez accéder au serveur web**

- avec un navigateur depuis votre PC
- ou la commande `curl` depuis l'autre machine (je veux ça dans le compte-rendu :3)
- sur l'IP de la VM, port 8888

### B. Modification de l'unité

🌞 **Préparez l'environnement pour exécuter le mini serveur web Python**

- créer un utilisateur `web`
- créer un dossier `/var/www/meow/`
- créer un fichier dans le dossier `/var/www/meow/` (peu importe son nom ou son contenu, c'est pour tester)
- montrez à l'aide d'une commande les permissions positionnées sur le dossier et son contenu

```
[toto@node1 ~]$ sudo useradd web
[toto@node1 ~]$ mkdir /var/www/meow/
mkdir: cannot create directory ‘/var/www/meow/’: No such file or directory
[toto@node1 ~]$ mkdir /var/www
mkdir: cannot create directory ‘/var/www’: Permission denied
[toto@node1 ~]$ sudo !!
sudo mkdir /var/www
[toto@node1 ~]$ sudo mkdir /var/www/meow/
[toto@node1 ~]$ sudo touch /var/www/meow/test
[toto@node1 ~]$ ls /var/www/meow/test
/var/www/meow/test
[toto@node1 ~]$ ls /var/www/meow/
test
[toto@node1 ~]$ ls -all /var/www/meow/
total 0
drwxr-xr-x. 2 root root 18 Nov 14 16:56 .
drwxr-xr-x. 3 root root 18 Nov 14 16:56 ..
-rw-r--r--. 1 root root  0 Nov 14 16:56 test
[toto@node1 ~]$
```

> Pour que tout fonctionne correctement, il faudra veiller à ce que le dossier et le fichier appartiennent à l'utilisateur `web` et qu'il ait des droits suffisants dessus.

```
[toto@node1 www]$ sudo chown web meow/
[toto@node1 www]$ ll
total 0
drwxr-xr-x. 2 web root 18 Nov 14 16:56 meow
[toto@node1 www]$
```

🌞 **Modifiez l'unité de service `web.service` créée précédemment en ajoutant les clauses**

- `User=` afin de lancer le serveur avec l'utilisateur `web` dédié
- `WorkingDirectory=` afin de lancer le serveur depuis le dossier créé au dessus : `/var/www/meow/`
- ces deux clauses sont à positionner dans la section `[Service]` de votre unité

```
[toto@node1 ~]$ cat /etc/systemd/system/web.service
[Unit]
Description=Very simple web service

[Service]
ExecStart=/usr/bin/python3 -m http.server 8888
User=web
WorkingDirectory=/var/www/meow/

[Install]
WantedBy=multi-user.target
```

🌞 **Vérifiez le bon fonctionnement avec une commande `curl`**

```
TP-Leo/linux on  main [✘?] took 1h21m59s
❯ curl 192.168.64.4:8888
<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01//EN" "http://www.w3.org/TR/html4/strict.dtd">
<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=utf-8">
<title>Directory listing for /</title>
</head>
<body>
<h1>Directory listing for /</h1>
<hr>
<ul>
<li><a href="test">test</a></li>
</ul>
<hr>
</body>
</html>
```