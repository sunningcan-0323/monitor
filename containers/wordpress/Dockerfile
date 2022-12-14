FROM wordpress:5.8-apache
ENV WORDPRESS_DB_USER=root
ENV WORDPRESS_DB_PASSWORD=petclinic
ENV WORDPRESS_DB_NAME=wordpress
ENV WORDPRESS_DB_HOST=database
ENV WORDPRESS_TABLE_PREFIX=wp_
ENV WORDPRESS_HOST=13.250.61.239:1180

COPY wp-config.php /var/www/html/wp-config.php
RUN mkdir /var/www/html/wp-content/upgrade
COPY migrate.php /var/www/html/migrate.php

RUN apt-get -y update
RUN apt-get -y install wget

# Install monitoring
RUN wget https://github.com/ncabatoff/process-exporter/releases/download/v0.7.5/process-exporter-0.7.5.linux-amd64.tar.gz -O /tmp/process-exporter-0.7.5.linux-amd64.tar.gz
RUN mkdir /opt/exporters
RUN cd /opt/exporters;
RUN tar xvf /tmp/process-exporter-0.7.5.linux-amd64.tar.gz; mv process-exporter-0.7.5.linux-amd64/process-exporter /opt/exporters/process-exporter
RUN rm -rf process-exporter-0.7.5.linux-amd64
RUN rm -f process-exporter-0.7.5.linux-amd64.tar.gz
COPY process-exporter.conf /opt/exporters
RUN wget https://github.com/prometheus/node_exporter/releases/download/v1.2.0/node_exporter-1.2.0.linux-amd64.tar.gz -O /tmp/node_exporter-1.2.0.linux-amd64.tar.gz
RUN cd /opt/exporters; tar xvf /tmp/node_exporter-1.2.0.linux-amd64.tar.gz; mv node*/node_exporter /opt/exporters/node_exporter
RUN rm -rf /tmp/node_exporter-1.2.0.linux-amd64.tar.gz /opt/exporters/node_exporter-*
RUN wget https://github.com/dundee/disk_usage_exporter/releases/download/v0.1.0/disk_usage_exporter_linux_amd64.tgz -O /tmp/disk_usage_exporter_linux_amd64.tgz
RUN cd /opt/exporters; tar xvf /tmp/disk_usage_exporter_linux_amd64.tgz; mv disk* disk_usage_exporter
COPY disk_usage_exporter.yaml /opt/exporters/disk_usage_exporter.yaml

# Change port 80 as OpenShift doesn't like it
RUN sed -i 's, \*:80>, *:8001>,' /etc/apache2/sites-enabled/000-default.conf
RUN sed -i 's/80/8001/' /etc/apache2/ports.conf

# Set logs to be in the container and to stdout/err
# RUN rm /var/log/apache2/*
# RUN sed -i 's,ErrorLog.*,ErrorLog "|/usr/bin/tee ${APACHE_LOG_DIR}/error.log",' /etc/apache2/sites-enabled/000-default.conf
# RUN sed -i 's,CustomLog.*,CustomLog "|/usr/bin/tee ${APACHE_LOG_DIR}/access.log" combined,' /etc/apache2/sites-enabled/000-default.conf
# RUN egrep "CustomLog|ErrorLog" /etc/apache2/sites-enabled/000-default.conf

# Not working, but get /server-status url working for apache-exporter
# COPY 001-status.conf /etc/apache2/sites-enabled/001-status.conf
# RUN sed -i 's/Require local/#Require local/' /etc/apache2/mods-enabled/status.conf


# Ports exposed for app and
EXPOSE 9100
EXPOSE 9995
EXPOSE 9256
EXPOSE 8001
EXPOSE 8080
EXPOSE 8443

# Install ITRS Geneos netprobe
RUN wget https://www.dropbox.com/sh/q63dnt8e4b99n0p/AACcQ2U4yRU2EmYpEcUISrr7a/geneos-netprobe-5.8.2-linux-x64.tar.gz?dl=0 -O /tmp/geneos-netprobe-5.8.2-linux-x64.tar.gz
RUN mkdir -p /opt/itrs/netprobe
RUN cd /opt/itrs; tar xvf /tmp/geneos-netprobe-5.8.2-linux-x64.tar.gz
COPY netprobe.conf /etc/ld.so.conf.d/netprobe.conf
RUN rm /tmp/*gz

# Update entrypoint
RUN sed -i '/exec "$@"/i\# Start Process exporter\n \
cd /opt/exporters\n \
./process-exporter -config.path process-exporter.conf &\n \
# Start Node exporter\n \
./node_exporter &\n \
# Start disk usage\n \
./disk_usage_exporter &\n \
# # Migrate the DB \n \
# php /var/www/html/migrate.php \n \
# Start netprobe\n \
cd /opt/itrs/netprobe\n \
./netprobe.linux_64 -port 7036 -nopassword &' /usr/local/bin/docker-entrypoint.sh
