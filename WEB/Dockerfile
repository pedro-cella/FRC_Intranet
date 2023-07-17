FROM ubuntu:18.04
WORKDIR /app

RUN apt update
RUN apt install apache2 -y && apt install vim -y && apt install network-manager -y
COPY ./config/* .


CMD apachectl -D FOREGROUND
