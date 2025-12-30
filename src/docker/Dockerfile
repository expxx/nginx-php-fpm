ARG phpVersion=8.3-dev
FROM php:${phpVersion}-fpm
ARG WORKDIR=/var/www/html
ENV DOCUMENT_ROOT=${WORKDIR}
ENV LARAVEL_PROCS_NUMBER=1
ENV DOMAIN=_
ENV CLIENT_MAX_BODY_SIZE=15M
ENV NODE_VERSION=20.x
ARG GROUP_ID=1000
ARG USER_ID=1000
ENV USER_NAME=www-data
ARG GROUP_NAME=www-data
# Install system dependencies
RUN apt-get update && apt-get install -y \
    git \
    curl \
    libfreetype6-dev \
    libjpeg62-turbo-dev \
    libmemcached-dev \
    libzip-dev \
    libpng-dev \
    libonig-dev \
    libxml2-dev \
    librdkafka-dev \
    libpq-dev \
    openssh-server \
    zip \
    unzip \
    supervisor \
    sqlite3  \
    nano \
    cron

RUN curl -fsSL https://deb.nodesource.com/setup_${NODE_VERSION} | bash -
 # Install Node    
RUN apt-get install -y nodejs     
# Install nginx 
RUN apt-get update && apt-get install -y nginx

# Clear cache
RUN apt-get clean && rm -rf /var/lib/apt/lists/*

RUN docker-php-ext-install mysqli \
        && docker-php-ext-install pdo_mysql \
        && docker-php-ext-install zip

# XenForo2 required extension GMP
RUN docker-php-ext-configure gmp && docker-php-ext-install gmp

# PHP 8 does not include pecl so switch to new module manager pickle
RUN curl -L --output /tmp/pickle.phar https://github.com/FriendsOfPHP/pickle/releases/download/v0.7.11/pickle.phar \
    && mv /tmp/pickle.phar /usr/local/bin/pickle \
    && chmod +x /usr/local/bin/pickle

# PHP 8 does not include pecl so switch to new module manager pickle
RUN curl -L --output /tmp/pickle.phar https://github.com/FriendsOfPHP/pickle/releases/download/v0.7.11/pickle.phar \
    && mv /tmp/pickle.phar /usr/local/bin/pickle \
    && chmod +x /usr/local/bin/pickle

RUN docker-php-ext-configure gd --with-freetype --with-jpeg \
    && docker-php-ext-install -j "$(nproc)" gd

RUN pickle install imagick && docker-php-ext-enable imagick
RUN pickle install redis && docker-php-ext-enable redis

# Set PHP configuration for upload limits
RUN echo "upload_max_filesize = 8M" >> /usr/local/etc/php/conf.d/uploads.ini \
    && echo "post_max_size = 8M" >> /usr/local/etc/php/conf.d/uploads.ini \
    && echo "max_execution_time = 300" >> /usr/local/etc/php/conf.d/uploads.ini \
    && echo "max_input_time = 300" >> /usr/local/etc/php/conf.d/uploads.ini

# Set working directory
WORKDIR $WORKDIR

RUN rm -Rf /var/www/* && \
mkdir -p /var/www/html

ADD src/index.php $WORKDIR/index.php
ADD src/php.ini $PHP_INI_DIR/conf.d/
ADD src/opcache.ini $PHP_INI_DIR/conf.d/
ADD src/supervisor/supervisord.conf /etc/supervisor/supervisord.conf

COPY src/entrypoint.sh /usr/local/bin/
RUN chmod +x /usr/local/bin/entrypoint.sh && \
 ln -s /usr/local/bin/entrypoint.sh /

RUN rm -rf /etc/nginx/conf.d/default.conf && \
rm -rf /etc/nginx/sites-enabled/default && \
 rm -rf /etc/nginx/sites-available/default && \
    rm -rf /etc/nginx/nginx.conf

COPY src/nginx.conf /etc/nginx/nginx.conf
COPY src/default.conf /etc/nginx/conf.d/

RUN usermod -u ${USER_ID} ${USER_NAME} && \
 groupmod -g ${USER_ID} ${GROUP_NAME}

RUN mkdir -p /var/log/supervisor /var/log/nginx /var/cache/nginx

RUN chown -R ${USER_NAME}:${GROUP_NAME} /var/www && \
  chown -R ${USER_NAME}:${GROUP_NAME} /var/log/ && \
  chown -R ${USER_NAME}:${GROUP_NAME} /etc/supervisor/conf.d/ && \
  chown -R ${USER_NAME}:${GROUP_NAME} $PHP_INI_DIR/conf.d/ && \
  touch /var/run/nginx.pid && \
  chown -R $USER_NAME:$USER_NAME /var/cache/nginx && \
  chown -R $USER_NAME:$USER_NAME /var/lib/nginx/ && \
  chown -R $USER_NAME:$USER_NAME /etc/nginx/nginx.conf && \
  chown -R $USER_NAME:$USER_NAME /var/run/nginx.pid && \
  chown -R $USER_NAME:$USER_NAME /var/log/supervisor && \
  chown -R $USER_NAME:$USER_NAME /etc/nginx/conf.d/ && \
  chown -R ${USER_NAME}:${GROUP_NAME} /tmp


#USER ${USER_NAME}
EXPOSE 80
ENTRYPOINT ["entrypoint.sh"]
