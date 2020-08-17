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
