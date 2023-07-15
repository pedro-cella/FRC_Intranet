# DNS

## Configuração

### <b>Montagem de rede interconectada para o experimento</b>

**1-** Certifique-se de ter o Docker e o Docker Compose instalados em seu sistema.

**2-** Crie um arquivo chamado docker-compose.yml e adicione o seguinte conteúdo:

```docker
version: '3'

services:
  machine1:
    build:
      context: .
    networks:
      mynetwork:
        ipv4_address: 192.168.10.1

  machine2:
    build:
      context: .
    networks:
      mynetwork:
        ipv4_address: 192.168.10.2

networks:
  mynetwork:
    driver: bridge
    ipam:
      config:
        - subnet: 192.168.10.0/24
```

**3-** Em paralelo crie um arquivo chamado Dockerfile e adicione o seguinte conteúdo:

```Dockerfile
FROM alpine
CMD tail -f /dev/null
```

**4-** No terminal, navegue até o diretório onde você salvou os arquivos 'docker-compose.yml' e 'Dockerfile'.

**5-** Execute o comando a seguir para construir as imagens e iniciar os containers:

```bash
docker-compose up -d
```

Aguarde até que os containers sejam criados e iniciados. Eles serão nomeados como nomedopdiretório_machine1_1 e nomedopdiretório_machine2_1

**6-** Para testar a conectividade entre os containers usando o comando ping, execute os seguintes comandos:

```bash
docker exec nomedopdiretório_machine1_1 ping 192.168.10.2
docker exec nomedopdiretório_machine2_1 ping 192.168.10.1
```

Os comandos acima executarão o ping de um container para o outro usando os endereços IP fornecidos. Se a conectividade estiver correta, você receberá respostas dos pings e poderá confirmar que os containers estão se comunicando com sucesso na mesma rede local.

<b>OBS:</b> Lembre-se de substituir nomedopdiretório pelo nome do diretório em que você salvou os arquivos docker-compose.yml e Dockerfile.

### <b>Configuração estática local de endereços/nomes (através arquivo hosts)</b>

**1-** Identifique os nomes que você deseja atribuir às máquinas da rede montada. Por exemplo, vamos usar os nomes "machine1" e "machine2".

**2-** Abra um terminal e execute o seguinte comando para acessar o shell de um dos containers:

```bash
docker exec -it nomedopdiretório_machine1_1 sh
```
Substitua nomedopdiretório pelo nome do diretório em que você salvou os arquivos do Docker Compose.

**3-** Dentro do shell do container, edite o arquivo /etc/hosts usando um editor de texto, como o vi:

```bash
vi /etc/hosts
```

**4-** No arquivo /etc/hosts, adicione uma linha para cada nome que você deseja atribuir ao endereço IP correspondente. Por exemplo:

machine1

```text
127.0.0.1       localhost
192.168.10.1    machine1
192.168.10.2    machine2 <-- Acrescente esse trecho
~
```

machine2

```text
127.0.0.1       localhost
192.168.10.2    machine2
192.168.10.1    machine1 <-- Acrescente esse trecho
~
```
Varia de caso a caso mas será algo parecido com isso

**5-** Salve e feche o arquivo.

**6-** Agora você configurou as associações fixas e estáticas entre os nomes e os endereços IP dentro dos containers.

Para testar a resolução de nomes local usando o arquivo /etc/hosts, execute o comando ping usando os nomes em vez dos endereços IP. Por exemplo:

```bash
docker exec -it nomedopdiretório_machine1_1 ping machine2
docker exec -it nomedopdiretório_machine2_1 ping machine1
```
Você deve ver os pacotes de ping sendo enviados e recebidos entre os containers usando os nomes configurados.

**7-** Para verificar o nome atual do sistema dentro de um container, use o comando hostname:

```bash
docker exec -it nomedopdiretório_machine1_1 hostname
```

Para alterar o nome do sistema dentro de um container, use o comando hostname seguido pelo novo nome:

```bash
docker exec -it nomedopdiretório_machine1_1 hostname novo-nome
```

Lembre-se de substituir nomedopdiretório pelo nome do diretório em que você salvou os arquivos do Docker Compose.

Caso você receba esse erro:

```bash
hostname: sethostname: Operation not permitted
```
Basta acrescentar o hostname ao arquivo docker-compose.yml

```docker
version: '3'

services:
  machine1:
    build:
      context: .
    networks:
      mynetwork:
        ipv4_address: 192.168.10.1
    hostname: machine1

  machine2:
    build:
      context: .
    networks:
      mynetwork:
        ipv4_address: 192.168.10.2
    hostname: machine2

networks:
  mynetwork:
    driver: bridge
    ipam:
      config:
        - subnet: 192.168.10.0/24
```

e recriar os contâiners com:

```bash
docker-compose up -d
```

**8-** Para verificar o domínio configurado no sistema dentro de um container, use o comando dnsdomainname.

<b>OBS: </b>O comando dnsdomainname não está disponível dentro dos contêineres Docker. No contexto dos contêineres, não existe um domínio configurado da mesma forma que em um sistema operacional completo.

Os contêineres Docker são isolados e têm seu próprio namespace de rede. Eles não estão cientes dos domínios configurados fora do contêiner. Portanto, ao executar o comando dnsdomainname dentro de um contêiner, ele não retornará nada.