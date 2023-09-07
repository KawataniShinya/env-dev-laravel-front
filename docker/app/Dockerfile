FROM php:8.2-fpm

COPY --from=composer:2.5 /usr/bin/composer /usr/bin/composer

RUN apt update
RUN curl -sL https://deb.nodesource.com/setup_19.x | bash -
RUN apt install -y procps vim less git zip unzip nodejs
RUN docker-php-ext-install pdo_mysql

RUN pecl install xdebug-3.2.2
RUN docker-php-ext-enable xdebug

COPY ./xdebug/php-xdebug.ini /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini

WORKDIR "/var/www/app"