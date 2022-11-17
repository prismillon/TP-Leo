# TP2 : Gestion de service

Dans ce TP on va s'orienter sur l'**utilisation des systèmes GNU/Linux** comme un outil pour **faire tourner des services**. C'est le principal travail de l'administrateur système : fournir des services.

Ces services, on fait toujours la même chose avec :

- **installation** (opération ponctuelle)
- **configuration** (opération ponctuelle)
- **maintien en condition opérationnelle** (opération continue, tant que le service est actif)
- **renforcement niveau sécurité** (opération ponctuelle et continue : on conf robuste et on se tient à jour)

**Dans cette première partie, on va voir la partie installation et configuration.** Peu importe l'outil visé, de la base de données au serveur cache, en passant par le serveur web, le serveur mail, le serveur DNS, ou le serveur privé de ton meilleur jeu en ligne, c'est toujours pareil : install into conf.

On abordera la sécurité et le maintien en condition opérationelle dans une deuxième partie.

**On va apprendre à maîtriser un peu ces étapes, et pas simplement suivre la doc.**

On va maîtriser le service fourni :

- manipulation du service avec systemd
- quelle IP et quel port il utilise
- quels utilisateurs du système sont mobilisés
- quels processus sont actifs sur la machine pour que le service soit actif
- gestion des fichiers qui concernent le service et des permissions associées
- gestion avancée de la configuration du service

---

Bon le service qu'on va setup c'est NextCloud. **JE SAIS** ça fait redite avec l'an dernier, me tapez pas. ME TAPEZ PAS.  

Mais vous inquiétez pas, on va pousser le truc, on va faire évoluer l'install, l'architecture de la solution. Cette première partie de TP, on réalise une install basique, simple, simple, basique, la version *vanilla* un peu. Ce que vous êtes censés commencer à maîtriser (un peu, faites moi plais).

Refaire une install guidée, ça permet de s'exercer à faire ça proprement dans un cadre, bien comprendre, et ça me fait un pont pour les B1C aussi :)

On va faire évoluer la solution dans la suite de ce TP.

# Sommaire

