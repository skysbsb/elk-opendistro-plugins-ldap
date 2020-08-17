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