FROM php:8.6-rc-fpm-trixie

ENV ASUSER= \
    UID= \
    COMPOSER_ALLOW_SUPERUSER=1 \
    COMPOSER_MEMORY_LIMIT=-1 \
    PHP_DATE_TIMEZONE=UTC \
    PHP_MEMORY_LIMIT=256M \
    PHP_MAX_INPUT_VARS=1000 \
    PHP_UPLOAD_MAX_FILESIZE=25M \
    PHP_POST_MAX_SIZE=25M \
    PHP_MAX_EXECUTION_TIME=30 \
    PHP_FPM_LISTEN=9000 \
    PHP_FPM_MAX_CHILDREN=10 \
    PHP_FPM_REQUEST_TERMINATE_TIMEOUT=60 \
    ENTRYPOINT=entrypoint.php.sh

WORKDIR /app

RUN useradd -m -u 1337 kool \
    && usermod -aG www-data kool \
    #pie
    && curl -fL -o /usr/local/bin/pie \
            https://github.com/php/pie/releases/latest/download/pie.phar \
    && chmod +x /usr/local/bin/pie \
    # dockerize
    && DOCKERIZE_ARCH="$( [ "$(uname -m)" = "aarch64" ] && echo arm64 || echo amd64 )" \
    && curl -L "https://github.com/jwilder/dockerize/releases/download/v0.15.2/dockerize-linux-${DOCKERIZE_ARCH}-v0.15.2.tar.gz" | tar xz \
    && mv dockerize /usr/local/bin/dockerize \
    # gosu
    && GOSU_ARCH="$( [ "$(uname -m)" = "aarch64" ] && echo arm64 || echo amd64 )" \
    && curl -L "https://github.com/tianon/gosu/releases/download/1.17/gosu-${GOSU_ARCH}" -o /usr/local/bin/gosu \
    && chmod +x /usr/local/bin/gosu \
    # deps
    && apt-get update && apt-get install -y --no-install-recommends \
        bash sed git openssh-client procps \
        libfreetype6 libpng16-16t64 libjpeg62-turbo libzip5 ghostscript imagemagick \
        jpegoptim optipng pngquant gifsicle \
        libldap2 libpq5 libicu76 less \
    # build-deps
    && cp "/usr/local/etc/php/php.ini-production" "/usr/local/etc/php/php.ini" \
    && apt-get install -y --no-install-recommends \
        $PHPIZE_DEPS \
        libfreetype6-dev libpng-dev libjpeg62-turbo-dev \
        libicu-dev libedit-dev libxml2-dev \
        libmagickwand-dev libldap2-dev libonig-dev libwebp-dev \
        libpq-dev libzip-dev libtool \
    # php-ext
    && docker-php-ext-configure gd --with-freetype --with-webp --with-jpeg \
    && export CFLAGS="$PHP_CFLAGS" CPPFLAGS="$PHP_CPPFLAGS" LDFLAGS="$PHP_LDFLAGS" \
    && docker-php-ext-install -j$(nproc) \
        bcmath \
        calendar \
        exif \
        gd \
        intl \
        ldap \
        mbstring \
        pcntl \
        pdo \
        pdo_mysql \
        pdo_pgsql \
        soap \
        xml \
        zip \
        sockets \
        mysqli \
        ftp \
    && pie install phpredis/phpredis:dev-develop@dev \
    && pie install imagick/imagick:dev-master \
    # composer
    && curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer \
    && curl -sS https://getcomposer.org/installer | php -- --1 --install-dir=/usr/local/bin --filename=composer1 \
    # cleanup
    && apt-get purge -y --auto-remove $PHPIZE_DEPS \
        libfreetype6-dev libpng-dev libjpeg62-turbo-dev \
        libicu-dev libedit-dev libxml2-dev \
        libmagickwand-dev libldap2-dev libonig-dev libwebp-dev \
        libpq-dev libzip-dev \
    && rm -rf /var/lib/apt/lists/* /tmp/*

COPY kool.ini /kool/kool.tmpl
COPY zz-docker.conf /kool/zz-docker.tmpl
COPY entrypoint /kool/entrypoint
RUN chmod +x /kool/entrypoint

EXPOSE 9000

ENTRYPOINT [ "/kool/entrypoint" ]
CMD [ "php-fpm" ]
