FROM debian:12-slim

ENV TZ UTC
SHELL ["/bin/bash", "-o", "pipefail", "-c"]

ARG DEBIAN_FRONTEND=noninteractive
RUN apt-get update && \
	apt-get install --no-install-recommends -y \
	ca-certificates cron \
	apache2 libapache2-mod-php \
	libapache2-mod-auth-openidc \
	php-curl php-gmp php-intl php-mbstring php-xml php-zip \
	php-sqlite3 php-mysql php-pgsql openjdk-17-jre-headless wget nano unzip && \
	rm -rf /var/lib/apt/lists/*
 

RUN mkdir -p /var/www/FreshRSS/ /run/apache2/ /var/www/FreshRSS/i2p/start.sh
WORKDIR /var/www/FreshRSS



COPY . /var/www/FreshRSS
COPY ./Docker/*.Apache.conf /etc/apache2/sites-available/

ARG FRESHRSS_VERSION
ARG SOURCE_COMMIT

LABEL \
	org.opencontainers.image.authors="Alkarex" \
	org.opencontainers.image.description="A self-hosted RSS feed aggregator" \
	org.opencontainers.image.documentation="https://freshrss.github.io/FreshRSS/" \
	org.opencontainers.image.licenses="AGPL-3.0" \
	org.opencontainers.image.revision="${SOURCE_COMMIT}" \
	org.opencontainers.image.source="https://github.com/FreshRSS/FreshRSS" \
	org.opencontainers.image.title="FreshRSS" \
	org.opencontainers.image.url="https://freshrss.org/" \
	org.opencontainers.image.vendor="FreshRSS" \
	org.opencontainers.image.version="$FRESHRSS_VERSION"

RUN a2dismod -q -f alias autoindex negotiation status && \
	a2dismod -q auth_openidc && \
	a2enmod -q deflate expires headers mime remoteip setenvif && \
	a2disconf -q '*' && \
	a2dissite -q '*' && \
	a2ensite -q 'FreshRSS*'



RUN sed -r -i "/^\s*(CustomLog|ErrorLog|Listen) /s/^/#/" /etc/apache2/apache2.conf && \
	sed -r -i "/^\s*Listen /s/^/#/" /etc/apache2/ports.conf && \
	# Disable built-in updates when using Docker, as the full image is supposed to be updated instead.
	sed -r -i "\\#disable_update#s#^.*#\t'disable_update' => true,#" ./config.default.php && \
	touch /var/www/FreshRSS/Docker/env.txt && \
	echo "7,37 * * * * . /var/www/FreshRSS/Docker/env.txt; \
 		su root -s /bin/sh -c '/var/www/FreshRSS/i2p/i2prouter start >> /i2prouter.log' \
