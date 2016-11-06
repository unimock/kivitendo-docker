#
#                    ##        .            
#              ## ## ##       ==            
#           ## ## ## ##      ===            
#       /""""""""""""""""\___/ ===        
#  ~~~ {~~ ~~~~ ~~~ ~~~~ ~~ ~ /  ===- ~~~   
#       \______ o          __/            
#         \    \        __/             
#          \____\______/                
# 
#          |          |
#       __ |  __   __ | _  __   _
#      /  \| /  \ /   |/  / _\ | 
#      \__/| \__/ \__ |\_ \__  |
#
FROM ubuntu:14.04

MAINTAINER Uni Mock <unimock@gate-m.de> version: 0.0

# Let the container know that there is no tty
ENV DEBIAN_FRONTEND noninteractive

# --- set timezone
ARG BUILD_TZ="Europe/Berlin"
RUN echo "$BUILD_TZ" > /etc/timezone && \
    dpkg-reconfigure -f noninteractive tzdata

#Packages 
RUN apt-get clean && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*
RUN apt-get update
RUN apt-get -y upgrade

# --- Install the SSH server
RUN apt-get -y install --no-install-recommends \
      openssh-server rsync git sendemail vim wget cron rsyslog

RUN DEBIAN_FRONTEND=noninteractive apt-get -y install apache2 libarchive-zip-perl libclone-perl \
    libconfig-std-perl libdatetime-perl libdbd-pg-perl libdbi-perl \
    libemail-address-perl  libemail-mime-perl libfcgi-perl libjson-perl \
    liblist-moreutils-perl libnet-smtp-ssl-perl libnet-sslglue-perl \
    libparams-validate-perl libpdf-api2-perl librose-db-object-perl \
    librose-db-perl librose-object-perl libsort-naturally-perl libpq5 \
    libstring-shellquote-perl libtemplate-perl libtext-csv-xs-perl \
    libtext-iconv-perl liburi-perl libxml-writer-perl libyaml-perl \
    libfile-copy-recursive-perl postgresql git build-essential \
    libgd-gd2-perl libimage-info-perl sed supervisor libgd-gd2-perl \
    libalgorithm-checkdigits-perl libapache2-mod-fcgid
RUN DEBIAN_FRONTEND=noninteractive apt-get -y install language-pack-de-base

#Install missing Perl Modules
RUN cpan HTML::Restrict
RUN cpan Image::Info

# ADD KIVITENDO
# Kivitendo intallation
ARG BUILD_KIVITENDO_VERSION="release-3.2.0"
RUN git clone https://github.com/kivitendo/kivitendo-erp.git /var/www/kivitendo-erp
RUN cd /var/www/kivitendo-erp && git checkout ${BUILD_KIVITENDO_VERSION}
ADD kivitendo.conf /var/www/kivitendo-erp/config/kivitendo.conf

#Check Kivitendo installation
RUN cd /var/www/kivitendo-erp/ && perl /var/www/kivitendo-erp/scripts/installation_check.pl


# ADD POSTGRES
# Add the PostgreSQL PGP key to verify their Debian packages.
# It should be the same key as https://www.postgresql.org/media/keys/ACCC4CF8.asc
RUN apt-key adv --keyserver keyserver.ubuntu.com --recv-keys B97B0AFCAA1A47F044F244A07FCC7D46ACCC4CF8

# Add PostgreSQL's repository. It contains the most recent stable release
RUN echo "deb http://apt.postgresql.org/pub/repos/apt/ precise-pgdg main" > /etc/apt/sources.list.d/pgdg.list

# Install ``python-software-properties``, ``software-properties-common`` and PostgreSQL

ARG BUILD_PG_VERSION="9.3"

RUN DEBIAN_FRONTEND=noninteractive apt-get update &&\
    apt-get install -y python-software-properties software-properties-common  \
    postgresql-${BUILD_PG_VERSION} postgresql-client-${BUILD_PG_VERSION} postgresql-contrib-${BUILD_PG_VERSION}

# Run the rest of the commands as the ``postgres`` user created by the ``postgres-${BUILD_PG_VERSION}`` package when it was ``apt-get installed``
USER postgres

# Set the locale of the db cluster
ARG BUILD_LOCALE="de_DE"
RUN pg_dropcluster --stop ${BUILD_PG_VERSION} main && pg_createcluster --locale ${BUILD_LOCALE}.UTF-8 --start ${BUILD_PG_VERSION} main

