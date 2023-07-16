# Configuração do SMTP e POP3 com Postfix e Dovecot

## SMTP (Envio de Emails)

### Passo 1: Instalação do Postfix

```bash
sudo apt install postfix
```

Respostas durante a instalação:
- Ok
- Site da Internet (Internet Site)
- pv-notebook

### Passo 2: Configuração do Postfix

Durante a configuração, responda as seguintes opções:
- Ok
- Site da Internet (Internet Site)
- pv-notebook NONE
- chmod.com.br
- chmod.com.br, pv-notebook, pv-notebook, localhost.localdomain, , localhost_
- Não
- 127.0.0.0/8 [::ffff:127.0.0.0]/104 [::1]/128_
- Limite de tamanho: 0
- Caractere de extensão: +
- Todos (all)

### Passo 3: Iniciando o Postfix

```bash
sudo service postfix start
```

### Passo 4: Adicionando usuários ao grupo

```bash
adduser francisco
adduser leonardo
adduser paulo
adduser pedro
```

Senha para todos os usuários: redes

### Passo 5: Instalação do utilitário de email

```bash
sudo apt install mailutils
```

### Passo 6: Enviando uma mensagem de email

Use o comando abaixo substituindo "nomeuser" pelo usuário desejado e "chmod.com.br" pelo domínio correto:

```bash
mail nomeuser@chmod.com.br
```

Escreva a mensagem e, quando terminar, utilize o comando CTRL + D para encerrar e enviar.

### Passo 7: Ver mensagens enviadas

As mensagens enviadas estarão na pasta:

```
/var/mail/
```

Acesse o usuário e veja as mensagens.

## POP3 e IMAP com Dovecot

### Passo 1: Instalação do Dovecot

```bash
sudo apt install dovecot-core dovecot-imapd dovecot-pop3d
```

### Passo 2: Configuração do Dovecot para aceitar POP3 e IMAP

Edite o arquivo de configuração:

```bash
sudo nano /etc/dovecot/dovecot.conf
```

Adicione ou descomente as seguintes linhas para aceitar POP3 e IMAP:

```
protocols = imap pop3
```

Realize o restart do Dovecot:

```bash
sudo systemctl restart dovecot
```

Agora o Dovecot está configurado para suportar ambos os protocolos, POP3 e IMAP. O servidor estará pronto para receber conexões dos clientes de email.

# SMTP com Docker

Para fazer a instalação e configuração do Postfix usando Docker, você pode seguir os seguintes passos:

## Passo 1: Acesse o diretório:

```bash
cd SMTP
```

## Passo 2: Crie um arquivo chamado `Dockerfile` com o seguinte conteúdo:

```Dockerfile
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
```

## Passo 3: Crie um arquivo chamado `postfix-config.txt` com o seguinte conteúdo:

```
# Configuração do Postfix
myhostname = chmod.com.br
mydestination = chmod.com.br, localhost.localdomain, localhost
mynetworks = 127.0.0.0/8, 192.168.10.0/24
inet_interfaces = all
```

Substitua "chmod.com.br" pelo nome do seu servidor de e-mail (definido no DNS) e o endereço "192.168.10.0/24" pelo endereço de rede que você definiu para a rede local da sua bancada.

## Passo 4: Construa a imagem Docker:

```bash
docker build -t smtp .
```

## Passo 5: Execute o contêiner Docker:

```bash
docker run -d --name smtp -p 2525:25 smtp
```

Agora você tem um servidor de e-mail Postfix em execução dentro do contêiner Docker. Você pode adicionar usuários e enviar e-mails da mesma forma como fez anteriormente:

```bash
docker exec -it smtp bash

# Agora dentro do contêiner, adicione os usuários
adduser francisco
adduser leonardo
adduser paulo
adduser pedro

# Enviando os e-mails
mail francisco@chmod.com.br
mail leonardo@chmod.com.br
mail paulo@chmod.com.br
mail pedro@chmod.com.br
```

Escreva a mensagem e, quando terminar, utilize o comando `CTRL + D` para encerrar e enviar.

Os e-mails enviados estarão disponíveis nos respectivos arquivos dentro do diretório `/var/mail` dentro do contêiner. Para ver os e-mails recebidos, você pode executar:

```bash
docker exec -it smtp /bin/bash
# Agora dentro do contêiner
cat /var/mail/francisco
cat /var/mail/gabriela
cat /var/mail/murilo
cat /var/mail/pedro
```

Lembrando que esse é um exemplo básico para executar o Postfix dentro do Docker, e em um ambiente real, você precisaria fazer algumas outras configurações para garantir a segurança e autenticação correta dos usuários.