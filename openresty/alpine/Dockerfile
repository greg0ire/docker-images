FROM alpine:3.3

MAINTAINER Arnaud Chevillard <achevillard@universcine.com>

ENV OPENRESTY_VERSION=1.9.7.4 \
    OPENRESTY_PREFIX=/usr/local/openresty \
    LUAROCKS_VERSION=2.3.0 \
    LUA_SUFFIX=jit-2.1.0-beta1 \
    LUA_JIT_VERSION=2.1

RUN set -xe \
        && apk add --no-cache --virtual .build-deps \
            gcc \
            libc-dev \
            make \
            openssl-dev \
            pcre-dev \
            zlib-dev \
            linux-headers \
            curl \
            gnupg \
            libxslt-dev \
            gd-dev \
            geoip-dev \
            perl-dev \
            musl-dev \
            ncurses-dev \
            readline-dev \

        && cd /tmp \

        # OPENRESTY
        && curl -SL "https://openresty.org/download/openresty-${OPENRESTY_VERSION}.tar.gz" -o openresty.tar.gz \
        && tar xfz openresty.tar.gz \
        && rm openresty.tar.gz \
        && cd openresty-${OPENRESTY_VERSION} \
        && readonly NPROC=$(grep -c ^processor /proc/cpuinfo 2>/dev/null || 1) \
        && ./configure \
            --prefix=${OPENRESTY_PREFIX} \
            --sbin-path=/usr/sbin/nginx \
            --conf-path=/etc/nginx/nginx.conf \
            --error-log-path=/var/log/nginx/error.log \
            --http-log-path=/var/log/nginx/access.log \
            --pid-path=/var/run/nginx.pid \
            --lock-path=/var/run/nginx.lock \
            --http-client-body-temp-path=/var/cache/nginx/client_temp \
            --http-proxy-temp-path=/var/cache/nginx/proxy_temp \
            --http-fastcgi-temp-path=/var/cache/nginx/fastcgi_temp \
            --http-uwsgi-temp-path=/var/cache/nginx/uwsgi_temp \
            --http-scgi-temp-path=/var/cache/nginx/scgi_temp \
            --with-http_ssl_module \
            --with-http_realip_module \
            --with-http_addition_module \
            --with-http_sub_module \
            --with-http_dav_module \
            --with-http_flv_module \
            --with-http_mp4_module \
            --with-http_gunzip_module \
            --with-http_gzip_static_module \
            --with-http_random_index_module \
            --with-http_secure_link_module \
            --with-http_stub_status_module \
            --with-http_auth_request_module \
            --with-http_xslt_module\
            --with-http_image_filter_module\
            --with-http_geoip_module\
            --with-threads \
            --with-stream \
            --with-stream_ssl_module \
            --with-mail \
            --with-mail_ssl_module \
            --with-file-aio \
            --with-http_v2_module \
            --with-ipv6 \
            --with-luajit \
            --with-pcre-jit \
            --without-http_ssi_module \
            --without-http_userid_module \
            --without-http_uwsgi_module \
            --without-http_scgi_module \
            -j${NPROC} \
        && make -j${NPROC} \
        && make install \
        && mkdir -p /etc/nginx/conf.d/ \
        && mkdir -p /var/cache/nginx/ \

        # LUAROCKS
        && curl -SL "http://luarocks.org/releases/luarocks-${LUAROCKS_VERSION}.tar.gz" -o luarocks.tar.gz \
        && tar xfz luarocks.tar.gz \
        && rm luarocks.tar.gz \
        && cd luarocks-${LUAROCKS_VERSION} \
        && ./configure  \
            --prefix=${OPENRESTY_PREFIX}/luajit \
            --with-lua=${OPENRESTY_PREFIX}/luajit \
            --lua-suffix=${LUA_SUFFIX} \
            --with-lua-include=${OPENRESTY_PREFIX}/luajit/include/luajit-${LUA_JIT_VERSION} \
        && make \
        && make install \
        && ln -sf $OPENRESTY_PREFIX/luajit/bin/luajit-* $OPENRESTY_PREFIX/luajit/bin/lua \
        && ln -sf $OPENRESTY_PREFIX/luajit/bin/luajit-* /usr/sbin/lua \ 

        && strip /usr/sbin/nginx* \
        #&& strip /usr/lib/nginx/modules/*.so \
        && runDeps="$( \
            scanelf --needed --nobanner /usr/sbin/nginx \
                | awk '{ gsub(/,/, "\nso:", $2); print "so:" $2 }' \
                | sort -u \
                | xargs -r apk info --installed \
                | sort -u \
        )" \
    
        && apk add --virtual .nginx-rundeps $runDeps \
        && apk del .build-deps \
        && apk add --no-cache gettext \
            libpcrecpp \
            libpcre16 \
            libpcre32 \
            openssl \
            libssl1.0 \
            pcre \
            libgcc \
            libstdc++ \

        && rm -rf /var/cache/apk/* /tmp/* \
   
        # forward request and error logs to docker log collector
        && ln -sf /dev/stdout /var/log/nginx/access.log \
        && ln -sf /dev/stderr /var/log/nginx/error.log

EXPOSE 80 443

CMD ["nginx", "-g", "daemon off;"]
