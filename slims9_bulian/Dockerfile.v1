FROM php:7.4-apache
# For SLiMS 9 version (Bulian) with https support
MAINTAINER Hendro Wicaksono <hendrowicaksono@gmail.com>

## ENVs
ENV SLIMS_VERSION 9.1.1
ENV SLIMS_MD5 1df72c24a47c37eb4dcac4f0874ab9e4

RUN set -eux; \
	apt-get update; \
	apt-get install -y --no-install-recommends \
		unzip \
		gettext \
		mariadb-client \
	; \
	rm -rf /var/lib/apt/lists/*
RUN set -ex; \
	\
	savedAptMark="$(apt-mark showmanual)"; \
	\
	apt-get update; \
	apt-get install -y --no-install-recommends \
		libfreetype6-dev \
		libjpeg-dev \
		libmagickwand-dev \
		libpng-dev \
		libzip-dev \
		#libmcrypt-dev \
		#libyaz4-dev \
		#yaz \
	; \
	\
	#docker-php-ext-configure gd --with-freetype-dir=/usr --with-jpeg-dir=/usr --with-png-dir=/usr; \
	docker-php-ext-configure gd --with-freetype --with-jpeg; \
	docker-php-ext-install -j "$(nproc)" \
		bcmath \
		exif \
		gd \
		mysqli \
		opcache \
		zip \
		gettext \
		#mcrypt \
		#mbstring \
		#soap \
	; \
	#pecl install imagick-3.4.4 yaz-1.2.3 ; \
	pecl install imagick-3.4.4 ; \
	pear install File_MARC-1.4.1 \
	docker-php-ext-enable imagick ; \
	apt-mark auto '.*' > /dev/null; \
	apt-mark manual $savedAptMark; \
	ldd "$(php -r 'echo ini_get("extension_dir");')"/*.so \
		| awk '/=>/ { print $3 }' \
		| sort -u \
		| xargs -r dpkg-query -S \
		| cut -d: -f1 \
		| sort -u \
		| xargs -rt apt-mark manual; \
	\
	apt-get purge -y --auto-remove -o APT::AutoRemove::RecommendsImportant=false; \
	rm -rf /var/lib/apt/lists/*
RUN { \
		echo 'opcache.memory_consumption=128'; \
		echo 'opcache.interned_strings_buffer=8'; \
		echo 'opcache.max_accelerated_files=4000'; \
		echo 'opcache.revalidate_freq=2'; \
		echo 'opcache.fast_shutdown=1'; \
	} > /usr/local/etc/php/conf.d/opcache-recommended.ini
RUN { \
		echo 'error_reporting = E_ERROR | E_WARNING | E_PARSE | E_CORE_ERROR | E_CORE_WARNING | E_COMPILE_ERROR | E_COMPILE_WARNING | E_RECOVERABLE_ERROR'; \
		echo 'display_errors = Off'; \
		echo 'display_startup_errors = Off'; \
		echo 'log_errors = On'; \
		echo 'error_log = /dev/stderr'; \
		echo 'log_errors_max_len = 1024'; \
		echo 'ignore_repeated_errors = On'; \
		echo 'ignore_repeated_source = Off'; \
		echo 'html_errors = Off'; \
	} > /usr/local/etc/php/conf.d/error-logging.ini

RUN pecl install redis && docker-php-ext-enable redis
#RUN pecl install yaz-1.2.3 && docker-php-ext-enable yaz

RUN set -eux; \
	a2enmod rewrite expires; \
	\
	cd /var/www/html && curl -fSL "https://github.com/slims/slims9_bulian/archive/v${SLIMS_VERSION}.zip" -o slims.zip; \
	echo "${SLIMS_MD5} *slims.zip" | md5sum -c -; \
	unzip slims.zip; \
	echo "Download done."; \
	mv slims9_bulian-9.1.1 slims; \
    #chmod -R 777 slims/config slims/images slims/files slims/repository; \
    chown -R www-data:www-data slims/config slims/images slims/files slims/repository slims/config

RUN openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/ssl/private/ssl-cert-snakeoil.key -out /etc/ssl/certs/ssl-cert-snakeoil.pem -subj "/C=AT/ST=Vienna/L=Vienna/O=Security/OU=Development/CN=example.com"

RUN a2enmod rewrite
RUN a2enmod rewrite
RUN a2ensite default-ssl
RUN a2enmod ssl