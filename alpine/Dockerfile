ARG TIMEZONE=UTC
ARG PHP_VERSION=cli-alpine
ARG PHP_EXTENSIONS="apcu bcmath exif gd intl opcache pdo_mysql xdebug xsl zip"
ARG COMPOSER_VERSION=latest
ARG NODE_VERSION=18.14.0

# PHP builder stage
FROM php:${PHP_VERSION} as builder

RUN apk update \
    && apk add bash \
    && curl -1sLf 'https://dl.cloudsmith.io/public/symfony/stable/setup.alpine.sh' | bash \
    && apk add symfony-cli

# Composer stage
FROM composer:${COMPOSER_VERSION} as composer

# Final PHP stage
FROM php:${PHP_VERSION}

ARG TIMEZONE
ARG PHP_EXTENSIONS
ARG NODE_VERSION

ENV TZ=$TIMEZONE

COPY --from=mlocati/php-extension-installer /usr/bin/install-php-extensions /usr/local/bin
COPY --from=builder /usr/bin/symfony /usr/local/bin
COPY --from=composer /usr/bin/composer /usr/local/bin

RUN apk update && apk upgrade --no-cache \
    # Install OS tools
    && apk add --no-cache git openssh-client unzip tzdata \
    # Install PHP extensions
    && if [ ! -z "${PHP_EXTENSIONS}" ]; then install-php-extensions ${PHP_EXTENSIONS}; fi \
    # Configure PHP
    && ln -s ${PHP_INI_DIR}/php.ini-development ${PHP_INI_DIR}/php.ini \
    && echo "apc.enable_cli = 1" >> ${PHP_INI_DIR}/conf.d/symfony.ini \
    && echo "date.timezone = $TIMEZONE" >> ${PHP_INI_DIR}/conf.d/symfony.ini \
    && echo "memory_limit = 512M" >> ${PHP_INI_DIR}/conf.d/symfony.ini \
    && echo "opcache.interned_strings_buffer = 16" >> ${PHP_INI_DIR}/conf.d/symfony.ini \
    && echo "opcache.max_accelerated_files = 20000" >> ${PHP_INI_DIR}/conf.d/symfony.ini \
    && echo "opcache.memory_consumption = 256" >> ${PHP_INI_DIR}/conf.d/symfony.ini \
    && echo "realpath_cache_size = 4096K" >> ${PHP_INI_DIR}/conf.d/symfony.ini \
    && echo "realpath_cache_ttl = 600" >> ${PHP_INI_DIR}/conf.d/symfony.ini \
    && echo "xdebug.client_host = 'host.docker.internal'" >> ${PHP_INI_DIR}/conf.d/symfony.ini \
    && sed -i '/exec/i \sed -i '\''s@date.timezone.*@'\''"date.timezone = $TZ"'\''@'\'' $PHP_INI_DIR/conf.d/symfony.ini\n' /usr/local/bin/docker-php-entrypoint \
    # Install Node and Yarn
    && ARCH= && alpineArch="$(apk --print-arch)" && if [ "${alpineArch##*-}" = "x86_64" ]; then ARCH="x64"; fi \
    && curl -fsSLO --compressed "https://unofficial-builds.nodejs.org/download/release/v$NODE_VERSION/node-v$NODE_VERSION-linux-$ARCH-musl.tar.xz" \
    && tar -xJf "node-v$NODE_VERSION-linux-$ARCH-musl.tar.xz" -C /usr/local --strip-components=1 --no-same-owner \
    && ln -s /usr/local/bin/node /usr/local/bin/nodejs \
    && npm update -g npm && npm install -g yarn \
    && rm -rf ~/.npm/ \
    # Cleanup
    && rm -rf /var/cache/apk/*
