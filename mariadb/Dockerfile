FROM mariadb:10.11
LABEL maintainer="Juan Luis Baptiste <juan.baptiste@gmail.com>"
ENV MARIADB_ROOT_PASSWORD changeme
# Backward compatibility with old ENV name
ENV MYSQL_ROOT_PASSWORD ${MARIADB_ROOT_PASSWORD}

#mariadb image runs as mysql user (uid 27) but we need to do some configuration
#changes so we need to temporarly switch to root
USER root

#Change db configuration as required by official install docs and Enable utf8 support
RUN echo "[mysqld]" > /etc/mysql/conf.d/otrs.cnf &&\
    echo "max_allowed_packet=64M" >> /etc/mysql/conf.d/otrs.cnf &&\
    echo "query_cache_size=32M" >> /etc/mysql/conf.d/otrs.cnf &&\
    echo "innodb_log_file_size=256M" >> /etc/mysql/conf.d/otrs.cnf &&\
    echo "skip-character-set-client-handshake" >> /etc/mysql/conf.d/otrs.cnf &&\
    echo "init-connect='SET NAMES utf8mb3'" >> /etc/mysql/conf.d/otrs.cnf &&\
    echo "init_connect='SET collation_connection = utf8mb3_unicode_ci'" >> /etc/mysql/conf.d/otrs.cnf &&\
    echo "collation-server = utf8mb3_general_ci" >> /etc/mysql/conf.d/otrs.cnf &&\
    echo "character-set-server = utf8mb3" >> /etc/mysql/conf.d/otrs.cnf
