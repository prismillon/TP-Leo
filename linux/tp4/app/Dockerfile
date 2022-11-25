FROM ubuntu

RUN apt update -y

RUN apt install -y golang git

RUN /usr/bin/git clone https://github.com/prismillon/Hangman_web.git /app

WORKDIR /app

CMD [ "/usr/bin/go", "run", "." ]

