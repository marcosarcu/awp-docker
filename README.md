# awp-docker

Imágenes Docker reutilizables para los sitios WordPress de AWP Host.

## Imágenes publicadas

Cada push a `main` que toque `nginx/` o `php-fpm/` dispara un build y publica a GHCR:

- `ghcr.io/marcosarcu/awp-nginx:latest` — nginx 1.30.2 (stable) sobre `nginxinc/nginx-unprivileged` alpine, módulo `ngx_cache_purge` compilado (commit pineado, tarball verificado con SHA256 + firmas PGP de los mantenedores nginx), configs FastCGI cache opt-in con bypass para WP / WooCommerce. **Escucha en `:8080`** (la imagen corre como usuario no-root).
- `ghcr.io/marcosarcu/awp-php-fpm:latest` — PHP 8.4-fpm alpine con extensiones GD (freetype/jpeg/webp/avif), mysqli, pdo_mysql, exif, intl, opcache, zip, bcmath, redis 6.3.0 (PECL) e imagick 3.8.1 (PECL). Incluye wp-cli 2.12.0 (descarga verificada por SHA-512 y SHA-256) y crontab para wp-cron + action-scheduler. Ships con `policy.xml` de ImageMagick endurecido (sin coders PS/EPS/PDF/SVG/MSL/MVG/etc.).

Tags adicionales: `:sha-<7chars>` por cada commit, útil para rollback.

## Cómo consumir desde un sitio

En `docker-compose.yml`:

```yaml
services:
  nginx:
    image: ghcr.io/marcosarcu/awp-nginx:latest
    pull_policy: always
    # El contenedor expone :8080 (corre como usuario no-root).
    # Apunta el proxy/Coolify a este puerto en lugar de :80.
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

