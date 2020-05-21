# use this to create and run a drupal project

FROM composer:1.10

RUN apk update && \
    apk add libpng libpng-dev libjpeg-turbo-dev libwebp-dev zlib-dev libxpm-dev gd && docker-php-ext-install gd
