# Docker container for Observium Community Edition
#
# It requires option of e.g. '--link observiumdb:observiumdb' with another MySQL or MariaDB container.
# Example usage:
# 1. MySQL or MariaDB container
#    $ docker run --name observiumdb \
#        -v /home/docker/observium/data:/var/lib/mysql \
#        -e MYSQL_ROOT_PASSWORD=passw0rd \
#        -e MYSQL_USER=observium \
#        -e MYSQL_PASSWORD=passw0rd \
#        -e MYSQL_DATABASE=observium \
#        mariadb
#
# 2. This Observium container
#    $ docker run --name observiumapp --link observiumdb:observiumdb \
#        -v /home/docker/observium/logs:/opt/observium/logs \
#        -v /home/docker/observium/rrd:/opt/observium/rrd \
#        -e OBSERVIUM_ADMIN_USER=admin \
#        -e OBSERVIUM_ADMIN_PASS=passw0rd \
#        -e OBSERVIUM_DB_HOST=observiumdb \
#        -e OBSERVIUM_DB_USER=observium \
#        -e OBSERVIUM_DB_PASS=passw0rd \
#        -e OBSERVIUM_DB_NAME=observium \
#        -e OBSERVIUM_BASE_URL=http://yourserver.yourdomain:80 \
#        -p 80:80 mbixtech/observium
#
# References:
#  - Follow platform guideline specified in https://github.com/docker-library/official-images
# 

FROM ubuntu:22.04

LABEL maintainer "noc"
LABEL version="1.0"
LABEL description="Docker container for Observium Community Edition, included Syslog-NG"

ARG OBSERVIUM_ADMIN_USER=admin
ARG OBSERVIUM_ADMIN_PASS=passw0rd
ARG OBSERVIUM_DB_HOST=observiumdb
ARG OBSERVIUM_DB_USER=observium
ARG OBSERVIUM_DB_PASS=passw0rd
ARG OBSERVIUM_DB_NAME=observium

# set environment variables
ENV LANG en_US.UTF-8
ENV LANGUAGE en_US.UTF-8
ENV OBSERVIUM_DB_HOST=$OBSERVIUM_DB_HOST
ENV OBSERVIUM_DB_USER=$OBSERVIUM_DB_USER
ENV OBSERVIUM_DB_PASS=$OBSERVIUM_DB_PASS
ENV OBSERVIUM_DB_NAME=$OBSERVIUM_DB_NAME

# install prerequisites
RUN echo 'debconf debconf/frontend select Noninteractive' | debconf-set-selections
RUN apt-get update
RUN apt install -y libapache2-mod-php php-cli php-mysql php-gd php-json \
      php-pear snmp fping mysql-client python3-mysqldb rrdtool subversion whois mtr-tiny ipmitool \
      graphviz imagemagick apache2
RUN apt-get install -y libvirt-bin
RUN apt-get install -y cron supervisor wget locales
RUN apt-get install -y syslog-ng
RUN apt-get clean

# set locale
RUN locale-gen en_US.UTF-8

# install observium package
RUN mkdir -p /opt/observium /opt/observium/lock /opt/observium/logs /opt/observium/rrd
RUN cd /opt && \
    wget http://www.observium.org/observium-community-latest.tar.gz && \
    tar zxvf observium-community-latest.tar.gz && \
    rm observium-community-latest.tar.gz

# check version
RUN [ -f /opt/observium/VERSION ] && cat /opt/observium/VERSION

# configure observium package
RUN cd /opt/observium && \
    cp config.php.default config.php && \
    sed -i -e "s/= 'localhost';/= getenv('OBSERVIUM_DB_HOST');/g" config.php && \
    sed -i -e "s/= 'USERNAME';/= getenv('OBSERVIUM_DB_USER');/g" config.php && \
    sed -i -e "s/= 'PASSWORD';/= getenv('OBSERVIUM_DB_PASS');/g" config.php && \
    sed -i -e "s/= 'observium';/= getenv('OBSERVIUM_DB_NAME');/g" config.php && \
    echo "\$config['base_url'] = getenv('OBSERVIUM_BASE_URL');" >> config.php && \
    echo "\$config['enable_syslog'] = 1;" >> config.php

COPY observium-init /opt/observium/observium-init.sh
RUN chmod a+x /opt/observium/observium-init.sh

RUN chown -R www-data:www-data /opt/observium
RUN find /opt -ls
#RUN cd /opt/observium && \
#    ./discovery.php -u && \
#    ./adduser.php $OBSERVIUM_ADMIN_USER $OBSERVIUM_ADMIN_PASS 10

# configure php modules
RUN phpenmod mcrypt

# configure apache modules
RUN a2dismod mpm_event && \
    a2enmod mpm_prefork && \
    a2enmod php7.0 && \
    a2enmod rewrite

# configure apache configuration
#RUN mv /etc/apache2/sites-available/000-default.conf /etc/apache2/sites-available/000-default.conf.orig
COPY observium-apache24 /etc/apache2/sites-available/000-default.conf
RUN rm -fr /var/www

# configure Syslog-NG configuration
COPY observium-syslogng /etc/syslog-ng/syslog-ng.conf

# configure observium cron job
#COPY observium-cron /etc/cron.d/observium
COPY observium-cron /tmp/observium
RUN echo "" >> /etc/crontab && \
    cat /tmp/observium >> /etc/crontab && \
    rm -f /tmp/observium

# configure container interfaces
COPY supervisord.conf /etc/supervisor/conf.d/supervisord.conf
CMD ["/usr/bin/supervisord"]

# Add Syslog-NG staring in supervisor config
RUN echo "" >> /etc/supervisord.conf && \
    echo "[program:syslog-ng]" >> /etc/supervisord.conf && \
    echo "command=/bin/bash -c \"/usr/sbin/syslog-ng -F\"" >> /etc/supervisord.conf && \
    echo "stdout_logfile=/dev/stdout" >> /etc/supervisord.conf && \
    echo "stdout_logfile_maxbytes=0" >> /etc/supervisord.conf && \
    echo "stderr_logfile=/dev/stderr" >> /etc/supervisord.conf && \
    echo "stderr_logfile_maxbytes=0" >> /etc/supervisord.conf && \
    echo "user=root" >> /etc/supervisord.conf

EXPOSE 80/tcp
EXPOSE 514/udp

VOLUME ["/opt/observium/lock", "/opt/observium/logs","/opt/observium/rrd"]

