# PHP images for PHP sites (Drupal, Laravel, Symfony, WordPress and others),
# built on the official Debian (trixie) images.
#
#   fpm  production PHP-FPM on port 9000, runs as user "app" (UID 7000)
#   cli  the same PHP plus Composer, git, unzip and patch, runs as root.
#        For Composer build stages and CI jobs (phpcs, phpunit).
#
# Both targets share the "base" stage, so they always have the same PHP
# version, extensions and settings.

ARG PHP_VERSION=8.2

FROM php:${PHP_VERSION}-fpm-trixie AS base

COPY --from=mlocati/php-extension-installer:2.9.0 /usr/bin/install-php-extensions /usr/local/bin/

# Common extensions for PHP frameworks and CMSs: apcu (local cache), bcmath
# (money and big numbers), gd and imagick (images and PDF rendering), intl,
# pdo_mysql, redis (cache and queues), zip.
RUN install-php-extensions apcu bcmath bz2 exif gd imagick intl opcache pdo_mysql redis zip

# mariadb-client is used by CLI tools such as drush, wp-cli and
# "php artisan db" for database dumps and queries.
RUN apt-get update && \
    apt-get install -y --no-install-recommends mariadb-client && \
    rm -rf /var/lib/apt/lists/*

COPY --chmod=755 bin/mysql-skip-ssl /usr/local/bin/mysql
COPY --chmod=755 bin/mysql-skip-ssl /usr/local/bin/mysqldump
COPY --chmod=755 bin/mysql-skip-ssl /usr/local/bin/mariadb
COPY --chmod=755 bin/mysql-skip-ssl /usr/local/bin/mariadb-dump

RUN cp "$PHP_INI_DIR/php.ini-production" "$PHP_INI_DIR/php.ini"
COPY conf/php/zz-base.ini $PHP_INI_DIR/conf.d/zz-base.ini


FROM base AS cli

COPY --from=composer:2.9.5 /usr/bin/composer /usr/local/bin/composer

RUN apt-get update && \
    apt-get install -y --no-install-recommends git patch unzip && \
    rm -rf /var/lib/apt/lists/*

ENV COMPOSER_ALLOW_SUPERUSER=1 \
    COMPOSER_MEMORY_LIMIT=-1

WORKDIR /app
CMD ["php", "-a"]


# Last stage, so a plain "docker build ." gives the production image.
FROM base AS fpm

ARG SETUID=7000

COPY conf/php-fpm/zz-pool.conf /usr/local/etc/php-fpm.d/zz-pool.conf

# Defaults for conf/php-fpm/zz-pool.conf. Override them per site with
# ENV in the site's Dockerfile or in docker-compose.
ENV PHP_MEMORY_LIMIT=256M \
    PHP_FPM_MAX_CHILDREN=20 \
    PHP_FPM_START_SERVERS=6 \
    PHP_FPM_MIN_SPARE_SERVERS=4 \
    PHP_FPM_MAX_SPARE_SERVERS=12 \
    PHP_OPCACHE_VALIDATE_TIMESTAMPS=0

ARG APP_USER=app
RUN groupadd -g ${SETUID} ${APP_USER} && \
    useradd -u ${SETUID} -g ${SETUID} -m -s /usr/sbin/nologin ${APP_USER}

WORKDIR /var/www/html
# PHP-FPM and "docker exec" (CLI tools, cron jobs) both run as this user,
# so files written by cron or the CLI stay writable by the site.
USER ${APP_USER}
