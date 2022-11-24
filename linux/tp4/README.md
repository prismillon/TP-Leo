# TP4 : Conteneurs

Dans ce TP on va aborder plusieurs points autour de la conteneurisation : 

- Docker et son empreinte sur le système
- Manipulation d'images
- `docker-compose`

![Headaches](./pics/headaches.jpg)

# Sommaire

- [TP4 : Conteneurs](#tp4--conteneurs)
- [Sommaire](#sommaire)
- [0. Prérequis](#0-prérequis)
  - [Checklist](#checklist)
- [I. Docker](#i-docker)
  - [1. Install](#1-install)
  - [2. Vérifier l'install](#2-vérifier-linstall)
  - [3. Lancement de conteneurs](#3-lancement-de-conteneurs)
- [II. Images](#ii-images)
- [III. `docker-compose`](#iii-docker-compose)
  - [1. Intro](#1-intro)
  - [2. Make your own meow](#2-make-your-own-meow)

# 0. Prérequis

➜ Machines Rocky Linux

➜ Un unique host-only côté VBox, ça suffira. **L'adresse du réseau host-only sera `10.104.1.0/24`.**

➜ Chaque **création de machines** sera indiquée par **l'emoji 🖥️ suivi du nom de la machine**

➜ Si je veux **un fichier dans le rendu**, il y aura l'**emoji 📁 avec le nom du fichier voulu**. Le fichier devra être livré tel quel dans le dépôt git, ou dans le corps du rendu Markdown si c'est lisible et correctement formaté.

## Checklist

A chaque machine déployée, vous **DEVREZ** vérifier la 📝**checklist**📝 :

- [x] IP locale, statique ou dynamique
- [x] hostname défini
- [x] firewall actif, qui ne laisse passer que le strict nécessaire
- [x] SSH fonctionnel avec un échange de clé
- [x] accès Internet (une route par défaut, une carte NAT c'est très bien)
- [x] résolution de nom
- [x] SELinux désactivé (vérifiez avec `sestatus`, voir [mémo install VM tout en bas](https://gitlab.com/it4lik/b2-reseau-2022/-/blob/main/cours/memo/install_vm.md#4-pr%C3%A9parer-la-vm-au-clonage))

**Les éléments de la 📝checklist📝 sont STRICTEMENT OBLIGATOIRES à réaliser mais ne doivent PAS figurer dans le rendu.**

# I. Docker

🖥️ Machine **docker1.tp4.linux**

## 1. Install

🌞 **Installer Docker sur la machine**

- en suivant [la doc officielle](https://docs.docker.com/engine/install/)
- démarrer le service `docker` avec une commande `systemctl`
- ajouter votre utilisateur au groupe `docker`
  - cela permet d'utiliser Docker sans avoir besoin de l'identité de `root`
  - avec la commande : `sudo usermod -aG docker $(whoami)`
  - déconnectez-vous puis relancez une session pour que le changement prenne effet

```bash
sudo dnf check-update
sudo dnf config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
sudo dnf install docker-ce docker-ce-cli containerd.io
sudo systemctl start docker
sudo systemctl status docker
sudo systemctl enable docker
sudo usermod -aG docker $(whoami)
docker --version
```

```
[user@localhost ~]$ sudo dnf check-update
[sudo] password for user:
Last metadata expiration check: 0:56:44 ago on Thu 24 Nov 2022 09:31:48 AM CET.
[user@localhost ~]$ sudo dnf config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
Adding repo from: https://download.docker.com/linux/centos/docker-ce.repo
[user@localhost ~]$ sudo dnf install docker-ce docker-ce-cli containerd.io
[...]
Complete!
[user@localhost ~]$ sudo systemctl start docker
[user@localhost ~]$ sudo systemctl status docker
● docker.service - Docker Application Container Engine
     Loaded: loaded (/usr/lib/systemd/system/docker.service; disabled; vendor preset: disabled)
     Active: active (running) since Thu 2022-11-24 10:29:40 CET; 5s ago
TriggeredBy: ● docker.socket
       Docs: https://docs.docker.com
   Main PID: 2549 (dockerd)
      Tasks: 7
     Memory: 33.2M
        CPU: 75ms
     CGroup: /system.slice/docker.service
             └─2549 /usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock

Nov 24 10:29:39 localhost.localdomain dockerd[2549]: time="2022-11-24T10:29:39.982382892+01:00" lev>
Nov 24 10:29:39 localhost.localdomain dockerd[2549]: time="2022-11-24T10:29:39.982706946+01:00" lev>
Nov 24 10:29:40 localhost.localdomain dockerd[2549]: time="2022-11-24T10:29:40.005459403+01:00" lev>
Nov 24 10:29:40 localhost.localdomain dockerd[2549]: time="2022-11-24T10:29:40.216797661+01:00" lev>
Nov 24 10:29:40 localhost.localdomain dockerd[2549]: time="2022-11-24T10:29:40.264372577+01:00" lev>
Nov 24 10:29:40 localhost.localdomain dockerd[2549]: time="2022-11-24T10:29:40.337860300+01:00" lev>
Nov 24 10:29:40 localhost.localdomain dockerd[2549]: time="2022-11-24T10:29:40.358000266+01:00" lev>
Nov 24 10:29:40 localhost.localdomain dockerd[2549]: time="2022-11-24T10:29:40.358107091+01:00" lev>
Nov 24 10:29:40 localhost.localdomain systemd[1]: Started Docker Application Container Engine.
Nov 24 10:29:40 localhost.localdomain dockerd[2549]: time="2022-11-24T10:29:40.373626069+01:00" lev>
[user@localhost ~]$ sudo systemctl enable docker
Created symlink /etc/systemd/system/multi-user.target.wants/docker.service → /usr/lib/systemd/system/docker.service.
[user@localhost ~]$ sudo usermod -aG docker $(whoami)
```


## 2. Vérifier l'install

➜ **Vérifiez que Docker est actif est disponible en essayant quelques commandes usuelles :**

```bash
# Info sur l'install actuelle de Docker
$ docker info

# Liste des conteneurs actifs
$ docker ps
# Liste de tous les conteneurs
$ docker ps -a

# Liste des images disponibles localement
$ docker images

# Lancer un conteneur debian
$ docker run debian
$ docker run -d debian sleep 99999
$ docker run -it debian bash

# Consulter les logs d'un conteneur
$ docker ps # on repère l'ID/le nom du conteneur voulu
$ docker logs <ID_OR_NAME>
$ docker logs -f <ID_OR_NAME> # suit l'arrivée des logs en temps réel

# Exécuter un processus dans un conteneur actif
$ docker ps # on repère l'ID/le nom du conteneur voulu
$ docker exec <ID_OR_NAME> <COMMAND>
$ docker exec <ID_OR_NAME> ls
$ docker exec -it <ID_OR_NAME> bash # permet de récupérer un shell bash dans le conteneur ciblé
```

➜ **Explorer un peu le help**, si c'est pas le man :

```bash
$ docker --help
$ docker run --help
$ man docker
```

## 3. Lancement de conteneurs

La commande pour lancer des conteneurs est `docker run`.

Certaines options sont très souvent utilisées :

```bash
# L'option --name permet de définir un nom pour le conteneur
$ docker run --name web nginx

# L'option -d permet de lancer un conteneur en tâche de fond
$ docker run --name web -d nginx

# L'option -v permet de partager un dossier/un fichier entre l'hôte et le conteneur
$ docker run --name web -d -v /path/to/html:/usr/share/nginx/html nginx

# L'option -p permet de partager un port entre l'hôte et le conteneur
$ docker run --name web -d -v /path/to/html:/usr/share/nginx/html -p 8888:80 nginx
# Dans l'exemple ci-dessus, le port 8888 de l'hôte est partagé vers le port 80 du conteneur
```

🌞 **Utiliser la commande `docker run`**

- lancer un conteneur `nginx`
  - l'app NGINX doit avoir un fichier de conf personnalisé
  - l'app NGINX doit servir un fichier `index.html` personnalisé
  - l'application doit être joignable grâce à un partage de ports
  - vous limiterez l'utilisation de la RAM et du CPU de ce conteneur
  - le conteneur devra avoir un nom

> Tout se fait avec des options de la commande `docker run`.

```bash
[user@docker1 ~]$ docker run -p 8080:90 -v /tmp/foo/nginx.conf:/etc/nginx/conf.d/super_page.conf -v /tmp/foo/index.html:/usr/share/nginx/html/index.html -m 200m --cpus="0.2" --name=mon_super_docker --rm nginx
/docker-entrypoint.sh: /docker-entrypoint.d/ is not empty, will attempt to perform configuration
/docker-entrypoint.sh: Looking for shell scripts in /docker-entrypoint.d/
/docker-entrypoint.sh: Launching /docker-entrypoint.d/10-listen-on-ipv6-by-default.sh
10-listen-on-ipv6-by-default.sh: info: Getting the checksum of /etc/nginx/conf.d/default.conf
10-listen-on-ipv6-by-default.sh: info: Enabled listen on IPv6 in /etc/nginx/conf.d/default.conf
/docker-entrypoint.sh: Launching /docker-entrypoint.d/20-envsubst-on-templates.sh
/docker-entrypoint.sh: Launching /docker-entrypoint.d/30-tune-worker-processes.sh
/docker-entrypoint.sh: Configuration complete; ready for start up
2022/11/24 10:40:42 [notice] 1#1: using the "epoll" event method
2022/11/24 10:40:42 [notice] 1#1: nginx/1.23.2
2022/11/24 10:40:42 [notice] 1#1: built by gcc 10.2.1 20210110 (Debian 10.2.1-6)
2022/11/24 10:40:42 [notice] 1#1: OS: Linux 5.14.0-70.30.1.el9.aarch64
2022/11/24 10:40:42 [notice] 1#1: getrlimit(RLIMIT_NOFILE): 1073741816:1073741816
2022/11/24 10:40:42 [notice] 1#1: start worker processes
2022/11/24 10:40:42 [notice] 1#1: start worker process 28
10.37.129.1 - - [24/Nov/2022:10:41:06 +0000] "GET / HTTP/1.1" 304 0 "-" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/107.0.0.0 Safari/537.36" "-"
```

sur ma machine:
```
~
❯ curl 10.37.129.10:8080
j'adore les chats
```

# II. Images

La construction d'image avec Docker est basée sur l'utilisation de fichiers `Dockerfile`.

🌞 **Construire votre propre image**

- image de base
  - une image du Docker Hub
  - digne de confiance
  - qui ne porte aucune application par défaut
- vous ajouterez
  - mise à jour du système
  - installation de Apache
  - page d'accueil Apache HTML personnalisée

Exemple de `Dockerfile` qui :

- se base sur une image ubuntu
- la met à jour
- installe nginx

```Dockerfile
FROM ubuntu

RUN apt update -y

RUN apt install -y nginx
```

```
[user@docker1 ~]$ vim Dockerfile
[user@docker1 ~]$ docker build . -t my_own_apache
Sending build context to Docker daemon  19.97kB
Step 1/4 : FROM ubuntu
 ---> 3c2df5585507
Step 2/4 : RUN apt update -y
 ---> Using cache
 ---> 1a1d30d2bce9
Step 3/4 : RUN apt install -y apache2
 ---> Using cache
 ---> 8aec34d8726b
Step 4/4 : RUN echo "J'adore les chats" > /var/www/html/index.html
 ---> Running in de9f95fd8733
Removing intermediate container de9f95fd8733
 ---> 93644d889a88
Successfully built 93644d889a88
Successfully tagged my_own_apache:latest
[user@docker1 ~]$ docker images
REPOSITORY      TAG       IMAGE ID       CREATED          SIZE
my_own_apache   latest    93644d889a88   18 seconds ago   212MB
nginx           latest    6b3e1257976b   9 days ago       135MB
debian          latest    ef39063a944d   9 days ago       118MB
ubuntu          latest    3c2df5585507   3 weeks ago      69.2MB
```

```
[user@docker1 ~]$ ls
apache2.conf  Dockerfile  index.html
[user@docker1 ~]$ ls
apache2.conf  Dockerfile  index.html
[user@docker1 ~]$ cat Dockerfile
FROM ubuntu

RUN apt update -y

RUN apt install -y apache2

RUN /bin/mkdir "etc/apache2/logs/"

COPY ./index.html /var/www/html/index.html

COPY ./apache2.conf /etc/apache2/apache2.conf

CMD ["/usr/sbin/apache2ctl", "-DFOREGROUND"]
[user@docker1 ~]$ cat index.html
J'adore les chats
[user@docker1 ~]$ cat apache2.conf
ServerName 127.0.0.1
Listen 80
LoadModule mpm_event_module "/usr/lib/apache2/modules/mod_mpm_event.so"
#LoadModule mime_module "/usr/lib/apache2/modules/mod_mime.so"
LoadModule dir_module "/usr/lib/apache2/modules/mod_dir.so"
LoadModule authz_core_module "/usr/lib/apache2/modules/mod_authz_core.so"
DirectoryIndex index.html
DocumentRoot "/var/www/html/"
ErrorLog "logs/error.log"
LogLevel warn
```

```
docker run -d -p 8888:80 my_own_apache apache2ctl -DFOREGROUND
```

📁 **`Dockerfile`**

![Waiting for Docker](./pics/waiting_for_docker.jpg)

# III. `docker-compose`

## 1. Intro

➜ **Installer `docker-compose` sur la machine**

- en suivant [la doc officielle](https://docs.docker.com/compose/install/)

`docker-compose` est un outil qui permet de lancer plusieurs conteneurs en une seule commande.

> En plus d'être pratique, il fournit des fonctionnalités additionnelles, liés au fait qu'il s'occupe à lui tout seul de lancer tous les conteneurs. On peut par exemple demander à un conteneur de ne s'allumer que lorsqu'un autre conteneur est devenu "healthy". Idéal pour lancer une application après sa base de données par exemple.

Le principe de fonctionnement de `docker-compose` :

- on écrit un fichier qui décrit les conteneurs voulus
  - c'est le `docker-compose.yml`
  - tout ce que vous écriviez sur la ligne `docker run` peut être écrit sous la forme d'un `docker-compose.yml`
- on se déplace dans le dossier qui contient le `docker-compose.yml`
- on peut utiliser les commandes `docker-compose` :

```bash
# Allumer les conteneurs définis dans le docker-compose.yml
$ docker-compose up
$ docker-compose up -d

# Eteindre
$ docker-compose down

# Explorer un peu le help, il y a d'autres commandes utiles
$ docker-compose --help
```

La syntaxe du fichier peut par exemple ressembler à :

```yml
version: "3.8"

services:
  db:
    image: mysql:5.7
    restart: always
    ports:
      - '3306:3306'
    volumes:
      - "./db/mysql_files:/var/lib/mysql"
    environment:
      MYSQL_ROOT_PASSWORD: beep
      MYSQL_DATABASE: bip
      MYSQL_USER: bap
      MYSQL_PASSWORD: boop

  nginx:
    image: nginx
    ports:
      - "80:80"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf:ro
    restart: unless-stopped
```

> Pour connaître les variables d'environnement qu'on peut passer à un conteneur, comme `MYSQL_ROOT_PASSWORD` au dessus, il faut se rendre sur la doc de l'image en question, sur le Docker Hub par exemple.

## 2. Make your own meow

Pour cette partie, vous utiliserez une application à vous que vous avez sous la main.

N'importe quelle app fera le taff, un truc dév en cours, en temps perso, au taff, peu importe.

Peu importe le langage aussi ! Go, Python, PHP (désolé des gros mots), Node (j'ai déjà dit désolé pour les gros mots ?), ou autres.

🌞 **Conteneurisez votre application**

- créer un `Dockerfile` maison qui porte l'application
- créer un `docker-compose.yml` qui permet de lancer votre application
- vous préciserez dans le rendu les instructions pour lancer l'application
  - indiquer la commande `git clone`
  - le `cd` dans le bon dossier
  - la commande `docker build` pour build l'image
  - la commande `docker-compose` pour lancer le(s) conteneur(s)

📁 📁 `app/Dockerfile` et `app/docker-compose.yml`. Je veux un sous-dossier `app/` sur votre dépôt git avec ces deux fichiers dedans :)