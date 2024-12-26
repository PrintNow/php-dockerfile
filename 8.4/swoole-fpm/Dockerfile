FROM php:8.4.2-fpm-alpine3.21

ARG TIMEZONE
ENV TZ=${TIMEZONE:-"Asia/Shanghai"} \
  LANG="C.UTF-8"

RUN set -ex \
  && addgroup -S -g 1000 www \
  && adduser -S -D -u 1000 -s /sbin/nologin -G www www

RUN set -ex \
  && curl -sfL https://getcomposer.org/installer | php -- --install-dir=/usr/bin --filename=composer \
  && chmod +x /usr/bin/composer \
  && apk update \
  && apk add --no-cache libstdc++ \
  && apk add --no-cache --virtual .build-deps $PHPIZE_DEPS curl-dev linux-headers openssl-dev pcre-dev pcre2-dev zlib-dev \
  && apk add --no-cache tzdata librdkafka-dev icu-dev libzip-dev \
  && pecl install --configureoptions 'enable-redis-igbinary="no" enable-redis-lzf="no" enable-redis-zstd="no"' redis \
  && pecl install inotify \
  && pecl install rdkafka \
  && docker-php-ext-enable redis rdkafka inotify \
  && docker-php-ext-install bcmath intl pdo_mysql opcache zip pcntl \
  && docker-php-ext-install sockets \
  && docker-php-source extract \
  && mkdir /usr/src/php/ext/swoole \
  && curl -sfL https://github.com/swoole/swoole-src/archive/v6.0.0.tar.gz -o swoole.tar.gz \
  && tar xfz swoole.tar.gz --strip-components=1 -C /usr/src/php/ext/swoole \
  && docker-php-ext-configure swoole \
    --enable-mysqlnd \
    --enable-openssl \
    --enable-sockets --enable-swoole-curl \
  && docker-php-ext-install -j$(nproc) swoole \
  && rm -f swoole.tar.gz $HOME/.composer/*-old.phar \
  && apk add --no-cache supervisor \
  && mkdir -p /var/log/supervisor \
  && docker-php-source delete \
  && apk del .build-deps
