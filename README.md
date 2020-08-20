# Elasticsearch (BASIC) with Open Distro plugins

The [Open Distro for Elasticsearch](https://opendistro.github.io/for-elasticsearch/) is an Amazon creation based on Elasticsearch. The problem with it is that it was developed entirely on top of [Elasticsearch Open Source (OSS - Apache 2.0) version, and not the BASIC version](https://www.elastic.co/pt/subscriptions) (that contains a lot of more features), precisely because of the licensing. Basic version has it is own licensing model.

The Elasticsearch Open Source version (OSS) does not contain a number of features of the BASIC version, in particular:
* CANVAS
* APM
* ILM
* Cluster health monitoring (xpack)
* Access Control
* Node to Node Encryption (xpack)
* SQL
* Anomaly Detection

Also, obviously, it lacks the features of the paid version of Elasticsearch:
* Alerting
* Machine Learning
* Graph
* JDBC
* LDAP, AD, KERBEROS, SAML integrations authentication
* Access Control (at field level)

However, Open Distro developed some of these main features on top of the Elasticsearch OSS version. Some (most) can be installed as Elasticsearch plugins, and some others only run in the Open Distro core. Are they:
* Security (plugin): Similar to Authentication + Access Control at the field level + plugin for Kibana
* Alerting (plugin): Similar to the alerting + plugin for Kibana
* Index Management (plugin): Similar to ILM + plugin for Kibana
* SQL (plugin): Similar to SQL
* KNN (core): Machine learning
* Anomaly Detection (plugin): Machine Learning
* Performance Analyzer / Root Cause Analysis (plugin / core): similar to the monitor

Opendistro for Elasticsearch based on the Open Source version of Elasticsearch is available on Linux/Windows and Docker. They also distributed the plugins separately and all the [Opendistro source code is open](https://github.com/opendistro-for-elasticsearch).

A question arises.. would it be possible to run the Elasticsearch (BASIC version) with the plugins that Open Distro developed for the OSS version?
I searched the internet a lot and the answer is as follows: Yes, it is possible, but it has a number of limitations. And, officially speaking, it is not recommended:
* https://discuss.opendistrocommunity.dev/t/is-open-distro-production-ready/2631/6
* https://discuss.opendistrocommunity.dev/t/elasticsearch-security-config/2461

But researching about it, I found several people who are using it that way, including just for the purpose I was thinking of using for, ([Security](https://medium.com/@ibrahim.ayadhi/deploying-of-infrastructure-and-technologies-for-a-soc-as-a-service-socass-8e1bbb885149)):
![SoC with ELK](https://miro.medium.com/max/1000/1*TAoB_84vsDlRA3LhWAuxxA.png)

Searching on github, I found SEVERAL public code repositories containing Dockerfile files to generate Elasticsearch (BASIC) containers with installed plugins. Just search on [Github](https://github.com/): `filename:Dockerfile elasticsearch-plugin opendistro`

So, I decided to follow this path to test. I downloaded an [example](https://opendistro.github.io/for-elasticsearch-docs/docs/security/configuration/ldap/) of [OpenDistro using LDAP](https://opendistro.github.io/for-elasticsearch-docs/assets/examples/ldap-example.zip) and changed a few things to test whether Elasticsearch (basic) would work with some of the Open Distro plugins.

I also generated the following Dockerfile with the following Open Distro plugins:
* Security
* SQL
* Job Scheduler (required for Alerting to work)
* Anomaly Detection (only works via API)
* Alerting
* Index Management

This is the Elasticsearch 7.8.0 Dockerfile (other than OSS) with the Open Distro plugins:

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


# config security (certs, etc.)
RUN sh /usr/share/elasticsearch/plugins/opendistro_security/tools/install_demo_configuration.sh -y -i

EXPOSE 9200


```

This is the Kibana 7.8.0 Dockerfile (other than OSS) with Open Distro plugins:
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


# To run:
It is necessary to have the docker installed. Just enter the folder where this repository was clone (`git clone https://github.com/skysbsb/elk-opendistro-plugins-ldap.git`) and execute the command:` docker-compose up -d`.

The docker service will be in charge of downloading the Elasticsearch (basic) image directly from the Docker Hub registry, the Open Distro plugins (from another repo of mine) and installing all together.

# Containers
4 containers will go up.
* An elasticsearch (basic) with the Open Distro plugins installed.
* A kibana (basic) with Open Distro plugins installed
   * https://localhost:5601
   * admin:admin (internal user)
   * jroe:password (ldap user with admin role)
   * psantos:password (ldap user with developer/readonly role)
* An openldap
* A phpldapadmin
   * https://localhost:6443
   * user: cn=admin,dc=example,dc=org senha: changethis

# Documentation
For more information regarding these containers, visit: https://opendistro.github.io/for-elasticsearch-docs/docs/security/configuration/ldap/

# Evaluation
I could see that Elasticsearch (BASIC) worked well with the Open Distro plugins made for Elasticsearch (OSS). Some things didn't work visually (Anomaly), or got a little buggy, but I couldn't identify anything literally broken, however, I still haven't had time to do the tests exhaustively.

The Alerting part worked smoothly. It was possible to create the monitor, trigger and destination through the Kibana interface without any problems.

I was able to use SQL queries using the Open Distro plugin API.

It was possible to integrate with LDAP and create specific roles for LDAP groups. Open Distro keeps the security information all within an ELK index, but it is also possible to change the configuration files inside (or outside) the container and run securityadmin.sh to synchronize with the ELK via API. To add some authentication or authorization method (authc authz) or the form of access to the cluster (basic, kerberos, etc.), you can change the config.yml configuration file.

It was possible to create and edit CANVAS, which is part of the BASIC version.

It was possible to monitor the cluster with xpack.

That is, it was possible to have the best of both worlds (ELK BASIC + OPEN DISTRO), however, as warned at the beginning, Open Distro does not recommend that it be used in this way and there are many things that need to be tested before proceeding with this one. The risk of having something broken up front or something not working has to be taken into account.