# Create a PostgreSQL role named ``docker`` with ``docker`` as the password and
# then create a database `docker` owned by the ``docker`` role.
ARG BUILD_PG_PASSWD="docker"
RUN    /etc/init.d/postgresql start &&\
    psql --command "CREATE USER docker WITH SUPERUSER PASSWORD '${BUILD_PG_PASSWD}';" &&\
    psql --command "CREATE USER kivitendo WITH SUPERUSER PASSWORD '${BUILD_PG_PASSWD}';" &&\
    createdb -O docker docker &&\
    createdb -O kivitendo kivitendo

# Adjust PostgreSQL configuration so that remote connections to the
# database are possible. 
RUN echo "host	all	all	0.0.0.0/0	md5" >> /etc/postgresql/${BUILD_PG_VERSION}/main/pg_hba.conf

# And add ``listen_addresses`` to ``/etc/postgresql/${BUILD_PG_VERSION}/main/postgresql.conf``
RUN sed -ri "s/^#(listen_addresses\s*=\s*)\S+/\1'*'/" "/etc/postgresql/${BUILD_PG_VERSION}/main/postgresql.conf"

# Expose the PostgreSQL port
EXPOSE 5432


# ADD APACHE
# Run the rest of the commands as the ``root`` user
USER root

RUN mkdir -p /var/lock/apache2 /var/run/apache2 /var/run/sshd

# SET Servername to localhost
RUN echo "ServerName localhost" >> /etc/apache2/conf-available/servername.conf
RUN a2enconf servername

# Manually set up the apache environment variables
ENV APACHE_RUN_USER www-data
ENV APACHE_RUN_GROUP www-data
ENV APACHE_LOG_DIR /var/log/apache2
ENV APACHE_LOCK_DIR /var/lock/apache2
ENV APACHE_PID_FILE /var/run/apache2.pid
 
RUN chown -R www-data:www-data /var/www
RUN chmod u+rwx,g+rx,o+rx /var/www
RUN find /var/www -type d -exec chmod u+rwx,g+rx,o+rx {} +
RUN find /var/www -type f -exec chmod u+rw,g+rw,o+r {} +

#Kivitendo rights
RUN mkdir /var/www/kivitendo-erp/webdav
RUN chown -R www-data /var/www/kivitendo-erp/users /var/www/kivitendo-erp/spool /var/www/kivitendo-erp/webdav
RUN chown www-data /var/www/kivitendo-erp/templates /var/www/kivitendo-erp/users
RUN chmod -R +x /var/www/kivitendo-erp/

#Perl Modul im Apache laden
RUN a2enmod cgi.load
RUN a2enmod fcgid
EXPOSE 80
 
# Update the default apache site with the config we created.
ADD apache-config.conf /etc/apache2/sites-enabled/000-default.conf
 
# Add VOLUMEs to allow backup of config, logs and databases
VOLUME  ["/etc/postgresql", "/var/log/postgresql", "/var/lib/postgresql", "/var/log/apache2"]

############################################################################################
#
# docker-extensions
#
RUN mkdir -p  /usr/local/bin
COPY ./bin/*  /usr/local/bin/
RUN chmod a+x /usr/local/bin/*

#
# establisch supervisord
#
ADD /supervisor /etc/supervisor
# link old /etc/init.d/ startup scripts to supervisor
RUN ls -m1    /etc/supervisor/services.d | while read i; do mv /etc/init.d/$i /etc/init.d/$i-orig ;  ln -sf /etc/supervisor/super-init.sh /etc/init.d/$i ; done
RUN ln -sf    /etc/supervisor/systemctl /bin/systemctl
RUN chmod a+x /etc/supervisor/* /etc/supervisor/*.d/*
COPY /supervisor/invoke-rc.d /usr/sbin/invoke-rc.d
COPY /supervisor/reload      /sbin/reload
RUN  chmod a+x /sbin/reload /usr/sbin/invoke-rc.d
#
# create directory for service volume
#
RUN mkdir -p /service ; chmod a+rwx /service
ADD track.gitignore /.gitignore

ENV TERM xterm
#
# startup script
#
ADD ./start.sh /start.sh
RUN chmod 755 /start.sh
CMD ["/start.sh"]


