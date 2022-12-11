# TP5

# Installing pufferpanel

Using [a modified install script](./script.rpm.sh) we can install pufferpanel in rocky 9 even if it's not officially supported.

Then, we follow the normal installation

```sh
$ sudo yum install pufferpanel
$ sudo systemctl enable pufferpanel
```

Docker is also required to run pufferpanel in docker mode

```sh
$ sudo dnf config-manager --add-repo=https://download.docker.com/linux/centos/docker-ce.repo
$ sudo dnf update
$ sudo dnf install docker-ce docker-ce-cli containerd.io
$ sudo systemctl enable docker
```


The firewall should look like this:
```sh
$ sudo firewall-cmd --list-all
public (active)
  target: default
  icmp-block-inversion: no
  interfaces: enp0s3 enp0s8
  sources: 
  services: ssh
  ports: 8080/tcp 25565-25570/tcp
  protocols: 
  forward: yes
  masquerade: no
  forward-ports: 
  source-ports: 
  icmp-blocks: 
  rich rules: 
```

Note: We open a range of port because pufferpanel's pre-run hook doesnt seem to work.

# nginx reverse proxy install and config on a seprate machine

install nginx

```bash
sudo dnf install nginx
```

enable nginx at startup

```bash
sudo systemctl enable nginx
```

start nginx

```bash
sudo systemctl start nginx
```

edit the conf to remove the default server

```bash
sudo vim /etc/nginx/nginx.conf
```

add the conf for pufferpanel reverse proxy with ssl

```bash
sudo vim /etc/nginx/conf.d/puffer.conf
```

create the ssl certificate

```bash
cd /etc/nginx/
openssl req -x509 -sha256 -nodes -days 365 -newkey rsa:2048 -keyout localhost.key -out localhost.crt
```

restart nginx

```bash
sudo systemctl restart nginx
```

open the firewall for nginx

```bash
sudo firewall-cmd --add-port=443/tcp
sudo !! --permanent
```