- [TP2 : Gestion de service](#tp2--gestion-de-service)
- [Sommaire](#sommaire)
- [0. Prérequis](#0-prérequis)
  - [Checklist](#checklist)
- [I. Un premier serveur web](#i-un-premier-serveur-web)
  - [1. Installation](#1-installation)
  - [2. Avancer vers la maîtrise du service](#2-avancer-vers-la-maîtrise-du-service)
- [II. Une stack web plus avancée](#ii-une-stack-web-plus-avancée)
  - [1. Intro blabla](#1-intro-blabla)
  - [2. Setup](#2-setup)
    - [A. Base de données](#a-base-de-données)
    - [B. Serveur Web et NextCloud](#b-serveur-web-et-nextcloud)
    - [C. Finaliser l'installation de NextCloud](#c-finaliser-linstallation-de-nextcloud)

# 0. Prérequis

➜ Machines Rocky Linux

➜ Un unique host-only côté VBox, ça suffira. **L'adresse du réseau host-only sera `10.102.1.0/24`.**

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

![Checklist](./pics/checklist_is_here.jpg)

# I. Un premier serveur web

## 1. Installation

🖥️ **VM web.tp2.linux**

| Machine         | IP            | Service     |
|-----------------|---------------|-------------|
| `web.tp2.linux` | `10.102.1.11` | Serveur Web |

🐈‍⬛ Jui sur macos arm64 :( 

🌞 **Installer le serveur Apache**

- paquet `httpd`
- la conf se trouve dans `/etc/httpd/`
  - le fichier de conf principal est `/etc/httpd/conf/httpd.conf`
  - je vous conseille **vivement** de virer tous les commentaire du fichier, à défaut de les lire, vous y verrez plus clair
    - avec `vim` vous pouvez tout virer avec `:g/^ *#.*/d`

```
[user@web ~]$ sudo dnf install httpd
[sudo] password for user:
Rocky Linux 9 - BaseOS                                              4.6 kB/s | 3.6 kB     00:00
Rocky Linux 9 - BaseOS                                              751 kB/s | 1.3 MB     00:01
Rocky Linux 9 - AppStream                                           4.9 kB/s | 3.6 kB     00:00
Rocky Linux 9 - AppStream                                           2.3 MB/s | 4.9 MB     00:02
Rocky Linux 9 - Extras                                              5.7 kB/s | 2.9 kB     00:00
Rocky Linux 9 - Extras                                              8.0 kB/s | 7.2 kB     00:00
Dependencies resolved.
====================================================================================================
 Package                     Architecture      Version                    Repository           Size
====================================================================================================
Installing:
 httpd                       aarch64           2.4.51-7.el9_0             appstream           1.4 M
Installing dependencies:
 apr                         aarch64           1.7.0-11.el9               appstream           119 k
 apr-util                    aarch64           1.6.1-20.el9               appstream            96 k
 apr-util-bdb                aarch64           1.6.1-20.el9               appstream            13 k
 httpd-filesystem            noarch            2.4.51-7.el9_0             appstream            14 k
 httpd-tools                 aarch64           2.4.51-7.el9_0             appstream            80 k
 rocky-logos-httpd           noarch            90.11-1.el9                appstream            24 k
Installing weak dependencies:
 apr-util-openssl            aarch64           1.6.1-20.el9               appstream            15 k
 mod_http2                   aarch64           1.15.19-2.el9              appstream           144 k
 mod_lua                     aarch64           2.4.51-7.el9_0             appstream            59 k

Transaction Summary
====================================================================================================
Install  10 Packages

Total download size: 1.9 M
Installed size: 11 M
Is this ok [y/N]: y
Downloading Packages:
(1/10): rocky-logos-httpd-90.11-1.el9.noarch.rpm                    116 kB/s |  24 kB     00:00
(2/10): httpd-filesystem-2.4.51-7.el9_0.noarch.rpm                  108 kB/s |  14 kB     00:00
(3/10): httpd-tools-2.4.51-7.el9_0.aarch64.rpm                      229 kB/s |  80 kB     00:00
(4/10): mod_lua-2.4.51-7.el9_0.aarch64.rpm                          164 kB/s |  59 kB     00:00
(5/10): apr-util-openssl-1.6.1-20.el9.aarch64.rpm                   218 kB/s |  15 kB     00:00
(6/10): apr-util-bdb-1.6.1-20.el9.aarch64.rpm                       204 kB/s |  13 kB     00:00
(7/10): apr-util-1.6.1-20.el9.aarch64.rpm                           941 kB/s |  96 kB     00:00
(8/10): mod_http2-1.15.19-2.el9.aarch64.rpm                         1.3 MB/s | 144 kB     00:00
(9/10): apr-1.7.0-11.el9.aarch64.rpm                                1.2 MB/s | 119 kB     00:00
(10/10): httpd-2.4.51-7.el9_0.aarch64.rpm                           2.4 MB/s | 1.4 MB     00:00
----------------------------------------------------------------------------------------------------
Total                                                               1.4 MB/s | 1.9 MB     00:01
Running transaction check
Transaction check succeeded.
Running transaction test
Transaction test succeeded.
Running transaction
  Preparing        :                                                                            1/1
  Installing       : apr-1.7.0-11.el9.aarch64                                                  1/10
  Installing       : apr-util-bdb-1.6.1-20.el9.aarch64                                         2/10
  Installing       : apr-util-1.6.1-20.el9.aarch64                                             3/10
  Installing       : apr-util-openssl-1.6.1-20.el9.aarch64                                     4/10
  Installing       : httpd-tools-2.4.51-7.el9_0.aarch64                                        5/10
  Running scriptlet: httpd-filesystem-2.4.51-7.el9_0.noarch                                    6/10
useradd warning: apache's uid 48 outside of the SYS_UID_MIN 201 and SYS_UID_MAX 999 range.

  Installing       : httpd-filesystem-2.4.51-7.el9_0.noarch                                    6/10
  Installing       : rocky-logos-httpd-90.11-1.el9.noarch                                      7/10
  Installing       : mod_lua-2.4.51-7.el9_0.aarch64                                            8/10
  Installing       : mod_http2-1.15.19-2.el9.aarch64                                           9/10
  Installing       : httpd-2.4.51-7.el9_0.aarch64                                             10/10
  Running scriptlet: httpd-2.4.51-7.el9_0.aarch64                                             10/10
  Verifying        : rocky-logos-httpd-90.11-1.el9.noarch                                      1/10
  Verifying        : mod_lua-2.4.51-7.el9_0.aarch64                                            2/10
  Verifying        : httpd-tools-2.4.51-7.el9_0.aarch64                                        3/10
  Verifying        : httpd-filesystem-2.4.51-7.el9_0.noarch                                    4/10
  Verifying        : httpd-2.4.51-7.el9_0.aarch64                                              5/10
  Verifying        : apr-util-openssl-1.6.1-20.el9.aarch64                                     6/10
  Verifying        : apr-util-bdb-1.6.1-20.el9.aarch64                                         7/10
  Verifying        : apr-util-1.6.1-20.el9.aarch64                                             8/10
  Verifying        : mod_http2-1.15.19-2.el9.aarch64                                           9/10
  Verifying        : apr-1.7.0-11.el9.aarch64                                                 10/10

Installed:
  apr-1.7.0-11.el9.aarch64                       apr-util-1.6.1-20.el9.aarch64
  apr-util-bdb-1.6.1-20.el9.aarch64              apr-util-openssl-1.6.1-20.el9.aarch64
  httpd-2.4.51-7.el9_0.aarch64                   httpd-filesystem-2.4.51-7.el9_0.noarch
  httpd-tools-2.4.51-7.el9_0.aarch64             mod_http2-1.15.19-2.el9.aarch64
  mod_lua-2.4.51-7.el9_0.aarch64                 rocky-logos-httpd-90.11-1.el9.noarch

Complete!
```

> Ce que j'entends au-dessus par "fichier de conf principal" c'est que c'est **LE SEUL** fichier de conf lu par Apache quand il démarre. C'est souvent comme ça : un service ne lit qu'un unique fichier de conf pour démarrer. Cherchez pas, on va toujours au plus simple. Un seul fichier, c'est simple.  
**En revanche** ce serait le bordel si on mettait toute la conf dans un seul fichier pour pas mal de services.  
Donc, le principe, c'est que ce "fichier de conf principal" définit généralement deux choses. D'une part la conf globale. D'autre part, il inclut d'autres fichiers de confs plus spécifiques.  
On a le meilleur des deux mondes : simplicité (un seul fichier lu au démarrage) et la propreté (éclater la conf dans plusieurs fichiers).

🌞 **Démarrer le service Apache**

- le service s'appelle `httpd` (raccourci pour `httpd.service` en réalité)
  - démarrez le
  - faites en sorte qu'Apache démarre automatique au démarrage de la machine
  - ouvrez le port firewall nécessaire
    - utiliser une commande `ss` pour savoir sur quel port tourne actuellement Apache
    - [une petite portion du mémo est consacrée à `ss`](https://gitlab.com/it4lik/b2-linux-2021/-/blob/main/cours/memo/commandes.md#r%C3%A9seau)

```
[user@web ~]$ sudo systemctl start httpd
[user@web ~]$ sudo systemctl status httpd
● httpd.service - The Apache HTTP Server
     Loaded: loaded (/usr/lib/systemd/system/httpd.service; disabled; vendor preset: disabled)
     Active: active (running) since Tue 2022-11-15 10:00:19 CET; 4s ago
       Docs: man:httpd.service(8)
   Main PID: 1203 (httpd)
     Status: "Started, listening on: port 80"
      Tasks: 213 (limit: 4268)
     Memory: 22.8M
        CPU: 62ms
     CGroup: /system.slice/httpd.service
             ├─1203 /usr/sbin/httpd -DFOREGROUND
             ├─1204 /usr/sbin/httpd -DFOREGROUND
             ├─1205 /usr/sbin/httpd -DFOREGROUND
             ├─1206 /usr/sbin/httpd -DFOREGROUND
             └─1207 /usr/sbin/httpd -DFOREGROUND

Nov 15 10:00:19 web.tp2.linux systemd[1]: Starting The Apache HTTP Server...
Nov 15 10:00:19 web.tp2.linux systemd[1]: Started The Apache HTTP Server.
Nov 15 10:00:19 web.tp2.linux httpd[1203]: Server configured, listening on: port 80
[user@web ~]$ sudo systemctl enable httpd
Created symlink /etc/systemd/system/multi-user.target.wants/httpd.service → /usr/lib/systemd/system/httpd.service.
```

```
[user@web ~]$ sudo !!
sudo ss -lntp
State        Recv-Q       Send-Q             Local Address:Port             Peer Address:Port       Process
LISTEN       0            128                      0.0.0.0:22                    0.0.0.0:*           users:(("sshd",pid=649,fd=3))
LISTEN       0            511                            *:80                          *:*           users:(("httpd",pid=1207,fd=4),("httpd",pid=1206,fd=4),("httpd",pid=1205,fd=4),("httpd",pid=1203,fd=4))
LISTEN       0            128                         [::]:22                       [::]:*           users:(("sshd",pid=649,fd=4))
```

apache est sur le port 80

```
[user@web ~]$ sudo firewall-cmd --add-port=80/tcp --permanent
success
[user@web ~]$ sudo firewall-cmd --reload
success
```

**En cas de problème** (IN CASE OF FIIIIRE) vous pouvez check les logs d'Apache :

```bash
# Demander à systemd les logs relatifs au service httpd
$ sudo journalctl -xe -u httpd

# Consulter le fichier de logs d'erreur d'Apache
$ sudo cat /var/log/httpd/error_log

# Il existe aussi un fichier de log qui enregistre toutes les requêtes effectuées sur votre serveur
$ sudo cat /var/log/httpd/access_log
```

🌞 **TEST**

- vérifier que le service est démarré
- vérifier qu'il est configuré pour démarrer automatiquement
- vérifier avec une commande `curl localhost` que vous joignez votre serveur web localement
- vérifier avec votre navigateur (sur votre PC) que vous accéder à votre serveur web

```
[user@web ~]$ curl localhost
<!doctype html>
<html>
  <head>
    <meta charset='utf-8'>
    <meta name='viewport' content='width=device-width, initial-scale=1'>
    <title>HTTP Server Test Page powered by: Rocky Linux</title>
    <style type="text/css">
      /*<![CDATA[*/

      html {
        height: 100%;
        width: 100%;
      }
        body {
  background: rgb(20,72,50);
  background: -moz-linear-gradient(180deg, rgba(23,43,70,1) 30%, rgba(0,0,0,1) 90%)  ;
  background: -webkit-linear-gradient(180deg, rgba(23,43,70,1) 30%, rgba(0,0,0,1) 90%) ;
  background: linear-gradient(180deg, rgba(23,43,70,1) 30%, rgba(0,0,0,1) 90%);
  background-repeat: no-repeat;
  background-attachment: fixed;
  filter: progid:DXImageTransform.Microsoft.gradient(startColorstr="#3c6eb4",endColorstr="#3c95b4",GradientType=1);
        color: white;
        font-size: 0.9em;
        font-weight: 400;
        font-family: 'Montserrat', sans-serif;
        margin: 0;
        padding: 10em 6em 10em 6em;
        box-sizing: border-box;

      }


  h1 {
    text-align: center;
    margin: 0;
    padding: 0.6em 2em 0.4em;
    color: #fff;
    font-weight: bold;
    font-family: 'Montserrat', sans-serif;
    font-size: 2em;
  }
  h1 strong {
    font-weight: bolder;
    font-family: 'Montserrat', sans-serif;
  }
  h2 {
    font-size: 1.5em;
    font-weight:bold;
  }

  .title {
    border: 1px solid black;
    font-weight: bold;
    position: relative;
    float: right;
    width: 150px;
    text-align: center;
    padding: 10px 0 10px 0;
    margin-top: 0;
  }

  .description {
    padding: 45px 10px 5px 10px;
    clear: right;
    padding: 15px;
  }

  .section {
    padding-left: 3%;
   margin-bottom: 10px;
  }

  img {

    padding: 2px;
    margin: 2px;
  }
  a:hover img {
    padding: 2px;
    margin: 2px;
  }

  :link {
    color: rgb(199, 252, 77);
    text-shadow:
  }
  :visited {
    color: rgb(122, 206, 255);
  }
  a:hover {
    color: rgb(16, 44, 122);
  }
  .row {
    width: 100%;
    padding: 0 10px 0 10px;
  }

  footer {
    padding-top: 6em;
    margin-bottom: 6em;
    text-align: center;
    font-size: xx-small;
    overflow:hidden;
    clear: both;
  }

  .summary {
    font-size: 140%;
    text-align: center;
  }

  #rocky-poweredby img {
    margin-left: -10px;
  }

  #logos img {
    vertical-align: top;
  }

  /* Desktop  View Options */

  @media (min-width: 768px)  {

    body {
      padding: 10em 20% !important;
    }

    .col-md-1, .col-md-2, .col-md-3, .col-md-4, .col-md-5, .col-md-6,
    .col-md-7, .col-md-8, .col-md-9, .col-md-10, .col-md-11, .col-md-12 {
      float: left;
    }

    .col-md-1 {
      width: 8.33%;
    }
    .col-md-2 {
      width: 16.66%;
    }
    .col-md-3 {
      width: 25%;
    }
    .col-md-4 {
      width: 33%;
    }
    .col-md-5 {
      width: 41.66%;
    }
    .col-md-6 {
      border-left:3px ;
      width: 50%;


    }
    .col-md-7 {
      width: 58.33%;
    }
    .col-md-8 {
      width: 66.66%;
    }
    .col-md-9 {
      width: 74.99%;
    }
    .col-md-10 {
      width: 83.33%;
    }
    .col-md-11 {
      width: 91.66%;
    }
    .col-md-12 {
      width: 100%;
    }
  }

  /* Mobile View Options */
  @media (max-width: 767px) {
    .col-sm-1, .col-sm-2, .col-sm-3, .col-sm-4, .col-sm-5, .col-sm-6,
    .col-sm-7, .col-sm-8, .col-sm-9, .col-sm-10, .col-sm-11, .col-sm-12 {
      float: left;
    }

    .col-sm-1 {
      width: 8.33%;
    }
    .col-sm-2 {
      width: 16.66%;
    }
    .col-sm-3 {
      width: 25%;
    }
    .col-sm-4 {
      width: 33%;
    }
    .col-sm-5 {
      width: 41.66%;
    }
    .col-sm-6 {
      width: 50%;
    }
    .col-sm-7 {
      width: 58.33%;
    }
    .col-sm-8 {
      width: 66.66%;
    }
    .col-sm-9 {
      width: 74.99%;
    }
    .col-sm-10 {
      width: 83.33%;
    }
    .col-sm-11 {
      width: 91.66%;
    }
    .col-sm-12 {
      width: 100%;
    }
    h1 {
      padding: 0 !important;
    }
  }


  </style>
  </head>
  <body>
    <h1>HTTP Server <strong>Test Page</strong></h1>

    <div class='row'>

      <div class='col-sm-12 col-md-6 col-md-6 '></div>
          <p class="summary">This page is used to test the proper operation of
            an HTTP server after it has been installed on a Rocky Linux system.
            If you can read this page, it means that the software it working
            correctly.</p>
      </div>

      <div class='col-sm-12 col-md-6 col-md-6 col-md-offset-12'>


        <div class='section'>
          <h2>Just visiting?</h2>

          <p>This website you are visiting is either experiencing problems or
          could be going through maintenance.</p>

          <p>If you would like the let the administrators of this website know
          that you've seen this page instead of the page you've expected, you
          should send them an email. In general, mail sent to the name
          "webmaster" and directed to the website's domain should reach the
          appropriate person.</p>

          <p>The most common email address to send to is:
          <strong>"webmaster@example.com"</strong></p>

          <h2>Note:</h2>
          <p>The Rocky Linux distribution is a stable and reproduceable platform
          based on the sources of Red Hat Enterprise Linux (RHEL). With this in
          mind, please understand that:

        <ul>
          <li>Neither the <strong>Rocky Linux Project</strong> nor the
          <strong>Rocky Enterprise Software Foundation</strong> have anything to
          do with this website or its content.</li>
          <li>The Rocky Linux Project nor the <strong>RESF</strong> have
          "hacked" this webserver: This test page is included with the
          distribution.</li>
        </ul>
        <p>For more information about Rocky Linux, please visit the
          <a href="https://rockylinux.org/"><strong>Rocky Linux
          website</strong></a>.
        </p>
        </div>
      </div>
      <div class='col-sm-12 col-md-6 col-md-6 col-md-offset-12'>
        <div class='section'>

          <h2>I am the admin, what do I do?</h2>

        <p>You may now add content to the webroot directory for your
        software.</p>

        <p><strong>For systems using the
        <a href="https://httpd.apache.org/">Apache Webserver</strong></a>:
        You can add content to the directory <code>/var/www/html/</code>.
        Until you do so, people visiting your website will see this page. If
        you would like this page to not be shown, follow the instructions in:
        <code>/etc/httpd/conf.d/welcome.conf</code>.</p>

        <p><strong>For systems using
        <a href="https://nginx.org">Nginx</strong></a>:
        You can add your content in a location of your
        choice and edit the <code>root</code> configuration directive
        in <code>/etc/nginx/nginx.conf</code>.</p>

        <div id="logos">
          <a href="https://rockylinux.org/" id="rocky-poweredby"><img src="icons/poweredby.png" alt="[ Powered by Rocky Linux ]" /></a> <!-- Rocky -->
          <img src="poweredby.png" /> <!-- webserver -->
        </div>
      </div>
      </div>

      <footer class="col-sm-12">
      <a href="https://apache.org">Apache&trade;</a> is a registered trademark of <a href="https://apache.org">the Apache Software Foundation</a> in the United States and/or other countries.<br />
      <a href="https://nginx.org">NGINX&trade;</a> is a registered trademark of <a href="https://">F5 Networks, Inc.</a>.
      </footer>

  </body>
</html>
```

```
curl 192.168.64.6
<!doctype html>
<html>
  <head>
    <meta charset='utf-8'>
    <meta name='viewport' content='width=device-width, initial-scale=1'>
...
```

## 2. Avancer vers la maîtrise du service

🌞 **Le service Apache...**

- affichez le contenu du fichier `httpd.service` qui contient la définition du service Apache

```
[user@web ~]$ cat /etc/systemd/system/multi-user.target.wants/httpd.service
# See httpd.service(8) for more information on using the httpd service.

# Modifying this file in-place is not recommended, because changes
# will be overwritten during package upgrades.  To customize the
# behaviour, run "systemctl edit httpd" to create an override unit.

# For example, to pass additional options (such as -D definitions) to
# the httpd binary at startup, create an override unit (as is done by
# systemctl edit) and enter the following:

#	[Service]
#	Environment=OPTIONS=-DMY_DEFINE

[Unit]
Description=The Apache HTTP Server
Wants=httpd-init.service
After=network.target remote-fs.target nss-lookup.target httpd-init.service
Documentation=man:httpd.service(8)

[Service]
Type=notify
Environment=LANG=C

ExecStart=/usr/sbin/httpd $OPTIONS -DFOREGROUND
ExecReload=/usr/sbin/httpd $OPTIONS -k graceful
# Send SIGWINCH for graceful stop
KillSignal=SIGWINCH
KillMode=mixed
PrivateTmp=true
OOMPolicy=continue

[Install]
WantedBy=multi-user.target
```

🌞 **Déterminer sous quel utilisateur tourne le processus Apache**

- mettez en évidence la ligne dans le fichier de conf principal d'Apache (`httpd.conf`) qui définit quel user est utilisé
- utilisez la commande `ps -ef` pour visualiser les processus en cours d'exécution et confirmer que apache tourne bien sous l'utilisateur mentionné dans le fichier de conf
- la page d'accueil d'Apache se trouve dans `/usr/share/testpage/`
  - vérifiez avec un `ls -al` que tout son contenu est **accessible en lecture** à l'utilisateur mentionné dans le fichier de conf

```
User apache
Group apache
```

```
[user@web ~]$ ps -ef | grep apache
apache       754     727  0 10:09 ?        00:00:00 /usr/sbin/httpd -DFOREGROUND
apache       756     727  0 10:09 ?        00:00:00 /usr/sbin/httpd -DFOREGROUND
apache       757     727  0 10:09 ?        00:00:00 /usr/sbin/httpd -DFOREGROUND
apache       758     727  0 10:09 ?        00:00:00 /usr/sbin/httpd -DFOREGROUND
user        1045     984  0 10:27 pts/0    00:00:00 grep --color=auto apache
```

```
[user@web ~]$ ls -al /usr/share/testpage/
total 12
drwxr-xr-x.   2 root root   24 Nov 15 09:54 .
drwxr-xr-x. 100 root root 4096 Nov 15 09:54 ..
-rw-r--r--.   1 root root 7620 Jul  6 04:37 index.html
```

tout le monde peut le lire

🌞 **Changer l'utilisateur utilisé par Apache**

- créez un nouvel utilisateur
  - pour les options de création, inspirez-vous de l'utilisateur Apache existant
    - le fichier `/etc/passwd` contient les informations relatives aux utilisateurs existants sur la machine
    - servez-vous en pour voir la config actuelle de l'utilisateur Apache par défaut
- modifiez la configuration d'Apache pour qu'il utilise ce nouvel utilisateur
- redémarrez Apache
- utilisez une commande `ps` pour vérifier que le changement a pris effet

```
[user@web ~]$ sudo useradd toto -s /sbin/nologin -d /usr/share/httpd
[sudo] password for user:
useradd: warning: the home directory /usr/share/httpd already exists.
useradd: Not copying any file from skel directory into it.
```

```
[user@web ~]$ sudo vim /etc/httpd/conf/httpd.conf
[user@web ~]$ sudo systemctl restart httpd
[user@web ~]$ ps -ef | grep httpd
root        1075       1  0 10:40 ?        00:00:00 /usr/sbin/httpd -DFOREGROUND
toto        1076    1075  0 10:40 ?        00:00:00 /usr/sbin/httpd -DFOREGROUND
toto        1077    1075  0 10:40 ?        00:00:00 /usr/sbin/httpd -DFOREGROUND
toto        1078    1075  0 10:40 ?        00:00:00 /usr/sbin/httpd -DFOREGROUND
toto        1079    1075  0 10:40 ?        00:00:00 /usr/sbin/httpd -DFOREGROUND
user        1292     984  0 10:40 pts/0    00:00:00 grep --color=auto httpd
```


🌞 **Faites en sorte que Apache tourne sur un autre port**

- modifiez la configuration d'Apache pour lui demander d'écouter sur un autre port de votre choix
- ouvrez ce nouveau port dans le firewall, et fermez l'ancien
- redémarrez Apache
- prouvez avec une commande `ss` que Apache tourne bien sur le nouveau port choisi
- vérifiez avec `curl` en local que vous pouvez joindre Apache sur le nouveau port
- vérifiez avec votre navigateur que vous pouvez joindre le serveur sur le nouveau port

```
[user@web ~]$ sudo vim /etc/httpd/conf/httpd.conf
[user@web ~]$ sudo firewall-cmd --remove-port=80/tcp --permanent
success
[user@web ~]$ sudo firewall-cmd --add-port=8080/tcp --permanent
success
[user@web ~]$ sudo firewall-cmd --reload
success
```

```
[user@web ~]$ sudo systemctl restart httpd
[user@web ~]$ sudo ss -lntp
State        Recv-Q       Send-Q             Local Address:Port             Peer Address:Port       Process
LISTEN       0            128                      0.0.0.0:22                    0.0.0.0:*           users:(("sshd",pid=648,fd=3))
LISTEN       0            511                            *:8080                        *:*           users:(("httpd",pid=1334,fd=4),("httpd",pid=1333,fd=4),("httpd",pid=1332,fd=4),("httpd",pid=1329,fd=4))
LISTEN       0            128                         [::]:22                       [::]:*           users:(("sshd",pid=648,fd=4))
```

```
[user@web ~]$ curl localhost:8080
<!doctype html>
<html>
  <head>
    <meta charset='utf-8'>
    <meta name='viewport' content='width=device-width, initial-scale=1'>
    <title>HTTP Server Test Page powered by: Rocky Linux</title>
    <style type="text/css">
      /*<![CDATA[*/

      html {
        height: 100%;
        width: 100%;
      }
...
```

```
~
❯ curl 192.168.64.6:8080
<!doctype html>
<html>
  <head>
    <meta charset='utf-8'>
    <meta name='viewport' content='width=device-width, initial-scale=1'>
    <title>HTTP Server Test Page powered by: Rocky Linux</title>
    <style type="text/css">
      /*<![CDATA[*/
...
```

📁 **Fichier `/etc/httpd/conf/httpd.conf`**

# II. Une stack web plus avancée

⚠⚠⚠ **Réinitialiser votre conf Apache avant de continuer** ⚠⚠⚠  
En particulier :

- reprendre le port par défaut
- reprendre l'utilisateur par défaut

## 1. Intro blabla

**Le serveur web `web.tp2.linux` sera le serveur qui accueillera les clients.** C'est sur son IP que les clients devront aller pour visiter le site web.  

**Le service de base de données `db.tp2.linux` sera uniquement accessible depuis `web.tp2.linux`.** Les clients ne pourront pas y accéder. Le serveur de base de données stocke les infos nécessaires au serveur web, pour le bon fonctionnement du site web.

---

Bon le but de ce TP est juste de s'exercer à faire tourner des services, un serveur + sa base de données, c'est un peu le cas d'école. J'ai pas envie d'aller deep dans la conf de l'un ou de l'autre avec vous pour le moment, on va se contenter d'une conf minimale.

Je vais pas vous demander de coder une application, et cette fois on se contentera pas d'un simple `index.html` tout moche et on va se mettre dans la peau de l'admin qui se retrouve avec une application à faire tourner. **On va faire tourner un [NextCloud](https://nextcloud.com/).**

En plus c'est utile comme truc : c'est un p'tit serveur pour héberger ses fichiers via une WebUI, style Google Drive. Mais on l'héberge nous-mêmes :)

---

Le flow va être le suivant :

➜ **on prépare d'abord la base de données**, avant de setup NextCloud

- comme ça il aura plus qu'à s'y connecter
- ce sera sur une nouvelle machine `db.tp2.linux`
- il faudra installer le service de base de données, puis lancer le service
- on pourra alors créer, au sein du service de base de données, le nécessaire pour NextCloud

➜ **ensuite on met en place NextCloud**

- on réutilise la machine précédente avec Apache déjà installé, ce sera toujours Apache qui accueillera les requêtes des clients
- mais plutôt que de retourner une bête page HTML, NextCloud traitera la requête
- NextCloud, c'est codé en PHP, il faudra donc **installer une version de PHP précise** sur la machine
- on va donc : install PHP, configurer Apache, récupérer un `.zip` de NextCloud, et l'extraire au bon endroit !

![NextCloud install](./pics/nc_install.png)

## 2. Setup

🖥️ **VM db.tp2.linux**

**N'oubliez pas de dérouler la [📝**checklist**📝](#checklist).**

| Machines        | IP            | Service                 |
|-----------------|---------------|-------------------------|
| `web.tp2.linux` | `10.102.1.11` | Serveur Web             |
| `db.tp2.linux`  | `10.102.1.12` | Serveur Base de Données |

### A. Base de données

🌞 **Install de MariaDB sur `db.tp2.linux`**

- déroulez [la doc d'install de Rocky](https://docs.rockylinux.org/guides/database/database_mariadb-server/)
- je veux dans le rendu **toutes** les commandes réalisées
- vous repérerez le port utilisé par MariaDB avec une commande `ss` exécutée sur `db.tp2.linux`
  - il sera nécessaire de l'ouvrir dans le firewall

```
[user@localhost ~]$ sudo !!
sudo dnf install mariadb-server
[sudo] password for user:
Rocky Linux 9 -  933  B/s | 3.6 kB     00:03
Rocky Linux 9 -  1.0 MB/s | 1.3 MB     00:01
Rocky Linux 9 -  3.0 kB/s | 3.6 kB     00:01
Rocky Linux 9 -  2.3 MB/s | 4.9 MB     00:02
Rocky Linux 9 -  6.4 kB/s | 2.9 kB     00:00
Rocky Linux 9 -  6.3 kB/s | 7.2 kB     00:01
Dependencies resolved.
...
Complete!
```

```
[user@localhost ~]$ systemctl enable mariadb
==== AUTHENTICATING FOR org.freedesktop.systemd1.manage-unit-files ====
Authentication is required to manage system service or unit files.
Authenticating as: user
Password:
==== AUTHENTICATION COMPLETE ====
Created symlink /etc/systemd/system/mysql.service → /usr/lib/systemd/system/mariadb.service.
Created symlink /etc/systemd/system/mysqld.service → /usr/lib/systemd/system/mariadb.service.
Created symlink /etc/systemd/system/multi-user.target.wants/mariadb.service → /usr/lib/systemd/system/mariadb.service.
==== AUTHENTICATING FOR org.freedesktop.systemd1.reload-daemon ====
Authentication is required to reload the systemd state.
Authenticating as: user
Password:
==== AUTHENTICATION COMPLETE ====
```

```
[user@localhost ~]$ systemctl start mariadb
==== AUTHENTICATING FOR org.freedesktop.systemd1.manage-units ====
Authentication is required to start 'mariadb.service'.
Authenticating as: user
Password:
==== AUTHENTICATION COMPLETE ====
```

```
[user@localhost ~]$ sudo !!
sudo mysql_secure_installation

NOTE: RUNNING ALL PARTS OF THIS SCRIPT IS RECOMMENDED FOR ALL MariaDB
      SERVERS IN PRODUCTION USE!  PLEASE READ EACH STEP CAREFULLY!

In order to log into MariaDB to secure it, we'll need the current
password for the root user. If you've just installed MariaDB, and
haven't set the root password yet, you should just press enter here.

Enter current password for root (enter for none):
OK, successfully used password, moving on...

Setting the root password or using the unix_socket ensures that nobody
can log into the MariaDB root user without the proper authorisation.

You already have your root account protected, so you can safely answer 'n'.

Switch to unix_socket authentication [Y/n] n
 ... skipping.

You already have your root account protected, so you can safely answer 'n'.

Change the root password? [Y/n] n
 ... skipping.

By default, a MariaDB installation has an anonymous user, allowing anyone
to log into MariaDB without having to have a user account created for
them.  This is intended only for testing, and to make the installation
go a bit smoother.  You should remove them before moving into a
production environment.

Remove anonymous users? [Y/n]
 ... Success!

Normally, root should only be allowed to connect from 'localhost'.  This
ensures that someone cannot guess at the root password from the network.

Disallow root login remotely? [Y/n]
 ... Success!

By default, MariaDB comes with a database named 'test' that anyone can
access.  This is also intended only for testing, and should be removed
before moving into a production environment.

Remove test database and access to it? [Y/n]
 - Dropping test database...
 ... Success!
 - Removing privileges on test database...
 ... Success!

Reloading the privilege tables will ensure that all changes made so far
will take effect immediately.

Reload privilege tables now? [Y/n]
 ... Success!

Cleaning up...

All done!  If you've completed all of the above steps, your MariaDB
installation should now be secure.

Thanks for using MariaDB!
```

> La doc vous fait exécuter la commande `mysql_secure_installation` c'est un bon réflexe pour renforcer la base qui a une configuration un peu *chillax* à l'install.

🌞 **Préparation de la base pour NextCloud**

- une fois en place, il va falloir préparer une base de données pour NextCloud :
  - connectez-vous à la base de données à l'aide de la commande `sudo mysql -u root -p`
  - exécutez les commandes SQL suivantes :

```sql
-- Création d'un utilisateur dans la base, avec un mot de passe
-- L'adresse IP correspond à l'adresse IP depuis laquelle viendra les connexions. Cela permet de restreindre les IPs autorisées à se connecter.
-- Dans notre cas, c'est l'IP de web.tp2.linux
-- "pewpewpew" c'est le mot de passe hehe
CREATE USER 'nextcloud'@'10.102.1.11' IDENTIFIED BY 'pewpewpew';

-- Création de la base de donnée qui sera utilisée par NextCloud
CREATE DATABASE IF NOT EXISTS nextcloud CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci;

-- On donne tous les droits à l'utilisateur nextcloud sur toutes les tables de la base qu'on vient de créer
GRANT ALL PRIVILEGES ON nextcloud.* TO 'nextcloud'@'10.102.1.11';

-- Actualisation des privilèges
FLUSH PRIVILEGES;

-- C'est assez générique comme opération, on crée une base, on crée un user, on donne les droits au user sur la base
```

```sql
MariaDB [(none)]> CREATE USER 'nextcloud'@'192.168.64.6' IDENTIFIED BY 'pewpewpew';
Query OK, 0 rows affected (0.009 sec)
```

```sql
MariaDB [(none)]> CREATE DATABASE IF NOT EXISTS nextcloud CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci;
Query OK, 1 row affected (0.004 sec)
```

```sql
MariaDB [(none)]> GRANT ALL PRIVILEGES ON nextcloud.* TO 'nextcloud'@'192.168.64.6';
Query OK, 0 rows affected (0.008 sec)
```

```sql
MariaDB [(none)]> FLUSH PRIVILEGES;
Query OK, 0 rows affected (0.002 sec)
```

> Par défaut, vous avez le droit de vous connecter localement à la base si vous êtes `root`. C'est pour ça que `sudo mysql -u root` fonctionne, sans nous demander de mot de passe. Evidemment, n'importe quelles autres conditions ne permettent pas une connexion aussi facile à la base.

🌞 **Exploration de la base de données**

- afin de tester le bon fonctionnement de la base de données, vous allez essayer de vous connecter, comme NextCloud le fera :
  - depuis la machine `web.tp2.linux` vers l'IP de `db.tp2.linux`
  - utilisez la commande `mysql` pour vous connecter à une base de données depuis la ligne de commande
    - par exemple `mysql -u <USER> -h <IP_DATABASE> -p`
    - si vous ne l'avez pas, installez-là
    - vous pouvez déterminer dans quel paquet est disponible la commande `mysql` en saisissant `dnf provides mysql`
- **donc vous devez effectuer une commande `mysql` sur `web.tp2.linux`**
- une fois connecté à la base, utilisez les commandes SQL fournies ci-dessous pour explorer la base

```sql
SHOW DATABASES;
USE <DATABASE_NAME>;
SHOW TABLES;
```

```sql
[user@web ~]$ mysql -u nextcloud -h 192.168.64.7 -p
Enter password:
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 16
Server version: 5.5.5-10.5.16-MariaDB MariaDB Server

Copyright (c) 2000, 2022, Oracle and/or its affiliates.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> SHOW DATABASES;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| nextcloud          |
+--------------------+
2 rows in set (0.01 sec)

mysql> USE nextcloud
Database changed
mysql> SHOW TABLES;
Empty set (0.01 sec)
```


🌞 **Trouver une commande SQL qui permet de lister tous les utilisateurs de la base de données**

> Les utilisateurs de la base de données sont différents des utilisateurs du système Rocky Linux qui porte la base. Les utilisateurs de la base définissent des identifiants utilisés pour se connecter à la base afin d'y voir ou d'y modifier des données.

```sql
MariaDB [(none)]> SELECT * FROM mysql.user;
+--------------+-------------+-------------------------------------------+-------------+-------------+-------------+-------------+-------------+-----------+-------------+---------------+--------------+-----------+------------+-----------------+------------+------------+--------------+------------+-----------------------+------------------+--------------+-----------------+------------------+------------------+----------------+---------------------+--------------------+------------------+------------+--------------+------------------------+---------------------+----------+------------+-------------+--------------+---------------+-------------+-----------------+----------------------+-----------------------+-------------------------------------------+------------------+---------+--------------+--------------------+
| Host         | User        | Password                                  | Select_priv | Insert_priv | Update_priv | Delete_priv | Create_priv | Drop_priv | Reload_priv | Shutdown_priv | Process_priv | File_priv | Grant_priv | References_priv | Index_priv | Alter_priv | Show_db_priv | Super_priv | Create_tmp_table_priv | Lock_tables_priv | Execute_priv | Repl_slave_priv | Repl_client_priv | Create_view_priv | Show_view_priv | Create_routine_priv | Alter_routine_priv | Create_user_priv | Event_priv | Trigger_priv | Create_tablespace_priv | Delete_history_priv | ssl_type | ssl_cipher | x509_issuer | x509_subject | max_questions | max_updates | max_connections | max_user_connections | plugin                | authentication_string                     | password_expired | is_role | default_role | max_statement_time |
+--------------+-------------+-------------------------------------------+-------------+-------------+-------------+-------------+-------------+-----------+-------------+---------------+--------------+-----------+------------+-----------------+------------+------------+--------------+------------+-----------------------+------------------+--------------+-----------------+------------------+------------------+----------------+---------------------+--------------------+------------------+------------+--------------+------------------------+---------------------+----------+------------+-------------+--------------+---------------+-------------+-----------------+----------------------+-----------------------+-------------------------------------------+------------------+---------+--------------+--------------------+
| localhost    | mariadb.sys |                                           | N           | N           | N           | N           | N           | N         | N           | N             | N            | N         | N          | N               | N          | N          | N            | N          | N                     | N                | N            | N               | N                | N                | N              | N                   | N                  | N                | N          | N            | N                      | N                   |          |            |             |              |             0 |           0 |               0 |                    0 | mysql_native_password |                                           | Y                | N       |              |           0.000000 |
| localhost    | root        | invalid                                   | Y           | Y           | Y           | Y           | Y           | Y         | Y           | Y             | Y            | Y         | Y          | Y               | Y          | Y          | Y            | Y          | Y                     | Y                | Y            | Y               | Y                | Y                | Y              | Y                   | Y                  | Y                | Y          | Y            | Y                      | Y                   |          |            |             |              |             0 |           0 |               0 |                    0 | mysql_native_password | invalid                                   | N                | N       |              |           0.000000 |
| localhost    | mysql       | invalid                                   | Y           | Y           | Y           | Y           | Y           | Y         | Y           | Y             | Y            | Y         | Y          | Y               | Y          | Y          | Y            | Y          | Y                     | Y                | Y            | Y               | Y                | Y                | Y              | Y                   | Y                  | Y                | Y          | Y            | Y                      | Y                   |          |            |             |              |             0 |           0 |               0 |                    0 | mysql_native_password | invalid                                   | N                | N       |              |           0.000000 |
| 192.168.64.6 | nextcloud   | *AF136CF35F0D546F69717A7F18C18849666E64D0 | N           | N           | N           | N           | N           | N         | N           | N             | N            | N         | N          | N               | N          | N          | N            | N          | N                     | N                | N            | N               | N                | N                | N              | N                   | N                  | N                | N          | N            | N                      | N                   |          |            |             |              |             0 |           0 |               0 |                    0 | mysql_native_password | *AF136CF35F0D546F69717A7F18C18849666E64D0 | N                | N       |              |           0.000000 |
+--------------+-------------+-------------------------------------------+-------------+-------------+-------------+-------------+-------------+-----------+-------------+---------------+--------------+-----------+------------+-----------------+------------+------------+--------------+------------+-----------------------+------------------+--------------+-----------------+------------------+------------------+----------------+---------------------+--------------------+------------------+------------+--------------+------------------------+---------------------+----------+------------+-------------+--------------+---------------+-------------+-----------------+----------------------+-----------------------+-------------------------------------------+------------------+---------+--------------+--------------------+
4 rows in set (0.003 sec)
```

Une fois qu'on s'est assurés qu'on peut se co au service de base de données depuis `web.tp2.linux`, on peut continuer.

### B. Serveur Web et NextCloud

⚠️⚠️⚠️ **N'OUBLIEZ PAS de réinitialiser votre conf Apache avant de continuer. En particulier, remettez le port et le user par défaut.**

🌞 **Install de PHP**

```bash
# On ajoute le dépôt CRB
$ sudo dnf config-manager --set-enabled crb
# On ajoute le dépôt REMI
$ sudo dnf install dnf-utils http://rpms.remirepo.net/enterprise/remi-release-9.rpm -y

# On liste les versions de PHP dispos, au passage on va pouvoir accepter les clés du dépôt REMI
$ dnf module list php

# On active le dépôt REMI pour récupérer une version spécifique de PHP, celle recommandée par la doc de NextCloud
$ sudo dnf module enable php:remi-8.1 -y

# Eeeet enfin, on installe la bonne version de PHP : 8.1
$ sudo dnf install -y php81-php
```

```
sudo apt install apache2 libapache2-mod-php mariadb-server php-xml php-cli php-cgi php-mysql php-mbstring php-gd php-curl php-zip
```

🌞 **Install de tous les modules PHP nécessaires pour NextCloud**

```bash
# eeeeet euuuh boom. Là non plus j'ai pas pondu ça, c'est la doc :
$ sudo dnf install -y libxml2 openssl php81-php php81-php-ctype php81-php-curl php81-php-gd php81-php-iconv php81-php-json php81-php-libxml php81-php-mbstring php81-php-openssl php81-php-posix php81-php-session php81-php-xml php81-php-zip php81-php-zlib php81-php-pdo php81-php-mysqlnd php81-php-intl php81-php-bcmath php81-php-gmp
```

🌞 **Récupérer NextCloud**

- créez le dossier `/var/www/tp2_nextcloud/`
  - ce sera notre *racine web* (ou *webroot*)
  - l'endroit où le site est stocké quoi, on y trouvera un `index.html` et un tas d'autres marde, tout ce qui constitue NextClo :D
- récupérer le fichier suivant avec une commande `curl` ou `wget` : https://download.nextcloud.com/server/prereleases/nextcloud-25.0.0rc3.zip
- extrayez tout son contenu dans le dossier `/var/www/tp2_nextcloud/` en utilisant la commande `unzip`
  - installez la commande `unzip` si nécessaire
  - vous pouvez extraire puis déplacer ensuite, vous prenez pas la tête
  - contrôlez que le fichier `/var/www/tp2_nextcloud/index.html` existe pour vérifier que tout est en place
- assurez-vous que le dossier `/var/www/tp2_nextcloud/` et tout son contenu appartient à l'utilisateur qui exécute le service Apache

```
wget https://download.nextcloud.com/server/releases/nextcloud-25.0.1.zip
ls
unzip nextcloud-*.zip
sudo cp -r nextcloud /var/www/html/nextcloud
sudo chown -R www-data:www-data /var/www/html/nextcloud
sudo systemctl restart apache
sudo systemctl restart apache2.service
```

> A chaque fois que vous faites ce genre de trucs, assurez-vous que c'est bien ok. Par exemple, vérifiez avec un `ls -al` que tout appartient bien à l'utilisateur qui exécute Apache.

🌞 **Adapter la configuration d'Apache**

- regardez la dernière ligne du fichier de conf d'Apache pour constater qu'il existe une ligne qui inclut d'autres fichiers de conf
- créez en conséquence un fichier de configuration qui porte un nom clair et qui contient la configuration suivante :

```apache
<VirtualHost *:80>
  DocumentRoot /var/www/tp2_nextcloud/ # on indique le chemin de notre webroot
  ServerName  web.tp2.linux # on précise le nom que saisissent les clients pour accéder au service
  <Directory /var/www/tp2_nextcloud/> # on définit des règles d'accès sur notre webroot
    Require all granted
    AllowOverride All
    Options FollowSymLinks MultiViews
    <IfModule mod_dav.c>
      Dav off
    </IfModule>
  </Directory>
</VirtualHost>
```

```
user@web:/etc/apache2$ vim nextcloud.conf
user@web:/etc/apache2$ sudo !!
sudo vim nextcloud.conf
user@web:/etc/apache2$ vim apache2.conf
user@web:/etc/apache2$ sudo !!
sudo vim apache2.conf
```

🌞 **Redémarrer le service Apache** pour qu'il prenne en compte le nouveau fichier de conf

```
user@web:~$ sudo systemctl restart apache2.service
```

### C. Finaliser l'installation de NextCloud

➜ **Sur votre PC**

- modifiez votre fichier `hosts` (oui, celui de votre PC, de votre hôte)
  - pour pouvoir joindre l'IP de la VM en utilisant le nom `web.tp2.linux`
- avec un navigateur, visitez NextCloud à l'URL `http://web.tp2.linux`
  - c'est possible grâce à la modification de votre fichier `hosts`
- on va vous demander un utilisateur et un mot de passe pour créer un compte admin
  - ne saisissez rien pour le moment
- cliquez sur "Storage & Database" juste en dessous
  - choisissez "MySQL/MariaDB"
  - saisissez les informations pour que NextCloud puisse se connecter avec votre base
- saisissez l'identifiant et le mot de passe admin que vous voulez, et validez l'installation

🌴 **C'est chez vous ici**, baladez vous un peu sur l'interface de NextCloud, faites le tour du propriétaire :)

🌞 **Exploration de la base de données**

- connectez vous en ligne de commande à la base de données après l'installation terminée
- déterminer combien de tables ont été crées par NextCloud lors de la finalisation de l'installation
  - ***bonus points*** si la réponse à cette question est automatiquement donnée par une requête SQL

```
MariaDB [nextcloud]> SHOW TABLES;
+-----------------------------+
| Tables_in_nextcloud         |
+-----------------------------+
| oc_accounts                 |
| oc_accounts_data            |
| oc_activity                 |
| oc_activity_mq              |
| oc_addressbookchanges       |
| oc_addressbooks             |
| oc_appconfig                |
| oc_authorized_groups        |
| oc_authtoken                |
| oc_bruteforce_attempts      |
| oc_calendar_invitations     |
| oc_calendar_reminders       |
| oc_calendar_resources       |
| oc_calendar_resources_md    |
| oc_calendar_rooms           |
| oc_calendar_rooms_md        |
| oc_calendarchanges          |
| oc_calendarobjects          |
| oc_calendarobjects_props    |
| oc_calendars                |
| oc_calendarsubscriptions    |
| oc_cards                    |
| oc_cards_properties         |
| oc_circles_circle           |
| oc_circles_event            |
| oc_circles_member           |
| oc_circles_membership       |
| oc_circles_mount            |
| oc_circles_mountpoint       |
| oc_circles_remote           |
| oc_circles_share_lock       |
| oc_circles_token            |
| oc_collres_accesscache      |
| oc_collres_collections      |
| oc_collres_resources        |
| oc_comments                 |
| oc_comments_read_markers    |
| oc_dav_cal_proxy            |
| oc_dav_shares               |
| oc_direct_edit              |
| oc_directlink               |
| oc_federated_reshares       |
| oc_file_locks               |
| oc_file_metadata            |
| oc_filecache                |
| oc_filecache_extended       |
| oc_files_trash              |
| oc_flow_checks              |
| oc_flow_operations          |
| oc_flow_operations_scope    |
| oc_group_admin              |
| oc_group_user               |
| oc_groups                   |
| oc_jobs                     |
| oc_known_users              |
| oc_login_flow_v2            |
| oc_migrations               |
| oc_mimetypes                |
| oc_mounts                   |
| oc_notifications            |
| oc_notifications_pushhash   |
| oc_notifications_settings   |
| oc_oauth2_access_tokens     |
| oc_oauth2_clients           |
| oc_open_local_editor        |
| oc_photos_albums            |
| oc_photos_albums_files      |
| oc_photos_collaborators     |
| oc_preferences              |
| oc_privacy_admins           |
| oc_profile_config           |
| oc_properties               |
| oc_ratelimit_entries        |
| oc_reactions                |
| oc_recent_contact           |
| oc_schedulingobjects        |
| oc_share                    |
| oc_share_external           |
| oc_storages                 |
| oc_storages_credentials     |
| oc_systemtag                |
| oc_systemtag_group          |
| oc_systemtag_object_mapping |
| oc_text_documents           |
| oc_text_sessions            |
| oc_text_steps               |
| oc_trusted_servers          |
| oc_twofactor_backupcodes    |
| oc_twofactor_providers      |
| oc_user_status              |
| oc_user_transfer_owner      |
| oc_users                    |
| oc_vcategory                |
| oc_vcategory_to_object      |
| oc_webauthn                 |
| oc_whats_new                |
+-----------------------------+
96 rows in set (0.008 sec)
```