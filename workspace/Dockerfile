ARG PHP_VERSION=7.2-fpm
FROM php:${PHP_VERSION}

LABEL maintainer="Abdelrahman Omran <me@omranic.com>"

# Set Environment Variables
ENV DEBIAN_FRONTEND noninteractive

# Set Timezone
ARG TZ=UTC
ENV TZ ${TZ}

RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

COPY ./php.ini /usr/local/etc/php/php.ini
COPY ./xdebug.ini /usr/local/etc/php/xdebug.ini
COPY ./www.conf /usr/local/etc/php-fpm.d/www.conf

RUN apt-get update \
    && apt-get install -y --no-install-recommends git zip \
    apt-utils nano libz-dev libpq-dev libpng-dev \
    libjpeg-dev libmcrypt-dev libfreetype6-dev \
    libzip-dev libicu-dev jpegoptim optipng pngquant gifsicle \
    python python-pip python-dev build-essential \
    libmagickwand-dev imagemagick libxml2-dev libxslt1-dev \
    libc-client-dev libkrb5-dev \
    && python -m pip install --upgrade pip  \
    && python -m pip install --upgrade virtualenv

RUN pecl channel-update pecl.php.net \
    && pecl install -o -f redis xdebug imagick \
    && docker-php-ext-configure zip --with-libzip \
    && docker-php-ext-configure intl \
    && docker-php-ext-configure gd \
    --with-jpeg-dir=/usr/lib \
    --with-freetype-dir=/usr/include/freetype2 \
    && docker-php-ext-configure imap --with-kerberos --with-imap-ssl \
    && docker-php-ext-install zip gd pdo_mysql calendar imap \
    exif opcache bcmath mysqli intl sockets gettext soap xsl \
    && docker-php-ext-enable redis imagick xdebug

# Install Composer
ENV COMPOSER_DIR /root/.composer
COPY ./composer.json $COMPOSER_DIR/composer.json
RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer \
    && printf "\nPATH=\"$COMPOSER_DIR/vendor/bin:\$PATH\"\n" | tee -a ~/.bashrc \
    && composer global install

# Node / NVM:
ARG NODE_VERSION=stable
ENV NODE_VERSION ${NODE_VERSION}
ENV NVM_DIR /root/.nvm

RUN mkdir -p $NVM_DIR \
    && curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.33.11/install.sh | bash \
    && . $NVM_DIR/nvm.sh \
    && nvm install ${NODE_VERSION} \
    && nvm use ${NODE_VERSION} \
    && nvm alias ${NODE_VERSION}

# Wouldn't execute when added to the RUN statement in the above block
# Source NVM when loading bash since ~/.profile isn't loaded on non-login shell
RUN echo "" >> ~/.bashrc \
    && echo 'export NVM_DIR="$NVM_DIR/.nvm"' >> ~/.bashrc \
    && echo '[ -s "$NVM_DIR/nvm.sh" ] && . "$NVM_DIR/nvm.sh"  # This loads nvm' >> ~/.bashrc \
    && printf "\nPATH=\"$NVM_DIR/vendor/bin:\$PATH\"\n" | tee -a ~/.bashrc \
    && exec bash && . ~/.bashrc && npm install -g svgo

# Clean up
RUN apt-get clean && rm -rf /tmp/pear && \
    rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/* && \
    rm /var/log/lastlog /var/log/faillog

WORKDIR /var/www
