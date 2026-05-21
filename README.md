# awp-docker

Imágenes Docker reutilizables para los sitios WordPress de la "stack AWP" (Agency WordPress).

## Imágenes publicadas

Cada push a `main` que toque `nginx/` o `php-fpm/` dispara un build y publica a GHCR:

- `ghcr.io/marcosarcu/awp-nginx:latest` — nginx 1.27 alpine con módulo `ngx_cache_purge` compilado, configs FastCGI cache opt-in con bypass para WP / WooCommerce.
- `ghcr.io/marcosarcu/awp-php-fpm:latest` — PHP 8.4-fpm alpine con extensiones GD (freetype/jpeg/webp/avif), mysqli, pdo_mysql, exif, intl, opcache, zip, bcmath, redis (PECL) e imagick 3.8.1 (PECL). Incluye wp-cli y crontab para wp-cron + action-scheduler.

Tags adicionales: `:sha-<7chars>` por cada commit, útil para rollback.

## Cómo consumir desde un sitio

En `docker-compose.yml`:

```yaml
services:
  nginx:
    image: ghcr.io/marcosarcu/awp-nginx:latest
    pull_policy: always
    # ...volumes, healthcheck, depends_on

  php-fpm:
    image: ghcr.io/marcosarcu/awp-php-fpm:latest
    pull_policy: always
    # ...environment, volumes, depends_on

  wp-cron:
    image: ghcr.io/marcosarcu/awp-php-fpm:latest
    pull_policy: always
    command: ["crond", "-f", "-l", "8", "-L", "/dev/stdout", "-c", "/etc/crontabs"]
    # ...rest
```

## Forzar rebuild manual

GitHub Actions → workflow `build-and-push` → Run workflow.

## Estructura

```
nginx/        Dockerfile + default.conf + fastcgi-cache.conf (configs horneadas)
php-fpm/      Dockerfile + conf/zz-custom.ini + conf/crontab
.github/      workflows/build-and-push.yml
```

Sitios que consumen: `mastercoip`, `coipmethod`.
