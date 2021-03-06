FROM php:7.0-fpm-alpine

MAINTAINER Arnaud Chevillard <achevillard@universcine.com>

RUN set -xe \
        && apk add --no-cache --virtual .phpext-builddeps \
            libbz2 \
            gettext-dev \
            libmcrypt-dev \
            libxslt-dev \
            zlib-dev \
            postgresql-dev \
            autoconf gcc libc-dev make \
            libpng-dev libjpeg-turbo-dev \
            icu-dev \
            libev \
            openldap-dev \

        && docker-php-ext-configure gd --with-png-dir=/usr --with-jpeg-dir=/usr \
        && docker-php-ext-configure ldap --with-libdir=lib/ \
        && docker-php-ext-install \
            bcmath \
            bz2 \
            calendar \
            exif \
            ftp \
            gd \
            gettext \
            intl \
            ldap \
            mbstring \
            mcrypt \
            opcache \
            pcntl \
            pdo_mysql \
            pdo_pgsql \
            pspell \
            shmop \
            sockets \
            soap \
            sysvmsg \
            sysvsem \
            sysvshm \
            xsl \
            zip \

        && find /usr/local/lib/php/extensions -name '*.a' -delete \
        && find /usr/local/lib/php/extensions -name '*.so' -exec strip --strip-all '{}' \; \
        && runDeps="$( \
            scanelf --needed --nobanner --recursive \
            /usr/local/lib/php/extensions \
            | awk '{ gsub(/,/, "\nso:", $2); print "so:" $2 }' \
            | sort -u \
            | xargs -r apk info --installed \
            | sort -u \
        )" \
        && apk add --virtual .phpext-rundeps $runDeps \
        && apk del .phpext-builddeps \

        && cp /usr/src/php/php.ini-production $PHP_INI_DIR/php.ini \

        # Opcache settings
        && { \
            echo 'opcache.memory_consumption=128'; \
            echo 'opcache.interned_strings_buffer=8'; \
            echo 'opcache.max_accelerated_files=4000'; \
            echo 'opcache.revalidate_freq=60'; \
            echo 'opcache.fast_shutdown=1'; \
            echo 'opcache.enable_cli=1'; \
        } > $PHP_INI_DIR/conf.d/opcache-recommended.ini \

        # Time Zone
        && echo "date.timezone=${PHP_TIMEZONE:-UTC}" > $PHP_INI_DIR/conf.d/date_timezone.ini \

        # install composer
        && wget -O /usr/local/bin/composer https://getcomposer.org/composer.phar \
        && chmod +x /usr/local/bin/composer \
        && composer self-update \

        # cleanup from apk
        && rm -rf /tmp/* /var/cache/apk/*


WORKDIR /srv 
