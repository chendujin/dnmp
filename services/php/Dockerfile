###########################################################################
# PHP-FPM Version Of Alpine:
###########################################################################
ARG PHP_VERSION
FROM ${PHP_VERSION}

LABEL maintainer="Thomas <13728656748@163.com>"

ARG TZ
ARG PHP_EXTENSIONS
ARG CONTAINER_PACKAGE_URL

###########################################################################
# Alpine Mirrors:
###########################################################################
RUN if [ $CONTAINER_PACKAGE_URL ] ; then sed -i "s/dl-cdn.alpinelinux.org/${CONTAINER_PACKAGE_URL}/g" /etc/apk/repositories ; fi

###########################################################################
# System Extensions:
###########################################################################

RUN apk --update add \
        autoconf \
        build-base \
        linux-headers \
        libaio-dev \
        zlib-dev \
        curl \
        git \
        freetype-dev \
        libjpeg-turbo-dev \
        libmcrypt-dev \
        libpng-dev \
        libtool \
        libbz2 \
        bzip2 \
        bzip2-dev \
        libstdc++ \
        libxslt-dev \
        openldap-dev \
        imagemagick-dev \
        libzip-dev \
        make \
        zip \
        unzip \
        wget \
        libmemcached-dev


###########################################################################
# PHP Core Extensions:
###########################################################################

# Core Extensions For Default Installation:
#RUN docker-php-ext-install \
#        bcmath \
#        pdo_mysql \
#        mysqli \
#        sockets \
#    && docker-php-ext-configure gd --with-freetype-dir=/usr/include/ --with-jpeg-dir=/usr/include/ \
#    && docker-php-ext-install gd
RUN docker-php-ext-install \
        bcmath \
        pdo_mysql \
        mysqli \
        sockets \
    && docker-php-ext-configure gd --with-freetype --with-jpeg \
    && docker-php-ext-install -j$(nproc) gd


# Core Extensions pcntl:
ARG INSTALL_PCNTL=false
RUN if [ ${INSTALL_PCNTL} = true ]; then \
    docker-php-ext-install pcntl \
;fi


# Core Extensions opcache:
ARG INSTALL_OPCACHE=false
RUN if [ ${INSTALL_OPCACHE} = true ]; then \
    docker-php-ext-install opcache \
;fi


# Core Extensions zip:
ARG INSTALL_ZIP=false
RUN if [ ${INSTALL_ZIP} = true ]; then \
    if [ ${PHP_VERSION} = "7.3" ] || [ ${PHP_VERSION} = "7.4" ]; then \
        docker-php-ext-configure zip \
    else \
        docker-php-ext-configure zip --with-libzip \
    ;fi \
    && docker-php-ext-install zip \
;fi


###########################################################################
# PHP PECL Extensions:
###########################################################################

# PECL Extensions reids:
ARG INSTALL_REDIS=false
ARG INSTALL_REDIS_VERSION
RUN if [ ${INSTALL_REDIS} = true ]; then \
    pecl install redis-${INSTALL_REDIS_VERSION} \
    && rm -rf /tmp/pear \
    && docker-php-ext-enable redis \
;fi


# PECL Extensions mongodb:
ARG INSTALL_MONGODB=false
ARG INSTALL_MONGODB_VERSION
RUN if [ ${INSTALL_MONGODB} = true ]; then \
    pecl install mongodb-${INSTALL_MONGODB_VERSION} \
    && rm -rf /tmp/pear \
    && docker-php-ext-enable mongodb \
;fi


# PECL Extensions memcached:
ARG INSTALL_MEMCACHED=false
ARG INSTALL_MEMCACHED_VERSION
RUN if [ ${INSTALL_MEMCACHED} = true ]; then \
    pecl install memcached-${INSTALL_MEMCACHED_VERSION} \
    && rm -rf /tmp/pear \
    && docker-php-ext-enable memcached \
;fi


# PECL Extensions swoole:
ARG INSTALL_SWOOLE=false
ARG INSTALL_SWOOLE_VERSION
RUN if [ ${INSTALL_SWOOLE} = true ]; then \
    pecl install swoole-${INSTALL_SWOOLE_VERSION} \
    && rm -rf /tmp/pear \
    && docker-php-ext-enable swoole \
;fi


###########################################################################
# PHP Thrid Extensions:
###########################################################################

# Thrid Extensions Xunsearch:
ARG INSTALL_XUNSEARCH=false
RUN if [ ${INSTALL_XUNSEARCH} = true ]; then \
    curl 'http://www.xunsearch.com/scws/down/scws-1.2.3.tar.bz2' -o scws.tar.bz2 \
    && tar xvjf scws.tar.bz2 \
    && cd scws-1.2.3 \
    && ./configure --prefix=/usr/local/scws \
    && make \
    && make install \
    && cd phpext \
    && phpize \
    && ./configure --with-scws=/usr/local/scws \
    && make \
    && make install \
    && cd ../../ \
    && rm -rf scws scws-1.2.3.tar.bz2 \
    && docker-php-ext-enable scws \
    && echo "scws.default.charset = utf8" >> /usr/local/etc/php/conf.d/docker-php-ext-scws.ini \
    && echo "scws.default.fpath = /usr/local/scws/etc" >> /usr/local/etc/php/conf.d/docker-php-ext-scws.ini \
;fi


# Composer:
ARG INSTALL_COMPOSER=false
RUN if [ ${INSTALL_COMPOSER} = true ]; then \
    curl -sS http://getcomposer.org/installer | php \
    && mv ./composer.phar /usr/local/bin/composer \
    && composer config -g repo.packagist composer https://mirrors.aliyun.com/composer \
;fi

RUN apk del build-base \
    	linux-headers \
    	libaio-dev \
    && rm -rf /var/cache/apk/*

# php image's www-data user uid & gid are 82, change them to 1000 (primary user)
RUN apk --no-cache add shadow && usermod -u 1000 www-data && groupmod -g 1000 www-data


WORKDIR /www
