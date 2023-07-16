FROM ubuntu:18.04
WORKDIR /app

RUN apt update
RUN apt install apache2 -y

CMD apachectl -D FOREGROUND
