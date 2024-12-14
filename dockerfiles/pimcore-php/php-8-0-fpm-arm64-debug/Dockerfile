FROM ghcr.io/laurenz-glueck/pimcore-php-docker-images/pimcore-php/php-8-0-fpm-arm64:1.5

LABEL maintainer="laurenz.glueck@me.com"
LABEL org.opencontainers.image.source="https://github.com/laurenz-glueck/pimcore-php-docker-images"

##install <xdebug>##
RUN apt-get update && apt-get install -y --no-install-recommends \
      autoconf automake libtool nasm make pkg-config libz-dev build-essential g++ iproute2 \
    && pecl install xdebug \
    && docker-php-ext-enable xdebug \
    && apt-get autoremove -y \
    && apt-get remove -y autoconf automake libtool nasm make pkg-config libz-dev build-essential g++ \
    && apt-get clean; rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/* /usr/share/doc/* ~/.composer

RUN rm -f /usr/local/etc/php/conf.d/20-xdebug.ini && \
    touch /usr/local/etc/php/conf.d/20-xdebug.ini && \
    echo "xdebug.idekey = PHPSTORM" >> /usr/local/etc/php/conf.d/20-xdebug.ini && \
    echo "xdebug.mode = debug" >> /usr/local/etc/php/conf.d/20-xdebug.ini && \
    echo "xdebug.start_with_request = yes" >> /usr/local/etc/php/conf.d/20-xdebug.ini && \
    echo "xdebug.discover_client_host = true" >> /usr/local/etc/php/conf.d/20-xdebug.ini && \
    echo "xdebug.remote_handler = dbgp" >> /usr/local/etc/php/conf.d/20-xdebug.ini && \
    echo "xdebug.client_port = 9003" >> /usr/local/etc/php/conf.d/20-xdebug.ini && \
    echo "xdebug.client_host = host.docker.internal" >> /usr/local/etc/php/conf.d/20-xdebug.ini

ENV PHP_DEBUG 1
ENV PHP_IDE_CONFIG serverName=localhost

COPY entrypoint.sh /usr/local/bin
RUN chmod +x /usr/local/bin/entrypoint.sh
ENTRYPOINT ["/usr/local/bin/entrypoint.sh"]
##</xdebug>##

RUN echo "umask 000" >> /root/.bashrc && usermod -a -G root www-data && chsh www-data -s /bin/bash

COPY ./www.conf /usr/local/etc/php-fpm.d/www.conf

CMD ["php-fpm", "-R"]
