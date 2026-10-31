# php

PHP images for PHP sites (Drupal, Laravel, Symfony, WordPress and others),
built on the official `php:<version>-fpm-trixie` image (Debian 13).

| Image | Use |
|---|---|
| `tuanhmt/php:8.2-fpm` | Production PHP-FPM on port 9000. Runs as `app` (UID 7000). |
| `tuanhmt/php:8.2-cli` | Composer build stages and CI jobs. Same PHP, plus Composer, git, unzip and patch. Runs as root. |

Each build is also tagged with the exact PHP version, for example
`8.2.35-fpm`.

## What is inside

- Extensions: apcu, bcmath, bz2, exif, gd, imagick, intl, opcache, pdo_mysql,
  redis, zip, plus the ones built into PHP.
- `mariadb-client`, with `mysql` and `mysqldump` wrappers that add
  `--skip-ssl`. MariaDB 11.4+ clients require TLS by default, and tools such
  as Drush pass `--defaults-file`, so an option file cannot turn it off. See
  `bin/mysql-skip-ssl`.
- PHP settings in `conf/php/zz-base.ini`: 100M uploads, 120s execution time,
  2G memory for the CLI.
- PHP-FPM pool in `conf/php-fpm/zz-pool.conf`.

## Settings you can change per site

Set these as environment variables on the fpm container:

| Variable | Default | Meaning |
|---|---|---|
| `PHP_MEMORY_LIMIT` | `256M` | Memory limit per web request |
| `PHP_FPM_MAX_CHILDREN` | `20` | Maximum PHP-FPM worker processes |

## Use in a site's Dockerfile

```dockerfile
FROM tuanhmt/php:8.2-cli AS composer
WORKDIR /app
COPY . /app/
RUN composer install --no-dev --optimize-autoloader

FROM tuanhmt/php:8.2-fpm
COPY --from=composer --chown=7000:7000 /app /var/www/html
```

## Build locally

From this folder:

```bash
docker build --target fpm -t php:8.2-fpm .
docker build --target cli -t php:8.2-cli .
```

Use `--build-arg PHP_VERSION=8.3` for another PHP version. The fpm user is
`app` with UID 7000 by default. Change them with `--build-arg APP_USER=...`
and `--build-arg SETUID=...`. Keep the UID stable for a site, because volume
files are owned by UID.

To publish another PHP version, add it to the `php_version` list in
`.github/workflows/php.yml`.
