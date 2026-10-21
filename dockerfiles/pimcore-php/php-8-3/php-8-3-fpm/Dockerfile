FROM php:8.3.36-fpm-bookworm

LABEL maintainer=laurenz.glueck@me.com
LABEL org.opencontainers.image.source=https://github.com/laurenz-glueck/pimcore-php-docker-images

# Replace shell with bash so we can source files
RUN rm /bin/sh && ln -s /bin/bash /bin/sh

# Set debconf to run non-interactively
RUN echo 'debconf debconf/frontend select Noninteractive' | debconf-set-selections

## install pimcore dependencies
RUN apt-get update && apt-get install -y lsb-release \
    && echo "deb http://deb.debian.org/debian $(lsb_release -sc)-backports main" > /etc/apt/sources.list.d/backports.list \
    && apt-get update && apt-get install -y --no-install-recommends \
        autoconf automake libtool nasm make gcc pkg-config libc-dev libz-dev build-essential openssl g++ \
        zlib1g-dev libicu-dev libjpeg62-turbo-dev libbz2-dev libc-client-dev \
        libkrb5-dev libxml2-dev libxslt1.1 libxslt1-dev locales locales-all \
        ffmpeg html2text ghostscript libreoffice pngcrush jpegoptim exiftool poppler-utils git wget \
        libx11-dev libjpeg-dev python3-pip opencv-data webp graphviz cmake unzip cron gzip \
        liblcms2-dev liblqr-1-0-dev libpng-dev libopenjp2-7-dev libtiff-dev libfreetype6-dev \
        libfontconfig1-dev libfftw3-dev libltdl-dev liblzma-dev libopenexr-dev \
        libwmf-dev libdjvulibre-dev libpango1.0-dev libxext-dev libxt-dev librsvg2-bin libzip-dev libmcrypt-dev \
        nano nodejs openssh-client mariadb-client libonig-dev libsodium-dev libssh2-1-dev lsof zip procps rsync grsync software-properties-common

RUN docker-php-ext-configure gd --with-freetype --with-jpeg

RUN docker-php-ext-install \
      bcmath \
      bz2 \
      calendar \
      exif \
      gd \
      gettext \
      intl \
      mbstring \
      mysqli \
      opcache \
      pcntl \
      pdo_mysql \
      soap \
      sockets \
      sodium \
      sysvmsg \
      sysvsem \
      sysvshm \
      xsl \
      zip

RUN docker-php-ext-enable \
      bcmath \
      bz2 \
      calendar \
      exif \
      gd \
      gettext \
      intl \
      mbstring \
      mysqli \
      opcache \
      pcntl \
      pdo_mysql \
      soap \
      sockets \
      sodium \
      sysvmsg \
      sysvsem \
      sysvshm \
      xsl \
      zip

COPY ImageMagick-7.1.1-41.tar.xz /ImageMagick-7.1.1-41.tar.xz

RUN cd / \
        && tar -xvf ImageMagick-7.1.1-41.tar.xz \
        && cd ImageMagick-7.1.1-41 \
        && ./configure \
        && make \
        && make install \
        && ldconfig /usr/local/lib \
        && cd .. \
        && rm -rf ImageMagick-7.1.1-41 \
        && rm ImageMagick-7.1.1-41.tar.xz

