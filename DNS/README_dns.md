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
docker-compose up --build -d
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

## **Configurando o DNS no docker**

**9-** Altere o arquivo resolv.conf:

Entre no container criado:

```
docker exec -it dns_machine2_1 /bin/bash/
```

Digite:

```bash
vi /etc/resolv.conf
```
Dentro do resolv.conf, escreva:

```bash
nameserver 127.0.0.11
options edns0 trust-ad ndots:0
domain queropassar.com
nameserver 192.168.10.2
nameserver 192.168.10.3
```

**10-** Altere os arquivos named.conf.local

Acesse:

```bash
cd /etc/bind/
```
Inicie alterando o arquivo named.conf.local

```bash
//
// Do any local configuration here
//

// Consider adding the 1918 zones here, if they are not used in your
// organization
//include "/etc/bind/zones.rfc1918";
zone "queropassar.com" in {
   type master;
   file "/etc/bind/db.queropassar";
};
# Zona para dns reverso
zone "10.168.192.in-addr.arpa" in {
    type master;
    file "/etc/bind/db.queroreprovar";
};
```

Depois crie os arquivos db.dns e db.dnsreverse, com seus respectivos nomes:

Primeiro o db.dns:

```bash
# Cada definição de master deve se iniciar com uma entrada SOA
# Ela indica o servidor de nomes para o domínio em questão e parâmetros de operação
@ IN SOA flpp.queropassar.com. root.flpp.queropassar.com. (
2022092601 ;numero serial - deve ser incrementado a cada mudança neste arquivo
21600 ;refresh - das informa��ões para slaves
1800 ;retry �~@~S tempo entre as tentativas
604800 ;expire - tempo para se desistir de contactar master
86400 ) ;mí nimo - tempo a manter a informação no cache (TTL)
IN NS flpp.queropassar.com.
queropassar.com. IN MX 10 FERNANDO.queropassar.com. ;entrada MX (mail server)
localhost IN A 127.0.0.1
machine2 IN A 192.168.10.2
machine3 IN A 192.168.10.3
flpp IN A 192.168.10.66
FERNANDO IN A 192.168.10.100
```

Primeiro o db.dnsreverse:

```bash
# Realiza a resolução reversa
# O tipo PTR significa um alias para o endereço IP
@ IN SOA flpp.queropassar.com. root.flpp.queropassar.com. (
2022092601
21600
1800
604800
86400 )
IN NS flpp.queropassar.com.
2 IN PTR machine2.queropassar.com.
3 IN PTR machine3.queropassar.com.
66 IN PTR flpp.queropassar.com.
100 IN PTR FERNANDO.queropassar.com.
```

**11-** Rode o comando para fazer as mudanças realizadas:

```bash
/etc/init.d/bind9 start
```

ou 

```bash
sudo /usr/sbin/named -f -g -d 1
```

Esse segundo comando serve para rodar mostrando o que está sendo feito durante o processo.

**12-** Testando o DNS

Rode:

```bash
nslookup
```

Consulte:

```bash
server <nome ou IP>
```

