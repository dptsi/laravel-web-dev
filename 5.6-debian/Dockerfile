FROM php:5.6-fpm-stretch

# Fix Debian Stretch EOL repos
RUN echo "deb [trusted=yes] http://archive.debian.org/debian stretch main contrib non-free" > /etc/apt/sources.list && \
    echo "deb [trusted=yes] http://archive.debian.org/debian-security stretch/updates main contrib non-free" >> /etc/apt/sources.list

# Install system dependencies
RUN apt-get -o Acquire::Check-Valid-Until=false update && \
    DEBIAN_FRONTEND=noninteractive apt-get install -y --no-install-recommends \
    build-essential \
    gnupg2 \
    curl \
    git \
    nginx \
    supervisor \
    libpng-dev \
    libjpeg-dev \
    libxml2-dev \
    libxslt1-dev \
    libmcrypt-dev \
    libicu-dev \
    freetds-dev \
    freetds-bin \
    pkg-config \
    unixodbc-dev \
    locales \
    && apt-get clean && rm -rf /var/lib/apt/lists/*

# Build librdkafka from source because Debian Stretch package is too old for PECL rdkafka
RUN curl -fsSL https://github.com/edenhill/librdkafka/archive/refs/tags/v0.11.6.tar.gz -o /tmp/librdkafka.tar.gz && \
    mkdir -p /tmp/librdkafka-src && \
    tar -xzf /tmp/librdkafka.tar.gz -C /tmp/librdkafka-src --strip-components=1 && \
    cd /tmp/librdkafka-src && \
    ./configure --prefix=/usr && \
    make -j"$(nproc)" && \
    make install && \
    ldconfig && \
    rm -rf /tmp/librdkafka.tar.gz /tmp/librdkafka-src

# Link FreeTDS library before installing pdo_dblib
RUN ln -s /usr/lib/x86_64-linux-gnu/libsybdb.so /usr/lib/

# Install PHP extensions
RUN docker-php-ext-install \
    bcmath \
    gd \
    intl \
    mbstring \
    mcrypt \
    pdo_dblib \
    soap \
    xsl \
    zip

# Install PECL extensions (latest releases dropped PHP 5.6 support)
RUN pecl channel-update pecl.php.net || true && \
    echo '' | pecl install -o -f redis-4.3.0 rdkafka-4.1.2 && \
    docker-php-ext-enable redis rdkafka

# Get Composer 1.x (Composer 2 requires PHP 7.2+)
COPY --from=composer:1.10.26 /usr/bin/composer /usr/bin/composer

# SQL Server connectivity for PHP 5.6 via FreeTDS PDO driver

# Configure nginx
COPY config/nginx.conf /etc/nginx/nginx.conf

# Configure PHP
COPY config/php/php.conf /usr/local/etc/php/conf.d/custom.ini

# Configure supervisord
COPY config/supervisord.conf /etc/supervisor/conf.d/supervisord.conf

# Install locales
RUN echo "en_US.UTF-8 UTF-8" >> /etc/locale.gen && locale-gen

RUN mkdir -p /var/www/html && \
    chown -R www-data:www-data /var/www/html

WORKDIR /var/www/html/

VOLUME /var/www/html

EXPOSE 80

CMD ["/usr/bin/supervisord", "-c", "/etc/supervisor/conf.d/supervisord.conf"]
