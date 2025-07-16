ARG ALPINE_VERSION=3.21
ARG PHP_VERSION=8.3

FROM alpine:${ALPINE_VERSION}
FROM php:${PHP_VERSION}-fpm-alpine

ARG PHP_MODULE="php83"

# Setup unique user and group - needs bash
RUN apk add --no-cache bash

ARG UID
ARG GID
ENV UID=${UID}
ENV GID=${GID}

# Create user and group
RUN addgroup -S --gid ${GID} myuser \
    && adduser -S --uid ${UID} myuser -s /bin/sh -G myuser myuser \
    && apk add --no-cache sudo \
    && chmod u+s /bin/su \
    && echo "myuser ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers

# Install main packages and remove default server definition
RUN apk add --no-cache \
  openrc \
  curl \
  wget \
  openssl \
  nginx \
  zip \
  nano \
  git \
  #ffmpeg \
  libva-dev \
  supervisor

RUN set -xe \
    && apk add --no-cache --virtual .build-deps \
        libzip-dev \
        freetype-dev \
        icu-dev \
        libmcrypt-dev \
        libjpeg-turbo-dev \
        libpng-dev \
        libxslt-dev \
        patch \
        openssh-client

# Install PHP and its extensions packages and remove default server definition
ENV PHP_V=$PHP_MODULE

RUN apk add --no-cache \
  ${PHP_V} \
  ${PHP_V}-cli \
  ${PHP_V}-ctype \
  ${PHP_V}-curl \
  ${PHP_V}-dom \
  ${PHP_V}-fileinfo \
  ${PHP_V}-fpm \
  ${PHP_V}-gd \
  ${PHP_V}-intl \
  ${PHP_V}-mbstring \
  ${PHP_V}-opcache \
  ${PHP_V}-openssl \
  ${PHP_V}-phar \
  ${PHP_V}-session \
  ${PHP_V}-tokenizer \
  ${PHP_V}-soap \
  ${PHP_V}-xml \
  ${PHP_V}-xmlreader \
  ${PHP_V}-xmlwriter \
  ${PHP_V}-simplexml \
  # Databases
  ${PHP_V}-pdo \
  ${PHP_V}-pdo_sqlite \
  ${PHP_V}-sqlite3 \
  ${PHP_V}-pdo_mysql \
  ${PHP_V}-mysqlnd \
  ${PHP_V}-mysqli \
  #${PHP_V}-pgsql \
  #${PHP_V}-pdo_pgsql \
  #${PHP_V}-mongodb \
  #${PHP_V}-redis \
  # Postgres
  #postgresql-dev \
  # Last
  ${PHP_V}-zip

# Install PHP required extensions from Docker - add if needed: pdo_pgsql
RUN docker-php-ext-install zip gd pdo pdo_mysql

# PHP required PECL extensions - (optional: for XDebug uncomment line to use it and add a backslash in previous line)
RUN apk add \
  ${PHP_V}-pecl-amqp
  #${PHP_V}-pecl-xdebug

# Configure NGINX and remove created its default html directory
COPY config/nginx/nginx.conf /etc/nginx/nginx.conf
COPY config/nginx/conf.d /etc/nginx/conf.d/
RUN rm -rf /var/www/html

# Configure PHP-FPM
COPY config/php/fpm-pool.conf /etc/${PHP_V}/php-fpm.d/www.conf
COPY config/php/php.ini /etc/${PHP_V}/conf.d/custom.ini

# Create symlink for php
RUN ln -s -f /usr/bin/${PHP_V} /usr/bin/php

# Configure Supervisor
COPY config/supervisor/supervisord.conf /etc/supervisor/supervisord.conf
COPY config/supervisor/conf.d/. /etc/supervisor/conf.d/

# Create predifined Crontab and apply file permissions
COPY config/crontab /var/spool/cron/crontabs/root
RUN chmod -R 0644 /var/spool/cron/crontabs/root

# PHP Composer
COPY --from=composer:latest /usr/bin/composer /usr/local/bin/composer

## Configure XDebug - (optional - uncomment line to use it)
#COPY config/php/xdebug.ini /etc/${PHP_V}/conf.d/xdebug.ini

# Create project root and set it as work directory
RUN mkdir -p /var/www
WORKDIR /var/www

# Add application from docker-compose.yml -> volumes: source
COPY --chown=myuser:myuser ../ /var/www

# Remove unused directories
RUN rm -rf /var/www/html /var/www/localhost

# Make sure files/folders needed by the processes are accessable when they run under the nobody user
RUN chown -R myuser:myuser /var/www /run /var/lib/nginx /var/log/nginx

# Switch to use a non-root user from here on
USER myuser

# Expose port/s Nginx will listen - Add more as required: (9003 - Xdebug) - Removed for testing
EXPOSE 80 443

# Let supervisord start nginx & php-fpm
CMD ["/usr/bin/supervisord", "-c", "/etc/supervisor/supervisord.conf"]
