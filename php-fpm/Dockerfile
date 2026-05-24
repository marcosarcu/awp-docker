FROM php:8.4-fpm-alpine

ARG WP_CLI_VERSION=2.12.0
ARG WP_CLI_PHAR_SHA512=be928f6b8ca1e8dfb9d2f4b75a13aa4aee0896f8a9a0a1c45cd5d2c98605e6172e6d014dda2e27f88c98befc16c040cbb2bd1bfa121510ea5cdf5f6a30fe8832
ARG WP_CLI_PHAR_SHA256=ce34ddd838f7351d6759068d09793f26755463b4a4610a5a5c0a97b68220d85c
ARG PECL_REDIS_VERSION=6.3.0
ARG PECL_IMAGICK_VERSION=3.8.1

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
    pecl install "redis-${PECL_REDIS_VERSION}"; \
    docker-php-ext-enable redis; \
    pecl install "imagick-${PECL_IMAGICK_VERSION}"; \
    docker-php-ext-enable imagick; \
    apk del .build-deps; \
    rm -rf /tmp/pear /var/cache/apk/*

RUN set -eux; \
    curl -fsSL -o /tmp/wp.phar \
        "https://github.com/wp-cli/wp-cli/releases/download/v${WP_CLI_VERSION}/wp-cli-${WP_CLI_VERSION}.phar"; \
    echo "${WP_CLI_PHAR_SHA512}  /tmp/wp.phar" | sha512sum -c -; \
    echo "${WP_CLI_PHAR_SHA256}  /tmp/wp.phar" | sha256sum -c -; \
    mv /tmp/wp.phar /usr/local/bin/wp; \
    chmod +x /usr/local/bin/wp

COPY conf/zz-custom.ini /usr/local/etc/php/conf.d/zz-custom.ini
COPY conf/crontab /etc/crontabs/www-data
COPY conf/imagemagick-policy.xml /etc/ImageMagick-7/policy.xml

WORKDIR /var/www/html

USER www-data
