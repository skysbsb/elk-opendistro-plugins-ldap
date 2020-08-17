# ElasticSearch (BASIC) com plugins do Open Distro for Elasticsearch

O [Open Distro for Elasticsearch](https://opendistro.github.io/for-elasticsearch/) é uma criação da Amazon baseado no Elasticsearch. O problema é que foi desenvolvido todo em cima da [versão Open Source (OSS - Apache 2.0) do Elasticsearch, e não da versão BASIC](https://www.elastic.co/pt/subscriptions), justamente por conta do licenciamento da versão BASIC sem mais restritivo.

Assim sendo, ele não contém uma série de funcionalidades da versão BASIC, em especial:
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

Porém, eles desenvolveram algumas dessas funcionalidades essenciais em cima da versão OSS. Algumas (a maioria) podem ser instaladas como plugins do Elasticsearch, e algumas outras somente rodando o core do Open Distro. São elas:
* Security (plugin): Similar ao Authentication + Access Control a nível de field + plugin para o Kibana
* Alerting (plugin): Similar ao alerting + plugin para o Kibana
* Index Management (plugin): Similar ao ILM + plugin para o Kibana
* SQL (plugin): Similar ao SQL
* KNN (core): Machine learning
* Anomaly Detection (plugin)
* Performance Analyzer / Root Cause Analysis (plugin/core): similar ao monitor


O Opendistro for Elasticsearch baseado na versão Open Source do Elasticsearch está disponível em Linux/Windows e Docker. Eles também distribuiram os plugins separadamente, e todo o [código fonte do Opendistro é aberto](https://github.com/opendistro-for-elasticsearch).

Baixei um exemplo do [OpenDistro usando LDAP](https://opendistro.github.io/for-elasticsearch-docs/assets/examples/ldap-example.zip) e só alterei as imagens utilizadas no campo "image:" do docker-compose.yml.

Gerei as seguintes imagens:

* Elasticsearch 7.8.0 (sem ser OSS) com os seguintes plugins do OpenDistro:

```Dockerfile
ARG VERSION="7.8.0"

FROM docker.elastic.co/elasticsearch/elasticsearch:${VERSION}
#USER root
#WORKDIR /usr/share/elasticsearch/

# install open distro security plugin
RUN bin/elasticsearch-plugin install -b https://raw.githubusercontent.com/skysbsb/opendistro-plugins/master/7.8.0/elasticsearch/opendistro-job-scheduler.zip
RUN bin/elasticsearch-plugin install -b https://raw.githubusercontent.com/skysbsb/opendistro-plugins/master/7.8.0/elasticsearch/opendistro-alerting.zip
RUN bin/elasticsearch-plugin install -b https://raw.githubusercontent.com/skysbsb/opendistro-plugins/master/7.8.0/elasticsearch/opendistro-anomaly-detection.zip
RUN bin/elasticsearch-plugin install -b https://raw.githubusercontent.com/skysbsb/opendistro-plugins/master/7.8.0/elasticsearch/opendistro-index-management.zip
RUN bin/elasticsearch-plugin install -b https://raw.githubusercontent.com/skysbsb/opendistro-plugins/master/7.8.0/elasticsearch/opendistro-sql.zip
RUN bin/elasticsearch-plugin install -b https://raw.githubusercontent.com/skysbsb/opendistro-plugins/master/7.8.0/elasticsearch/opendistro-security.zip
#RUN bin/elasticsearch-plugin install -b https://raw.githubusercontent.com/skysbsb/opendistro-plugins/master/7.8.0/elasticsearch/performance-analyzer.zip

# configura performance-analyzer
#ADD https://raw.githubusercontent.com/opendistro-for-elasticsearch/performance-analyzer/master/packaging/opendistro-performance-analyzer.service /usr/lib/systemd/system/opendistro-performance-analyzer.service

# configura security (certificados, etc.)
RUN sh /usr/share/elasticsearch/plugins/opendistro_security/tools/install_demo_configuration.sh -y -i

#USER elasticsearch

EXPOSE 9200


### Success
### Execute this script now on all your nodes and then start all nodes
### Open Distro Security will be automatically initialized.
### If you like to change the runtime configuration
### change the files in ../securityconfig and execute:
#"/usr/share/elasticsearch/plugins/opendistro_security/tools/securityadmin.sh" -cd "/usr/share/elasticsearch/plugins/opendistro_security/securityconfig" -icl -key "/usr/share/elasticsearch/config/kirk-key.pem" -cert "/usr/share/elasticsearch/config/kirk.pem" -cacert "/usr/share/elasticsearch/config/root-ca.pem" -nhnv
### or run ./securityadmin_demo.sh
### To use the Security Plugin ConfigurationGUI
### To access your secured cluster open https://<hostname>:<HTTP port> and log in with admin/admin.
### (Ignore the SSL certificate warning because we installed self-signed demo certificates)
```

* Kibana 7.8.0 (sem ser OSS) com os seguintes plugins do Open Distro:
```Dockerfile
ARG VERSION="7.8.0"

FROM docker.elastic.co/kibana/kibana:${VERSION}
#USER root
#WORKDIR /usr/share/kibana/

RUN chown kibana:kibana config/kibana.yml

# install open distro security plugin to Kibana
RUN bin/kibana-plugin install --allow-root  https://raw.githubusercontent.com/skysbsb/opendistro-plugins/master/7.8.0/kibana/opendistro-alerting.zip
RUN bin/kibana-plugin install --allow-root  https://raw.githubusercontent.com/skysbsb/opendistro-plugins/master/7.8.0/kibana/opendistro-index-management.zip
RUN bin/kibana-plugin install --allow-root  https://raw.githubusercontent.com/skysbsb/opendistro-plugins/master/7.8.0/kibana/opendistro-security.zip

#USER kibana

EXPOSE 5601

```


 

Doc: https://opendistro.github.io/for-elasticsearch-docs/docs/security/configuration/ldap/

# LDAP
You can access the administration tool at https://localhost:6443. Acknowledge the security warning and log in using cn=admin,dc=example,dc=org and changethis.

* psantos is in the Administrator and Developers groups. jroe and jdoe are in the Developers group. The security plugin loads these groups as backend roles.
* roles_mapping.yml maps the Administrator and Developers LDAP groups (as backend roles) to security roles so that users gain the appropriate permissions after authenticating.
* curl -XPUT https://localhost:9200/new-index/_doc/1 -H 'Content-Type: application/json' -d '{"title": "Spirited Away"}' -u psantos:password -
* curl -XGET https://localhost:9200/new-index/_search?pretty -u jroe:password -k
