ARG NGINX_VERSION=1.22.1

FROM nginx:$NGINX_VERSION-alpine

ARG NGINX_VERSION=1.22.1
ARG BROTLI_VERSION=master

RUN apk --update --no-cache add \
        gcc \
        make \
        libc-dev \
        g++ \
        openssl-dev \
        linux-headers \
        pcre-dev \
        zlib-dev \
        libtool \
        automake \
        autoconf \
        git

RUN cd /opt \
    && git clone --depth 1 -b $BROTLI_VERSION --single-branch https://github.com/google/ngx_brotli.git \
    && cd /opt/ngx_brotli \
    && git submodule update --init \
    && cd /opt \
    && wget -O - http://nginx.org/download/nginx-$NGINX_VERSION.tar.gz | tar zxfv - \
    && mv /opt/nginx-$NGINX_VERSION /opt/nginx \
    && cd /opt/nginx \
    && ./configure --with-compat --add-dynamic-module=/opt/ngx_brotli \
    && make modules 

FROM nginx:$NGINX_VERSION-alpine

COPY --from=0 /opt/nginx/objs/ngx_http_brotli_filter_module.so /usr/lib/nginx/modules
COPY --from=0 /opt/nginx/objs/ngx_http_brotli_static_module.so /usr/lib/nginx/modules

RUN chmod -R 644 \
        /usr/lib/nginx/modules/ngx_http_brotli_filter_module.so \
        /usr/lib/nginx/modules/ngx_http_brotli_static_module.so \
    && sed -i '1iload_module \/usr\/lib\/nginx\/modules\/ngx_http_brotli_filter_module.so;' /etc/nginx/nginx.conf \
    && sed -i '1iload_module \/usr\/lib\/nginx\/modules\/ngx_http_brotli_static_module.so;' /etc/nginx/nginx.conf \
    && sed -i '/^http {/a \ \ \ \ brotli on;\n\ \ \ \ brotli_comp_level 6;\n\ \ \ \ brotli_static on;\n\ \ \ \ brotli_types application/atom+xml application/javascript application/json application/rss+xml application/vnd.ms-fontobject application/x-font-opentype application/x-font-truetype application/x-font-ttf application/x-javascript application/xhtml+xml application/xml font/eot font/opentype font/otf font/truetype image/svg+xml image/vnd.microsoft.icon image/x-icon image/x-win-bitmap text/css text/javascript text/plain text/xml;' /etc/nginx/nginx.conf