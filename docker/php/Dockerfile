FROM php:8.2-fpm

SHELL ["/bin/bash", "-l", "-c"]

USER root

ARG USER_ID
ARG GROUP_ID

# Update and install necessary packages
RUN apt-get update && apt-get install -y \
    nano git curl zip unzip zlib1g-dev libicu-dev g++ libxml2-dev supervisor \
    default-mysql-client cron libmcrypt-dev openssh-client libzip-dev \
    libmagickwand-dev imagemagick python3 \
    && rm -rf /var/lib/apt/lists/*

# Manage user and group
RUN userdel -f www-data \
    && if getent group www-data; then groupdel www-data; fi \
    && groupadd -g ${GROUP_ID} www-data \
    && useradd -l -u ${USER_ID} -g www-data www-data \
    && install -d -m 0755 -o www-data -g www-data /home/www-data \
    && chown -R www-data:www-data /var/www /home/www-data /var/log/supervisor /var/run \
    && touch /var/run/supervisord.pid /var/run/crond.pid \
    && chown www-data:www-data /var/run/supervisord.pid /var/run/crond.pid

# Update PECL channels and install Composer
RUN pecl update-channels \
    && curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer

# Install PHP extensions
RUN docker-php-ext-install intl zip exif pcntl bcmath soap pdo_mysql gd sockets gettext \
    && docker-php-ext-configure gd --with-freetype --with-jpeg \
    && pecl install imagick redis xdebug \
    && docker-php-ext-enable imagick redis xdebug

# Install Node.js
RUN curl -sL https://deb.nodesource.com/setup_lts.x | bash - \
    && apt-get install -y nodejs \
    && npm update -g

# Add crontab
ADD ./docker/php/crontab /etc/cron.d/ecommerce-cron

# Ensure the crontab file ends with a newline
RUN echo "" >>/etc/cron.d/ecommerce-cron

# Set correct permissions and ownership
RUN chmod 0644 /etc/cron.d/ecommerce-cron \
    && chown www-data:www-data /etc/cron.d/ecommerce-cron \
    && crontab -u www-data /etc/cron.d/ecommerce-cron \
    && usermod -a -G crontab www-data \
    && chmod gu+s /usr/sbin/cron

# Python socket setup
RUN python3 -c "import socket as s; sock = s.socket(s.AF_UNIX); sock.bind('/dev/shm/supervisor.sock')"

# Create the Supervisor configuration directory if it doesn't exist
RUN mkdir -p /etc/supervisor/conf.d

# Copy the existing Supervisor configuration file
COPY ./docker/php/supervisord.conf /etc/supervisor/conf.d/supervisord.conf

# Ensure proper permissions for Supervisor log directory
RUN mkdir -p /var/www/storage/logs \
    && chown -R www-data:www-data /var/www/storage/logs

# Change to www-data user
USER www-data

# Set work directory
WORKDIR /var/www

# Start supervisord
CMD ["/usr/bin/supervisord", "-c", "/etc/supervisor/conf.d/supervisord.conf"]
