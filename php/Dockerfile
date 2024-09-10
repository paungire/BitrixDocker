FROM node:latest AS node
FROM php:8.3-fpm

# настройка node-js
COPY --from=node /usr/local/lib/node_modules /usr/local/lib/node_modules
COPY --from=node /usr/local/bin/node /usr/local/bin/node
RUN ln -s /usr/local/lib/node_modules/npm/bin/npm-cli.js /usr/local/bin/npm

# настройка php
RUN apt-get update
RUN docker-php-ext-install pdo pdo_mysql mysqli
RUN apt-get update -y && apt-get install -y sendmail libpng-dev
RUN apt-get update && \
    apt-get install -y \
        zlib1g-dev

RUN docker-php-ext-install gd