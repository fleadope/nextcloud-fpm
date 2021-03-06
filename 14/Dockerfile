# Nextcloud
FROM craftdock/alpine-php7:7.2

LABEL maintainer="Hexosse <hexosse@gmail.com>" \
      description="Nextcloud image based on alpine with runit process supervisor."

# Nextcloud version
ENV NEXTCLOUD_VERSION 14.0.3

# Trust this project public key to trust the php packages.
ADD https://repos.php.earth/alpine/phpearth.rsa.pub /etc/apk/keys/phpearth.rsa.pub


# Ensure nextcloud user exists
RUN set -x \
    && addgroup -S nextcloud \
    && adduser -S -D -G nextcloud nextcloud \
    && adduser nextcloud www-data

RUN \
	# Print executed commands
	set -x \
    # Update repository indexes
    && apk-update \
    # Add recommanded packages : https://docs.nextcloud.com/server/12/admin_manual/installation/source_installation.html#prerequisites-label
    && apk-install --repository https://repos.php.earth/alpine/v3.8 \
        # Required : ctype, dom, GD, iconv, JSON, libxml, mbstring, posix, SimpleXML, XMLReader, XMLWriter ,zip, zlib
        php7.2-ctype \
        php7.2-dom \
        php7.2-gd \
        php7.2-iconv \
        php7.2-json \
        libxml2 \
        php7.2-mbstring \
        php7.2-posix \
        php7.2-simplexml \
        php7.2-xmlreader \
        php7.2-xmlwriter \
        php7.2-zip \
        php7.2-zlib \
        # Database connectors : pdo_sqlite, pdo_mysql, pdo_pgsql
        php7.2-pdo_sqlite \
        php7.2-pdo_mysql \
        php7.2-pgsql \
        # Recommended packages : curl, fileinfo, bz2, intl, mcrypt, openssl
        php7.2-curl \
        php7.2-fileinfo \
        php7.2-bz2 \
        php7.2-intl \
        php7.2-mcrypt \
        php7.2-openssl \
        # Required for specific apps : ldap, **smbclient**, ftp, imap
        php7.2-ldap \
        php7.2-ftp \
        php7.2-imap \
        # Recommended for specific apps : exif, gmp
        php7.2-exif \
        php7.2-gmp \
        # For enhanced server performance : apcu, memcached, redis
        php7.2-apcu \
        php7.2-memcached \
        php7.2-redis \
        # For preview generation : imagick, avconv or ffmpeg, OpenOffice or LibreOffice
        imagemagick-libs \
        php7.2-imagick \
        ffmpeg libva \
        libreoffice icu-libs \
        # For command line processing : pcntl
        php7.2-pcntl \
        # Other tools
        icu-libs \
        boost-filesystem \
    # Add packages used in this image
    && apk-install --repository http://dl-cdn.alpinelinux.org/alpine/edge/community/ \
        rsync \
    # Download, build and instal libsmbclient-php
    # Add necessary packages to install libsmbclient-php
    && apk-install --repository http://dl-cdn.alpinelinux.org/alpine/edge/community/ \
        samba \
    # Add temporary packages to compile libsmbclient-php
    && apk-install --repository https://repos.php.earth/alpine/v3.8 --repository http://dl-cdn.alpinelinux.org/alpine/edge/community/ --virtual .build-dependencies \
        git \
        autoconf \
        automake \
        file \
        g++ \
        gcc \
        make \
        php7.2-dev \
        re2c \
        samba-dev \
        zlib-dev \
    # Compile libsmbclient-php
    && git clone https://github.com/eduardok/libsmbclient-php.git /tmp/smbclient \
    && cd /tmp/smbclient \
    && phpize \
    && ./configure --with-php-config=/usr/bin/php-config \
    && make \
    && make install \
    # Activate libsmbclient-php
    && echo "extension=\"smbclient.so\"" > /etc/php7/conf.d/00_smbclient.ini \
	# Clear apk's cache
	&& rm -rf /etc/service.d/php-fpm \
	&& apk-remove .build-dependencies \
	&& apk-cleanup


# Download, install and verify Nextcloud
RUN curl -fsSL -o nextcloud.tar.bz2 \
        "https://download.nextcloud.com/server/releases/nextcloud-${NEXTCLOUD_VERSION}.tar.bz2" \
    && mkdir /usr/src/ \
	&& tar -xjf nextcloud.tar.bz2 -C /usr/src/ \
	&& rm nextcloud.tar.bz2 \
    #
    && rm -rf /usr/src/nextcloud/updater \
    && mkdir -p /usr/src/nextcloud/assets \
    && mkdir -p /usr/src/nextcloud/config \
    && mkdir -p /usr/src/nextcloud/custom_apps \
    && mkdir -p /usr/src/nextcloud/themes \
    && mkdir -p /usr/src/nextcloud/updater \
    && chmod +x /usr/src/nextcloud/occ


# Copy scripts
COPY /rootfs /

# Make bin scripts executables
RUN find /usr/local/bin/ -type f -print0 | xargs -0 chmod +x

# Add volume to allow persistence
VOLUME ["/nextcloud", "/var/www"]
