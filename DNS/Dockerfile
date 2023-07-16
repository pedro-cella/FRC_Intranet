FROM ubuntu:18.04
WORKDIR /app
RUN apt update
RUN apt install bind9 -y
RUN apt install net-tools -y
RUN apt install dnsutils -y
RUN apt install vim -y
RUN apt install mailutils -y
RUN apt install postfix -y

COPY config/* .
COPY config.sh .
RUN chmod +x config.sh

CMD tail -f /dev/null
