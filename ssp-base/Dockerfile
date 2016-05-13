FROM php:5.6.14-apache
RUN apt-get update -y
# Install the various packages needed by ssp and composer
RUN apt-get install -y \
  file \
  libgmp-dev \
  libmcrypt-dev \
  libmhash-dev \
  re2c \
  zlib1g-dev

# Enable php extensions
RUN ln -s /usr/include/x86_64-linux-gnu/gmp.h /usr/local/include/ \
  && docker-php-ext-configure gmp \
  && docker-php-ext-install gmp \
  && docker-php-ext-configure mcrypt \
  && docker-php-ext-install mcrypt \
  && docker-php-ext-configure zip \
  && docker-php-ext-install zip


# Download and enable composer
ENV COMPOSER_VERSION=1.1.0
ENV COMPOSER_HASH d8e1d21081b7076dfc9a570620e082111ab196dabd83780e50025c52d62b801c
RUN curl -o /usr/local/bin/composer https://getcomposer.org/download/$COMPOSER_VERSION/composer.phar \
  &&  echo "$COMPOSER_HASH /usr/local/bin/composer" | sha256sum -c - \
  && chmod u+x  /usr/local/bin/composer

# TODO: Have composer do a pre-fetch of packagist so future build are faster?

ENV SSP_VERSION 1.14.3
ENV SSP_HASH 99e030d744ca4d8cd5033ef0da51d9f825ae05302045664fd49a2bae4e057a21
ENV SSP_DIR /var/simplesamlphp

RUN mkdir $SSP_DIR \
  && curl -o /tmp/ssp.tar.gz https://simplesamlphp.org/res/downloads/simplesamlphp-$SSP_VERSION.tar.gz \
  &&  echo "$SSP_HASH  /tmp/ssp.tar.gz" | sha256sum -c - \
  && tar xvzf /tmp/ssp.tar.gz --strip-components 1 -C $SSP_DIR

# make any additional dirs required by common modules
RUN mkdir $SSP_DIR/data \
  && chown -R www-data $SSP_DIR \
  && mkdir $SSP_DIR/ticketcache \
  && chown -R www-data $SSP_DIR/ticketcache

# Add default index page
RUN echo "You should access SSP on https.<p/>If you are using the nginx docker proxy make sure to set 'VIRTUAL_PORT=443' and 'VIRTUAL_PROTO=https'" > /var/www/html/index.html

# Enable Apache modules
RUN a2enmod ssl rewrite
EXPOSE 443
# Default certs
RUN apt-get install ssl-cert
# Default basepath in apache config
ENV SSP_APACHE_ALIAS simplesaml
ENV APACHE_CERT_NAME ssl-cert-snakeoil
# Sites must end with .conf
ADD ssp-apache.conf /etc/apache2/sites-available/ssp.conf
RUN a2ensite ssp


# Customize php.ini??

# TODO: Customize config.php to log to errlog

COPY ssp-startup.sh /opt/simplesaml/ssp-startup.sh
RUN chmod u+x /opt/simplesaml/ssp-startup.sh
CMD /opt/simplesaml/ssp-startup.sh