FROM php:8.2-fpm

COPY --from=composer:2.5 /usr/bin/composer /usr/bin/composer

RUN apt update
RUN curl -sL https://deb.nodesource.com/setup_19.x | bash -
RUN apt install -y vim less git zip unzip nodejs
RUN docker-php-ext-install pdo_mysql

WORKDIR "/var/www/app"