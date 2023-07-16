# Use a imagem base do Ubuntu (ou outra imagem de sua preferência)
FROM ubuntu:latest

# Instale o servidor de e-mail Postfix
RUN apt-get update && apt-get install -y postfix

# Instale o mail
RUN apt install mailutils -y

# Copie o arquivo de configuração pré-configurado para dentro do contêiner
COPY postfix-config.txt /etc/postfix/main.cf

# Inicie o serviço do Postfix quando o contêiner for iniciado
CMD service postfix start && tail -f /dev/null
