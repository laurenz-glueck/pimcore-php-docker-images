FROM php:7.3.14-fpm

LABEL maintainer="laurenz.glueck@me.com"

# Replace shell with bash so we can source files
RUN rm /bin/sh && ln -s /bin/bash /bin/sh

# Set debconf to run non-interactively
RUN echo 'debconf debconf/frontend select Noninteractive' | debconf-set-selections

## install pimcore dependencies
RUN apt-get update && apt-get install -y lsb-release \
    && echo "deb http://deb.debian.org/debian $(lsb_release -sc)-backports main" > /etc/apt/sources.list.d/backports.list \
    && apt-get update && apt-get install -y --no-install-recommends \
        autoconf automake libtool nasm make gcc pkg-config libc-dev libz-dev build-essential openssl g++ \
        zlib1g-dev libicu-dev libbz2-dev libpng-dev libc-client-dev \
        libkrb5-dev libxml2-dev libxslt1.1 libxslt1-dev locales locales-all \
        ffmpeg html2text ghostscript libreoffice pngcrush jpegoptim exiftool poppler-utils git wget \
        libx11-dev libjpeg-dev python3-pip opencv-data webp graphviz cmake unzip cron \
        liblcms2-dev liblqr-1-0-dev libpng-dev libopenjp2-7-dev libtiff-dev libfreetype6-dev \
        libfontconfig1-dev libfftw3-dev libltdl-dev liblzma-dev libopenexr-dev \
        libwmf-dev libdjvulibre-dev libpango1.0-dev libxext-dev libxt-dev librsvg2-dev libzip-dev libmcrypt-dev nano nodejs openssh-client mariadb-client

RUN curl -sfLO https://www.imagemagick.org/download/ImageMagick.tar.gz \
        && tar -xvf ImageMagick.tar.gz \
        && cd ImageMagick-7.* \
        && ./configure \
        && make \
        && make install \
        && ldconfig /usr/local/lib \
        && pecl install imagick \
        && pecl install apcu \
        && printf "\n" | pecl install mcrypt-1.0.2 \
        && cd .. \
        && rm -rf ImageMagick*

RUN docker-php-ext-install intl mbstring mysqli bcmath bz2 gd soap xmlrpc xsl pdo pdo_mysql fileinfo exif zip opcache \
    && pecl install redis \
    && docker-php-ext-enable redis imagick apcu \
    && docker-php-ext-configure imap --with-kerberos --with-imap-ssl \
    && docker-php-ext-install imap \
    && docker-php-ext-enable imap

RUN cd ~ \
    && curl -sfLO https://github.com/wkhtmltopdf/wkhtmltopdf/releases/download/0.12.4/wkhtmltox-0.12.4_linux-generic-amd64.tar.xz \
        && tar xvf wkhtmltox-0.12.4_linux-generic-amd64.tar.xz \
        && mv wkhtmltox/bin/wkhtmlto* /usr/bin/ \
        && rm -rf wkhtmltox

RUN cd ~ \
    && git clone https://github.com/mozilla/mozjpeg.git  \
        && cd mozjpeg \
        && cmake -G"Unix Makefiles" \
        && make \
        && make install \
        && ln -s /opt/mozjpeg/bin/cjpeg /usr/bin/cjpeg \
        && cd .. \
        && rm -rf mozjpeg

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
    && curl -sfLO http://static.jonof.id.au/dl/kenutils/pngout-20150319-linux.tar.gz \
        && tar -xf pngout-20150319-linux.tar.gz \
        && rm pngout-20150319-linux.tar.gz \
        && cp pngout-20150319-linux/x86_64/pngout /bin/pngout \
        && cd .. \
        && rm -rf pngout-20150319-linux

RUN cd ~ \
    && curl -sfLO http://prdownloads.sourceforge.net/advancemame/advancecomp-1.17.tar.gz \
        && tar zxvf advancecomp-1.17.tar.gz \
        && cd advancecomp-1.17 \
        && ./configure \
        && make \
        && make install \
        && cd .. \
        && rm -rf advancecomp-1.17

RUN apt-get install -y curl software-properties-common \
    \
    && apt-get install -y php-redis \
    \
    && pip3 install datadog \
    \
    && apt-get autoremove -y \
        && apt-get remove -y autoconf libtool nasm pkg-config libz-dev build-essential g++ \
        && apt-get clean; rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/* /usr/share/doc/* ~/.composer

ENV NVM_DIR /usr/local/nvm
ENV NODE_VERSION 10.15.3

# install nvm
# https://github.com/creationix/nvm#install-script
RUN mkdir -p $NVM_DIR \
    && curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.0/install.sh | bash

# Install node and npm
RUN source $NVM_DIR/nvm.sh \
    && nvm install $NODE_VERSION \
    && nvm alias default $NODE_VERSION \
    && nvm use default

ENV NODE_PATH $NVM_DIR/v$NODE_VERSION/lib/node_modules
ENV PATH      $NVM_DIR/v$NODE_VERSION/bin:$PATH

## install yarn
RUN curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | apt-key add \
    && echo "deb https://dl.yarnpkg.com/debian/ stable main" | tee /etc/apt/sources.list.d/yarn.list \
    && apt update -qq && apt install yarn -y

## setup composer env vars
ENV COMPOSER_ALLOW_SUPERUSER 1
ENV COMPOSER_MEMORY_LIMIT -1

# Install Composer
RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/bin --filename=composer

## add composer to path
ENV PATH="${PATH}:/root/.composer/vendor/bin"

RUN echo "umask 000" >> /root/.bashrc && usermod -a -G root www-data && chsh www-data -s /bin/bash

## prepear ssh agent
RUN mkdir -p ~/.ssh

RUN eval $(ssh-agent -s)

RUN echo "Host *\n\tStrictHostKeyChecking no\n\tForwardAgent yes\n\n" > ~/.ssh/config

COPY ./www.conf /usr/local/etc/php-fpm.d/www.conf

CMD ["php-fpm", "-R"]
