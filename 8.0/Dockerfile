FROM alpine
# add our user and group first to make sure their IDs get assigned consistently, regardless of whatever dependencies get added
RUN addgroup -S -g 1000 mysql && adduser -D -S -G mysql mysql
# -S: system group
# -g 1000: group id is 1000
# -D: do not assign a password
# -H: Do not create home directory (not used)
# -S: Create a system user
# -G mysql: add to group named "mysql"

# add gosu for easy step-down from root
ENV GOSU_VERSION 1.7
RUN set -x \
	&& apk update && apk add ca-certificates wget dpkg gnupg && rm -rf /var/lib/apk/* 
RUN wget -O /usr/local/bin/gosu "https://github.com/tianon/gosu/releases/download/$GOSU_VERSION/gosu-$(dpkg --print-architecture|awk -F'-' '{print $NF}')" \
	&& wget -O /usr/local/bin/gosu.asc "https://github.com/tianon/gosu/releases/download/$GOSU_VERSION/gosu-$(dpkg --print-architecture|awk -F'-' '{print $NF}').asc" \
	&& export GNUPGHOME="$(mktemp -d)" \
	&& gpg --keyserver ha.pool.sks-keyservers.net --recv-keys B42F6819007F00F88E364FD4036A9C25BF357DD4 \
	&& gpg --batch --verify /usr/local/bin/gosu.asc /usr/local/bin/gosu \
	&& rm -r "$GNUPGHOME" /usr/local/bin/gosu.asc \
	&& chmod +x /usr/local/bin/gosu \
	&& gosu nobody true \
	&& apk del ca-certificates wget dpkg

RUN mkdir /docker-entrypoint-initdb.d

RUN apk update && apk add \
# for MYSQL_RANDOM_ROOT_PASSWORD
		pwgen \
# for mysql_ssl_rsa_setup
		openssl \
# FATAL ERROR: please install the following Perl modules before executing /usr/local/mysql/scripts/mysql_install_db:
# File::Basename
# File::Copy
# Sys::Hostname
# Data::Dumper
		perl \
	&& rm -rf /var/lib/apk/*


#########################################################
#	 Yet to find out what this is for 		#
#########################################################
# RUN set -ex; \
# gpg: key 5072E1F5: public key "MySQL Release Engineering <mysql-build@oss.oracle.com>" imported
#	key='A4A9406876FCBD3C456770C88C718D3B5072E1F5'; \
#	export GNUPGHOME="$(mktemp -d)"; \
#	gpg --keyserver ha.pool.sks-keyservers.net --recv-keys "$key"; \
#	gpg --export "$key" > /etc/apt/trusted.gpg.d/mysql.gpg; \
#	rm -r "$GNUPGHOME"; \
#	apk list > /dev/null

ENV MYSQL_MAJOR 8.0
ENV MYSQL_VERSION 8.0.1-dmr-1debian8

#########################################################
#		We probably dont need it 		#
#########################################################
# RUN echo "deb x jessie mysql-${MYSQL_MAJOR}" > /etc/apk/sources.list.d/mysql.list



#########################################################
#	Debconf is not available on alpine 		#
#########################################################
RUN apk add debconf
# the "/var/lib/mysql" stuff here is because the mysql-server postinst doesn't have an explicit way to disable the mysql_install_db codepath besides having a database already "configured" (ie, stuff in /var/lib/mysql/mysql)
# also, we set debconf keys to make APT a little quieter
RUN { \
		echo mysql-community-server mysql-community-server/data-dir select ''; \
		echo mysql-community-server mysql-community-server/root-pass password ''; \
		echo mysql-community-server mysql-community-server/re-root-pass password ''; \
		echo mysql-community-server mysql-community-server/remove-test-db select false; \
	} | debconf-set-selections \
	&& apk update && apk add mysql-community-client-core="${MYSQL_VERSION}" mysql-community-server-core="${MYSQL_VERSION}" && rm -rf /var/lib/apk/* \
	&& rm -rf /var/lib/mysql && mkdir -p /var/lib/mysql /var/run/mysqld \
	&& chown -R mysql:mysql /var/lib/mysql /var/run/mysqld \
# ensure that /var/run/mysqld (used for socket and lock files) is writable regardless of the UID our mysqld instance ends up having at runtime
	&& chmod 777 /var/run/mysqld

VOLUME /var/lib/mysql
# Config files
COPY config/ /etc/mysql/
COPY docker-entrypoint.sh /usr/local/bin/
RUN ln -s usr/local/bin/docker-entrypoint.sh /entrypoint.sh # backwards compat
ENTRYPOINT ["docker-entrypoint.sh"]

EXPOSE 3306
CMD ["mysqld"]
