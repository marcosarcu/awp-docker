FROM php:8.4-fpm-alpine

RUN set -eux; \
    apk add --no-cache --virtual .build-deps \
        $PHPIZE_DEPS \
        autoconf \
        freetype-dev \
        libjpeg-turbo-dev \
        libpng-dev \
        libwebp-dev \
        libavif-dev \
        libzip-dev \
        icu-dev \
        oniguruma-dev \
        libxml2-dev \
        imagemagick-dev \
        pkgconf; \
    apk add --no-cache \
        freetype \
        libjpeg-turbo \
        libpng \
        libwebp \
        libavif \
        libzip \
        icu-libs \
        imagemagick \
        mariadb-client \
        bash \
        less \
        curl; \
    docker-php-ext-configure gd --with-freetype --with-jpeg --with-webp --with-avif; \
    docker-php-ext-install -j"$(nproc)" \
        gd \
        mysqli \
        pdo_mysql \
        exif \
        intl \
        opcache \
        zip \
        bcmath; \
    pecl install redis; \
    docker-php-ext-enable redis; \
    pecl install imagick-3.8.1; \
    docker-php-ext-enable imagick; \
    apk del .build-deps; \
    rm -rf /tmp/pear /var/cache/apk/*

RUN curl -sSL -o /usr/local/bin/wp \
        https://raw.githubusercontent.com/wp-cli/builds/gh-pages/phar/wp-cli.phar \
    && chmod +x /usr/local/bin/wp

COPY conf/zz-custom.ini /usr/local/etc/php/conf.d/zz-custom.ini
COPY conf/crontab /etc/crontabs/www-data

WORKDIR /var/www/html

USER www-data
