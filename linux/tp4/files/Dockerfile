FROM ubuntu

RUN apt update -y

RUN apt install -y apache2

RUN /bin/mkdir "etc/apache2/logs/"

COPY ./index.html /var/www/html/index.html

COPY ./apache2.conf /etc/apache2/apache2.conf

CMD ["/usr/sbin/apache2ctl", "-DFOREGROUND"]
