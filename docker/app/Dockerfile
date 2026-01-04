FROM php:8.4.16-fpm-bookworm

COPY --from=composer:2.9.3 /usr/bin/composer /usr/bin/composer

RUN apt update
RUN apt install -y procps vim less git zip unzip

# MySQL
RUN docker-php-ext-install pdo_mysql

# node
RUN curl -sL https://deb.nodesource.com/setup_22.x | bash -
RUN apt install -y nodejs

# xdebug
RUN pecl install xdebug-3.5.0
RUN docker-php-ext-enable xdebug

# phpredis
RUN pecl install redis \
 && docker-php-ext-enable redis

COPY ./xdebug/php-xdebug.ini /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini

WORKDIR "/var/www/app"