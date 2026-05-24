ARG NGINX_VERSION=1.30.2
ARG NGINX_TARBALL_SHA256=7df3090907fca3cc0e456d6dc00ceb230da74ea88026ceff0affc29dbbd9ac4c
ARG NGX_CACHE_PURGE_COMMIT=285354eddd5675c765ba2b79dac09f5d3065b22f

FROM nginx:${NGINX_VERSION}-alpine AS builder
ARG NGINX_VERSION
ARG NGINX_TARBALL_SHA256
ARG NGX_CACHE_PURGE_COMMIT

RUN apk add --no-cache git build-base pcre-dev pcre2-dev zlib-dev openssl-dev linux-headers wget gnupg

WORKDIR /build

RUN set -eux; \
    wget -q "https://nginx.org/download/nginx-${NGINX_VERSION}.tar.gz"; \
    wget -q "https://nginx.org/download/nginx-${NGINX_VERSION}.tar.gz.asc"; \
    echo "${NGINX_TARBALL_SHA256}  nginx-${NGINX_VERSION}.tar.gz" | sha256sum -c -; \
    \
    export GNUPGHOME="$(mktemp -d)"; \
    for key in nginx_signing arut pluknet sb thresh; do \
        wget -q -O - "https://nginx.org/keys/${key}.key" | gpg --batch --import; \
    done; \
    gpg --batch --verify "nginx-${NGINX_VERSION}.tar.gz.asc" "nginx-${NGINX_VERSION}.tar.gz"; \
    gpgconf --kill all; \
    rm -rf "$GNUPGHOME" "nginx-${NGINX_VERSION}.tar.gz.asc"; \
    \
    tar -xzf "nginx-${NGINX_VERSION}.tar.gz"; \
    \
    git clone https://github.com/nginx-modules/ngx_cache_purge.git; \
    git -C ngx_cache_purge -c advice.detachedHead=false checkout "${NGX_CACHE_PURGE_COMMIT}"; \
    rm -rf ngx_cache_purge/.git

RUN cd nginx-${NGINX_VERSION} \
 && ./configure --with-compat --add-dynamic-module=../ngx_cache_purge \
 && make modules

FROM nginxinc/nginx-unprivileged:${NGINX_VERSION}-alpine
ARG NGINX_VERSION

USER root

COPY --from=builder /build/nginx-${NGINX_VERSION}/objs/ngx_http_cache_purge_module.so /etc/nginx/modules/

RUN sed -i '1i load_module modules/ngx_http_cache_purge_module.so;' /etc/nginx/nginx.conf \
 && mkdir -p /var/cache/nginx/wordpress \
 && chown -R nginx:nginx /var/cache/nginx/wordpress

COPY fastcgi-cache.conf /etc/nginx/conf.d/00-fastcgi-cache.conf
COPY default.conf /etc/nginx/conf.d/default.conf

USER nginx