## pecl.php.net often answers with HTTP 500, so download with retries and install from the local file
RUN for p in redis-6.1.0 apcu-5.1.24; do \
        curl -fsSL --retry 10 --retry-all-errors --retry-delay 3 -o /tmp/$p.tgz https://pecl.php.net/get/$p.tgz || exit 1; \
    done \
    && pecl install /tmp/redis-6.1.0.tgz \
    && pecl install /tmp/apcu-5.1.24.tgz \
    && rm /tmp/*.tgz \
    && docker-php-ext-enable redis apcu \
    && docker-php-ext-configure imap --with-kerberos --with-imap-ssl \
    && docker-php-ext-install imap \
    && docker-php-ext-enable imap

#RUN pecl install imagick-3.6.0 \
#    && docker-php-ext-enable imagick

ADD --chmod=0755 \
  https://github.com/mlocati/docker-php-extension-installer/releases/download/2.6.3/install-php-extensions \
  /usr/local/bin/
# TODO: Use latest released version, after https://github.com/Imagick/imagick/issues/640 is fixed
RUN install-php-extensions imagick/imagick@28f27044e435a2b203e32675e942eb8de620ee58

#RUN cd ~ \
#    && curl -sfLO https://github.com/wkhtmltopdf/wkhtmltopdf/releases/download/0.12.4/wkhtmltox-0.12.4_linux-generic-amd64.tar.xz \
#        && tar xvf wkhtmltox-0.12.4_linux-generic-amd64.tar.xz \
#        && mv wkhtmltox/bin/wkhtmlto* /usr/bin/ \
#        && rm -rf wkhtmltox

RUN cd ~ \
    && git clone https://github.com/mozilla/mozjpeg.git  \
        && cd mozjpeg \
        && cmake -G"Unix Makefiles" \
        && make \
        && make install \
        && ln -s /opt/mozjpeg/bin/cjpeg /usr/bin/cjpeg \
        && cd .. \
        && rm -rf mozjpeg

# edjufy don't needs facedetect installed
#RUN cd ~ \
#    && git clone https://github.com/wavexx/facedetect \
#        && pip3 install numpy opencv-python \
#        && cd facedetect \
#        && cp facedetect /usr/local/bin \
#        && cd .. \
#        && rm -rf facedetect

RUN cd ~ \
    && git clone https://github.com/google/zopfli.git \
        && cd zopfli \
        && make \
        && cp zopfli /usr/bin/zopflipng \
        && cd .. \
        && rm -rf zopfli

RUN cd ~ \
    && curl -sfLO https://www.jonof.id.au/files/kenutils/pngout-20150319-linux.tar.gz \
        && echo "078a9ec068cc4d79ceca332b9e2117151c4584ee0e917e253d4fe80703499b47 *pngout-20150319-linux.tar.gz" | sha256sum -c - \
        && tar -xf pngout-20150319-linux.tar.gz \
        && rm pngout-20150319-linux.tar.gz \
        && cp pngout-20150319-linux/x86_64/pngout /bin/pngout \
        && cd .. \
        && rm -rf pngout-20150319-linux

#RUN cd ~ \
#    && curl -sfLO http://prdownloads.sourceforge.net/advancemame/advancecomp-1.17.tar.gz \
#        && tar zxvf advancecomp-1.17.tar.gz \
#        && cd advancecomp-1.17 \
#        && ./configure \
#        && make \
#        && make install \
#        && cd .. \
#        && rm -rf advancecomp-1.17

RUN cd ~ \
    && apt-get autoremove -y \
        && apt-get remove -y autoconf libtool nasm pkg-config libz-dev build-essential g++ \
        && apt-get clean; rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/* /usr/share/doc/* ~/.composer

ENV NVM_DIR=/usr/local/nvm
ENV NODE_VERSION=17.2.0

# install nvm
# https://github.com/creationix/nvm#install-script
RUN mkdir -p $NVM_DIR \
    && curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.1/install.sh | bash

# Install node and npm
RUN source $NVM_DIR/nvm.sh \
    && nvm install $NODE_VERSION \
    && nvm alias default $NODE_VERSION \
    && nvm use default \
    && npm install --global yarn

ENV NODE_PATH=$NVM_DIR/v$NODE_VERSION/lib/node_modules
ENV PATH=$NVM_DIR/v$NODE_VERSION/bin:$PATH

## setup composer env vars
ENV COMPOSER_ALLOW_SUPERUSER=1
ENV COMPOSER_MEMORY_LIMIT=-1

# Install Composer
RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/bin --filename=composer

## add composer to path
ENV PATH="${PATH}:/root/.composer/vendor/bin"

RUN echo "umask 000" >> /root/.bashrc && usermod -a -G root www-data && chsh www-data -s /bin/bash

## prepear ssh agent
RUN mkdir -p ~/.ssh

RUN eval $(ssh-agent -s)

RUN echo "Host *\n\tStrictHostKeyChecking no\n\tForwardAgent yes\n\n" > ~/.ssh/config

## set git config
RUN git config --global user.email 'dev@edjufy.com' && git config --global user.name 'edjufy Software GmbH'

COPY www.conf /usr/local/etc/php-fpm.d/www.conf

CMD ["php-fpm", "-R"]
