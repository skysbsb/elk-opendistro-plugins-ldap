# Elasticsearch (BASIC) com plugins do Open Distro

O [Open Distro for Elasticsearch](https://opendistro.github.io/for-elasticsearch/) é uma criação da Amazon baseado no Elasticsearch. O problema é que foi desenvolvido todo em cima da [versão Open Source (OSS - Apache 2.0) do Elasticsearch, e não da versão BASIC](https://www.elastic.co/pt/subscriptions), justamente por conta do licenciamento.

O Elasticsearch Open Source (OSS) não contém uma série de funcionalidades da versão BASIC, em especial:
* CANVAS
* APM
* ILM
* Monitoramento do cluster (xpack)
* Controle de acesso
* Node to Node Encryption (xpack)
* SQL
* Anomaly Detection


E também, obviamente, não possui as funcionalidades da versão paga do Elasticsearch:
* Alerting
* Machine Learning
* Graph
* JDBC
* Authentication LDAP, AD, KERBEROS, SAML integrations
* Access Control (a nível de field)

Porém, o Open Distro desenvolveu algumas dessas principais funcionalidades em cima da versão OSS. Algumas (a maioria) podem ser instaladas como plugins do Elasticsearch, e algumas outras somente rodando o core do Open Distro. São elas:
* Security (plugin): Similar ao Authentication + Access Control a nível de field + plugin para o Kibana
* Alerting (plugin): Similar ao alerting + plugin para o Kibana
* Index Management (plugin): Similar ao ILM + plugin para o Kibana
* SQL (plugin): Similar ao SQL
* KNN (core): Machine learning
* Anomaly Detection (plugin)
* Performance Analyzer / Root Cause Analysis (plugin/core): similar ao monitor


O Opendistro for Elasticsearch baseado na versão Open Source do Elasticsearch está disponível em Linux/Windows e Docker. Eles também distribuiram os plugins separadamente e todo o [código fonte do Opendistro é aberto](https://github.com/opendistro-for-elasticsearch).

Ai pintou uma dúvida.. será que seria possível rodar a versão BASIC do ELK com os plugins que o Open Distro desenvolveu para a versão OSS?
Procurei bastante na internet e a resposta é a seguinte: Sim, é possível, porém possui uma série de limitações. E, oficialmente falando, não é recomendado:
* https://github.com/opendistro-for-elasticsearch/security-kibana-plugin/issues/32
* https://discuss.opendistrocommunity.dev/t/is-open-distro-production-ready/2631/6
* https://doc.punchplatform.com/6.0.1/Reference_Guide/Security/Security_opendistro.html

Porém pesquisando a respeito, encontrei várias pessoas que estão usando dessa forma, inclusive justamente para a finalidade que eu estava pensando em usar ([Security](https://medium.com/@ibrahim.ayadhi/deploying-of-infrastructure-and-technologies-for-a-soc-as-a-service-socass-8e1bbb885149)):
![SoC with ELK](https://miro.medium.com/max/1000/1*TAoB_84vsDlRA3LhWAuxxA.png)

Pesquisando no github, encontrei DIVERSOS repositórios de código públicos contendo arquivos Dockerfile para gerar containers do Elasticsearch (BASIC) com os plugins instalados. Basta pesquisar no [Github](https://github.com/): `filename:Dockerfile elasticsearch-plugin opendistro`

Assim sendo, resolvi seguir esse caminho para testar. Baixei um [exemplo](https://opendistro.github.io/for-elasticsearch-docs/docs/security/configuration/ldap/) do [OpenDistro usando LDAP](https://opendistro.github.io/for-elasticsearch-docs/assets/examples/ldap-example.zip) e alterei algumas coisas para testar se funcionaria o Elasticsearch (basic) com alguns dos plugins do Open Distro.

Gerei os seguintes Dockerfile com os seguintes plugins do Open Distro:
* Security
* SQL
* Job Scheduler (necessário para o Alerting funcionar)
* Anomaly Detection (só funciona via API)
* Alerting
* Index Management

Esse é o Dockerfile do Elasticsearch 7.8.0 (sem ser OSS) com os plugins do Open Distro:

```Dockerfile
ARG VERSION="7.8.0"

FROM docker.elastic.co/elasticsearch/elasticsearch:${VERSION}

# install open distro security plugin
RUN bin/elasticsearch-plugin install -b https://raw.githubusercontent.com/skysbsb/opendistro-plugins/master/7.8.0/elasticsearch/opendistro-job-scheduler.zip
RUN bin/elasticsearch-plugin install -b https://raw.githubusercontent.com/skysbsb/opendistro-plugins/master/7.8.0/elasticsearch/opendistro-alerting.zip
RUN bin/elasticsearch-plugin install -b https://raw.githubusercontent.com/skysbsb/opendistro-plugins/master/7.8.0/elasticsearch/opendistro-anomaly-detection.zip
RUN bin/elasticsearch-plugin install -b https://raw.githubusercontent.com/skysbsb/opendistro-plugins/master/7.8.0/elasticsearch/opendistro-index-management.zip
RUN bin/elasticsearch-plugin install -b https://raw.githubusercontent.com/skysbsb/opendistro-plugins/master/7.8.0/elasticsearch/opendistro-sql.zip
RUN bin/elasticsearch-plugin install -b https://raw.githubusercontent.com/skysbsb/opendistro-plugins/master/7.8.0/elasticsearch/opendistro-security.zip
#RUN bin/elasticsearch-plugin install -b https://raw.githubusercontent.com/skysbsb/opendistro-plugins/master/7.8.0/elasticsearch/performance-analyzer.zip


# configura security (certificados, etc.)
RUN sh /usr/share/elasticsearch/plugins/opendistro_security/tools/install_demo_configuration.sh -y -i

EXPOSE 9200


```

Esse é o Dockerfile do Kibana 7.8.0 (sem ser OSS) com os plugins do Open Distro:
```Dockerfile
ARG VERSION="7.8.0"

FROM docker.elastic.co/kibana/kibana:${VERSION}

RUN chown kibana:kibana config/kibana.yml

# install open distro security plugin to Kibana
RUN bin/kibana-plugin install --allow-root  https://raw.githubusercontent.com/skysbsb/opendistro-plugins/master/7.8.0/kibana/opendistro-alerting.zip
RUN bin/kibana-plugin install --allow-root  https://raw.githubusercontent.com/skysbsb/opendistro-plugins/master/7.8.0/kibana/opendistro-index-management.zip
RUN bin/kibana-plugin install --allow-root  https://raw.githubusercontent.com/skysbsb/opendistro-plugins/master/7.8.0/kibana/opendistro-security.zip


EXPOSE 5601

```


# Para rodar:
É necessário ter o docker instalado. Eu baixei o [Docker Desktop para Windows](https://docs.docker.com/docker-for-windows/install/) e integrei com meu [WSL 2](https://docs.microsoft.com/en-us/windows/wsl/install-win10). Assim pude usar os comandos do Docker no Windows usando o Terminal novo do WSL.
Basta entrar na pasta onde foi feito o clone desse repositório (`git clone https://github.com/skysbsb/elk-opendistro-plugins-ldap.git`) e executar o comando: `docker-compose up -d`.

O docker vai se encarregar de baixar a imagem do Elasticsearch (basic) direto do registry do Docker Hub, os plugins do Open Distro (de outro repo meu) e instalar este último, naquele.

# Containers
Vão subir 4 containers.
* Um elasticsearch (basic) com os plugins do Open Distro instalados.
* Um kibana (basic) com os plugins do Open Distro instalados
  * https://localhost:5601
  * admin:admin (internal user)
  * jroe:password (ldap user com role de admin)
  * psantos:password (ldap user com role de developer/readonly)
* Um openldap
* Um phpldapadmin
  * https://localhost:6443
  * user: cn=admin,dc=example,dc=org senha: changethis

# Documentação
Para mais informações com relação a esses containers, acesse: https://opendistro.github.io/for-elasticsearch-docs/docs/security/configuration/ldap/

# Avaliação
Deu pra ver que o Elasticsearch (BASIC) funcionou bem com os plugins do Open Distro feitos para o Elasticsearch (OSS). Algumas coisas visualmente não funcionaram (Anomaly), ou ficaram meio bugadas, porém não consegui identificar nada literalmente quebrado, porém, eu ainda não tive tempo de fazer os testes exaustivamente.

A parte de Alerting funcionou tranquilo. Deu para criar o monitor, trigger e destination pela interface do Kibana sem problemas. 

Deu para usar consultas SQL usando a API do plugin do Open Distro.

Deu para integrar com o LDAP e criar roles específicas para grupos do LDAP. O Open Distro mantém as informações de segurança todas dentro de um índice do ELK, porém é possível também alterar os arquivos de configuração dentro (ou fora) do container e rodar o securityadmin.sh para sincronizar com o ELK via API. Para adicionar algum método de authentication ou autorization (authc authz) ou a forma de acesso ao cluster (basic, kerberos, etc.), pode-se alterar o arquivo de configuração config.yml.

Deu para criar e editar CANVAS, que faz parte da versão BASIC.

Deu para monitorar o cluster com o xpack.

Ou seja, deu pra ter o melhor dos dois mundos (ELK BASIC + OPEN DISTRO), porém, como alertado lá no começo, o Open Distro não recomenda que seja usado dessa maneira e existem muitas coisas que precisam ser testadas antes de seguir por esse caminho. O risco de ter alguma coisa quebrada lá na frente ou de alguma coisa não funcionar tem que ser levado em conta.



