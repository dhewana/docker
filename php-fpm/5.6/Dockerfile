FROM debian:jessie

RUN mv /etc/apt/sources.list /etc/apt/sources.list.bak \
        && echo "deb http://kambing.ui.ac.id/debian/ jessie main contrib non-free" > /etc/apt/sources.list \
        && echo "deb http://kambing.ui.ac.id/debian/ jessie-updates main contrib non-free" >> /etc/apt/sources.list \
        && echo "deb http://kambing.ui.ac.id/debian-security/ jessie/updates main contrib non-free" >> /etc/apt/sources.list

ENV PHP_INI_DIR /usr/local/etc/php
RUN mkdir -p $PHP_INI_DIR/conf.d

ENV PHP_EXTRA_CONFIGURE_ARGS \
        --with-config-file-path="$PHP_INI_DIR" \
        --with-config-file-scan-dir="$PHP_INI_DIR/conf.d" \
        --with-libdir=lib64 \
        --enable-fpm \
        --with-fpm-user=www-data \
        --with-fpm-group=www-data \
        --enable-cli \
        --with-zlib \
        --with-curl \
        --enable-mbstring \
        --with-gd \
        --with-jpeg-dir \
        --with-png-dir \
        --with-freetype-dir \
        --enable-gd-native-ttf \
        --with-pdo-mysql \
        --enable-ftp \
        --with-openssl \
        --with-mcrypt \
        --enable-zip

ENV GPG_KEYS 0BD78B5F97500D450838F95DFE857D9A90D90EC1 6E4F6AB321FDC07F2C332E3AC2BF0BC433CFC8B3

ENV PHP_VERSION 5.6.21
ENV PHP_FILENAME php-5.6.21.tar.xz
ENV PHP_SHA256 566ff1a486cb0485ed477a91ea292423f77a58671270ff73b74e67e3ce7084f9

COPY docker-php-ext-* /usr/local/bin/

COPY composer /usr/local/bin/

RUN set -xe \
        && buildDeps=" \
        $PHP_EXTRA_BUILD_DEPS \
        autoconf \
        build-essential \
        ca-certificates \
        curl \
        file \
        g++ \
        gcc \
        imagemagick \
        libbz2-dev \
        libc-dev \
        libcurl4-openssl-dev \
        libedit2 \
        libedit-dev \
        libgd2-noxpm-dev \
        libmagickwand-dev \
        libmcrypt-dev \
        libmcrypt4 \
        libmemcached-dev \
        libmysqld-dev \
        libpcre3-dev \
        libpng12-dev \
        libssl-dev \
        libxml2 \
        libxml2-dev \
        make \
        mcrypt \
        php-pear \
        php5-dev \
        pkg-config \
        re2c \
        wget \
        xz-utils \
        " \
        && apt-get update && apt-get install -y $buildDeps --no-install-recommends && rm -rf /var/lib/apt/lists/* \
        && curl -fSL "http://php.net/get/$PHP_FILENAME/from/this/mirror" -o "$PHP_FILENAME" \
        && echo "$PHP_SHA256 *$PHP_FILENAME" | sha256sum -c - \
        && curl -fSL "http://php.net/get/$PHP_FILENAME.asc/from/this/mirror" -o "$PHP_FILENAME.asc" \
        && export GNUPGHOME="$(mktemp -d)" \
        && for key in $GPG_KEYS; do \
                gpg --keyserver ha.pool.sks-keyservers.net --recv-keys "$key"; \
        done \
        && gpg --batch --verify "$PHP_FILENAME.asc" "$PHP_FILENAME" \
        && rm -r "$GNUPGHOME" "$PHP_FILENAME.asc" \
        && mkdir -p /usr/src/php \
        && tar -xf "$PHP_FILENAME" -C /usr/src/php --strip-components=1 \
        && rm "$PHP_FILENAME" \
        && cd /usr/src/php \
        && ./configure $PHP_EXTRA_CONFIGURE_ARGS \
        && make -j"$(nproc)" \
        && make install \
        && { find /usr/local/bin /usr/local/sbin -type f -executable -exec strip --strip-all '{}' + || true; } \
        && make clean \
        && pecl install imagick memcached redis mongodb \
        && docker-php-ext-enable imagick memcached redis \
        && apt-get purge -y --auto-remove -o APT::AutoRemove::RecommendsImportant=false -o APT::AutoRemove::SuggestsImportant=false autoconf build-essential ca-certificates curl file g++ gcc make mcrypt php-pear php5-dev pkg-config re2c wget xz-utils

WORKDIR /var/www/html

RUN set -ex \
        && cd /usr/local/etc \
        && if [ -d php-fpm.d ]; then \
                # for some reason, upstream's php-fpm.conf.default has "include=NONE/etc/php-fpm.d/*.conf"
                sed 's!=NONE/!=!g' php-fpm.conf.default | tee php-fpm.conf > /dev/null; \
                cp php-fpm.d/www.conf.default php-fpm.d/www.conf; \
        else \
                # PHP 5.x don't use "include=" by default, so we'll create our own simple config that mimics PHP 7+ for consistency
                mkdir php-fpm.d; \
                cp php-fpm.conf.default php-fpm.d/www.conf; \
                { \
                        echo '[global]'; \
                        echo 'include=etc/php-fpm.d/*.conf'; \
                } | tee php-fpm.conf; \
        fi \
        && { \
                echo '[global]'; \
                echo 'error_log = /proc/self/fd/2'; \
                echo; \
                echo '[www]'; \
                echo '; if we send this to /proc/self/fd/1, it never appears'; \
                echo 'access.log = /proc/self/fd/2'; \
                echo; \
                echo 'clear_env = no'; \
                echo; \
                echo '; Ensure worker stdout and stderr are sent to the main error log.'; \
                echo 'catch_workers_output = yes'; \
        } | tee php-fpm.d/docker.conf \
        && { \
                echo '[global]'; \
                echo 'daemonize = no'; \
                echo; \
                echo '[www]'; \
                echo 'listen = [::]:9000'; \
        } | tee php-fpm.d/zz-docker.conf

EXPOSE 9000

CMD ["php-fpm"]
