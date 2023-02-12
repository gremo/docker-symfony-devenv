ARG TIMEZONE=UTC
ARG PHP_VERSION=cli-bullseye
ARG PHP_EXTENSIONS="apcu bcmath exif gd intl opcache pdo_mysql xdebug xsl zip"
ARG COMPOSER_VERSION=latest
ARG NODE_VERSION=current

# PHP builder stage
FROM php:${PHP_VERSION} as builder

RUN apt-get update && export DEBIAN_FRONTEND=noninteractive \
    && if grep -q 'VERSION="8 (jessie)"' /etc/os-release; then echo 'APT::Get::AllowUnauthenticated "true";' >/etc/apt/apt.conf.d/99-fix-jessie-keyring; fi \
    && curl -1sLf 'https://dl.cloudsmith.io/public/symfony/stable/setup.deb.sh' | bash \
    && apt-get install -y symfony-cli

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

RUN apt-get update && export DEBIAN_FRONTEND=noninteractive \
    && if grep -q 'VERSION="8 (jessie)"' /etc/os-release; then echo 'APT::Get::AllowUnauthenticated "true";' > /etc/apt/apt.conf.d/99-fix-jessie-keyring; fi \
    # Install OS tools
    && apt-get -y install --no-install-recommends git openssh-client unzip \
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
    && curl -fsSL https://deb.nodesource.com/setup_${NODE_VERSION}.x | bash && apt-get -y install nodejs \
    && npm update -g npm && npm install -g yarn \
    && if grep -q 'VERSION="8 (jessie)"' /etc/os-release; then npm install -g bower; fi \
    && rm -rf ~/.npm/ \
    # Cleanup
    && rm -rf /etc/apt/apt.conf.d/99-fix-jessie-keyring \
    && apt-get clean -y && rm -rf /var/lib/apt/lists/*
