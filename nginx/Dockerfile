ARG NGINX_VERSION=1.27.5

FROM nginx:${NGINX_VERSION}-alpine AS builder
ARG NGINX_VERSION

RUN apk add --no-cache git build-base pcre-dev pcre2-dev zlib-dev openssl-dev linux-headers wget

WORKDIR /build
RUN wget -q https://nginx.org/download/nginx-${NGINX_VERSION}.tar.gz \
 && tar -xzf nginx-${NGINX_VERSION}.tar.gz \
 && git clone --depth 1 https://github.com/nginx-modules/ngx_cache_purge.git

RUN cd nginx-${NGINX_VERSION} \
 && ./configure --with-compat --add-dynamic-module=../ngx_cache_purge \
 && make modules

FROM nginx:${NGINX_VERSION}-alpine
ARG NGINX_VERSION

COPY --from=builder /build/nginx-${NGINX_VERSION}/objs/ngx_http_cache_purge_module.so /etc/nginx/modules/

RUN sed -i '1i load_module modules/ngx_http_cache_purge_module.so;' /etc/nginx/nginx.conf \
 && mkdir -p /var/cache/nginx/wordpress \
 && chown -R nginx:nginx /var/cache/nginx/wordpress

COPY fastcgi-cache.conf /etc/nginx/conf.d/00-fastcgi-cache.conf
COPY default.conf /etc/nginx/conf.d/default.conf
