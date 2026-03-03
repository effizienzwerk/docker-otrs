#OTRS ticketing system docker image.
FROM ubuntu:22.04
LABEL maintainer="Juan Luis Baptiste <juan@juanbaptiste.tech>"
ENV OTRS_VERSION=6.0.45
ENV OTRS_ROOT "/opt/otrs/"
ENV OTRS_BACKUP_DIR "/var/otrs/backups"
ENV OTRS_CONFIG_DIR "${OTRS_ROOT}Kernel"
ENV OTRS_CONFIG_MOUNT_DIR "/Kernel/"
ENV OTRS_SKINS_MOUNT_DIR "/skins/"
ENV SKINS_PATH "${OTRS_ROOT}/var/httpd/htdocs/skins/"
ENV LANG=en_US.UTF-8
ENV LANGUAGE=en_US.UTF-8
ENV DEBIAN_FRONTEND=noninteractive

RUN apt-get update && \
    apt-get install -y --no-install-recommends \
    apache2 \
    bzip2 \
    ca-certificates \
    cron \
    curl \
    debianutils \
    libapache2-mod-perl2 \
    libapache2-reload-perl \
    libarchive-zip-perl \
    libauthen-ntlm-perl \
    libauthen-sasl-perl \
    libcrypt-eksblowfish-perl \
    libcrypt-ssleay-perl \
    libcss-minifier-xs-perl \
    libdatetime-perl \
    libdbd-mysql-perl \
    libencode-hanextra-perl \
    libexporter-tiny-perl \
    libgd-graph-perl \
    libgd-perl \
    libgd-text-perl \
    libio-socket-ssl-perl \
    libjavascript-minifier-xs-perl \
    libjson-xs-perl \
    libmail-imapclient-perl \
    libmath-random-isaac-perl \
    libmath-random-secure-perl \
    libmodule-find-perl \
    libnamespace-clean-perl \
    libnet-dns-perl \
    libnet-ldap-perl \
    libpdf-api2-perl \
    libtemplate-perl \
    libtext-csv-xs-perl \
    libtype-tiny-perl \
    liburi-perl \
    libwww-perl \
    libxml-libxml-perl \
    libxml-libxslt-perl \
    libxml-parser-perl \
    libyaml-libyaml-perl \
    mysql-client \
    perl \
    procmail \
    rsyslog \
    supervisor \
    tar \
    wget \
    locales && \
    locale-gen en_US.UTF-8 && \
    wget https://download.znuny.org/releases/znuny-${OTRS_VERSION}.tar.gz -O /tmp/znuny.tar.gz && \
    mkdir -p /opt/otrs && \
    tar -xzf /tmp/znuny.tar.gz -C /opt/otrs --strip-components=1 && \
    cp /opt/otrs/Kernel/Config.pm.dist /opt/otrs/Kernel/Config.pm && \
    useradd -r -m -d /opt/otrs -c "OTRS user" -G www-data otrs && \
    /opt/otrs/bin/otrs.CheckModules.pl || true && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/* /tmp/znuny.tar.gz

# Add scripts and function files
COPY *.sh /
#Supervisord configuration
COPY etc/supervisord.d/otrs.ini /etc/supervisord.d/
RUN chmod 755 /*.sh  && \
    cp ${OTRS_ROOT}/var/httpd/htdocs/index.html /var/www/html && \
    chmod 644 /var/www/html/index.html && \
    ln -s /opt/otrs/scripts/apache2-httpd.include.conf /etc/apache2/conf-available/otrs.conf && \
    a2enconf otrs && \
    a2enmod cgi && \
    a2enmod perl && \
    a2enmod headers && \
    a2enmod rewrite && \
    sed -i 's/DirectoryIndex index.html/DirectoryIndex index.pl index.html/' /etc/apache2/mods-enabled/dir.conf && \
    echo "+ : otrs : cron crond" |cat >> /etc/security/access.conf                && \
    sed -i -e '/pam_loginuid.so/ s/^#*/#/' /etc/pam.d/cron                       && \
    sed -i -e "s/^nodaemon=false/nodaemon=true/" /etc/supervisor/supervisord.conf && \
    cp /etc/supervisord.d/otrs.ini /etc/supervisor/conf.d/otrs.conf && \
    sed -i -e '/<ValidateModule>Kernel::System::SysConfig::StateValidate<\/ValidateModule>/ s/^#*/#/' \
        ${OTRS_ROOT}Kernel/Config/Files/XML/Ticket.xml  && \
    mkdir -p ${OTRS_ROOT}var/{run,tmp}/ && \
    sed -i -e '/\$ModLoad imjournal/ s/^#*/#/' /etc/rsyslog.conf && \
    sed -i -e '/\$IMJournalStateFile imjournal.state/ s/^#*/#/' /etc/rsyslog.conf && \
    sed -i 's/\(^\$OmitLocalLogging \).*/\1off/' /etc/rsyslog.conf
#To be able to use a host-mounted volume for OTRS configuration we need to move
#away the contents of ${OTRS_CONFIG_DIR} to another place and move them back
#on first container run (see check_host_mount_dir on functions.sh), after the
#host-volume is mounted. The same for the skins.
RUN mv /opt/otrs/Kernel /Kernel && \
    mv /opt/otrs/var/httpd/htdocs/skins /skins && \
    touch ${OTRS_ROOT}var/tmp/firsttime && \
    ln -sf /dev/stdout /var/log/apache2/access.log && \
    ln -sf /dev/stderr /var/log/apache2/error.log && \
    ln -sf /dev/stdout /var/log/cron
EXPOSE 80
CMD ["/run.sh"]
