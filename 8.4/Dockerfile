FROM php:8.4.8-fpm-alpine3.20

# Install required packages
RUN apk --no-cache add \
    gnupg \
    autoconf \
    make \
    g++ \
    nginx \
    supervisor \
    zlib-dev \
    libpng-dev \
    icu-dev \
    icu-libs \
    librdkafka-dev \
    git \
    libzip-dev \
    shadow \
    nodejs \
    npm \
    file \
    imagemagick \
    imagemagick-dev \
    unixodbc-dev \
    linux-headers

# Install PHP extensions
RUN docker-php-ext-install \
    bcmath \
    gd \
    exif \
    pcntl \
    intl \
    zip

# Install Composer
COPY --from=composer:latest /usr/bin/composer /usr/bin/composer

# Download Microsoft ODBC and Tools packages
WORKDIR /tmp/msodbc

RUN curl -O https://download.microsoft.com/download/e/4/e/e4e67866-dffd-428c-aac7-8d28ddafb39b/msodbcsql17_17.10.6.1-1_amd64.apk && \
    curl -O https://download.microsoft.com/download/e/4/e/e4e67866-dffd-428c-aac7-8d28ddafb39b/mssql-tools_17.10.1.1-1_amd64.apk && \
    curl -O https://download.microsoft.com/download/e/4/e/e4e67866-dffd-428c-aac7-8d28ddafb39b/msodbcsql17_17.10.6.1-1_amd64.sig && \
    curl -O https://download.microsoft.com/download/e/4/e/e4e67866-dffd-428c-aac7-8d28ddafb39b/mssql-tools_17.10.1.1-1_amd64.sig && \
    curl https://packages.microsoft.com/keys/microsoft.asc | gpg --import - && \
    gpg --verify msodbcsql17_17.10.6.1-1_amd64.sig msodbcsql17_17.10.6.1-1_amd64.apk && \
    gpg --verify mssql-tools_17.10.1.1-1_amd64.sig mssql-tools_17.10.1.1-1_amd64.apk && \
    apk add --allow-untrusted msodbcsql17_17.10.6.1-1_amd64.apk mssql-tools_17.10.1.1-1_amd64.apk && \
    rm -f *.apk *.sig

# Set mssql-tools path
ENV PATH="/opt/mssql-tools/bin:$PATH"

# Install PHP PECL extensions
RUN pecl channel-update pecl.php.net && \
    pecl install sqlsrv-5.12.0 pdo_sqlsrv-5.12.0 rdkafka && \
    echo '' | pecl install -o -f redis && \
    docker-php-ext-enable sqlsrv pdo_sqlsrv rdkafka redis

# Optional Xdebug (uncomment if needed)
# RUN pecl install xdebug-3.2.0 && \
#     docker-php-ext-enable --ini-name 30-xdebug.ini xdebug

# Configure nginx
COPY config/nginx.conf /etc/nginx/nginx.conf

# Configure PHP
RUN mv "$PHP_INI_DIR/php.ini-development" "$PHP_INI_DIR/php.ini"
COPY config/php.ini /usr/local/etc/php/conf.d/custom.ini

# Configure supervisord
COPY config/supervisord.conf /etc/supervisor/conf.d/supervisord.conf

# Create document root
RUN mkdir -p /var/www/html
WORKDIR /var/www/html

# Volume for app code
VOLUME ["/var/www/html"]

# Expose port
EXPOSE 8080

# Start supervisor (nginx + php-fpm)
CMD ["/usr/bin/supervisord", "-c", "/etc/supervisor/conf.d/supervisord.conf"]
