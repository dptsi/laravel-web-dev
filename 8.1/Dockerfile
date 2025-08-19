FROM php:8.1.11-fpm-alpine3.16

# Install dependencies
RUN apk --no-cache add --virtual .build-deps \
      autoconf make g++ zlib-dev libpng-dev icu-dev libzip-dev shadow \
      imagemagick-dev unixodbc-dev git curl gnupg \
    && apk --no-cache add \
      nginx supervisor icu-libs librdkafka-dev \
      nodejs npm file imagemagick \
    && docker-php-ext-install -j$(nproc) bcmath gd exif pcntl intl zip \
    && pecl channel-update pecl.php.net \
    && pecl install sqlsrv-5.12.0 pdo_sqlsrv-5.12.0 rdkafka \
    && echo '' | pecl install -o -f redis \
    && docker-php-ext-enable \
         redis sqlsrv pdo_sqlsrv rdkafka \
    && rm -rf /tmp/pear

# Composer
COPY --from=composer:latest /usr/bin/composer /usr/bin/composer

# Install Microsoft ODBC + Tools
RUN curl -sSL -O https://download.microsoft.com/download/e/4/e/e4e67866-dffd-428c-aac7-8d28ddafb39b/msodbcsql17_17.10.6.1-1_amd64.apk \
    -O https://download.microsoft.com/download/e/4/e/e4e67866-dffd-428c-aac7-8d28ddafb39b/mssql-tools_17.10.1.1-1_amd64.apk \
    -O https://download.microsoft.com/download/e/4/e/e4e67866-dffd-428c-aac7-8d28ddafb39b/msodbcsql17_17.10.6.1-1_amd64.sig \
    -O https://download.microsoft.com/download/e/4/e/e4e67866-dffd-428c-aac7-8d28ddafb39b/mssql-tools_17.10.1.1-1_amd64.sig \
    && curl -sSL https://packages.microsoft.com/keys/microsoft.asc | gpg --import - \
    && gpg --verify msodbcsql17_17.10.6.1-1_amd64.sig msodbcsql17_17.10.6.1-1_amd64.apk \
    && gpg --verify mssql-tools_17.10.1.1-1_amd64.sig mssql-tools_17.10.1.1-1_amd64.apk \
    && apk add --allow-untrusted \
         msodbcsql17_17.10.6.1-1_amd64.apk \
         mssql-tools_17.10.1.1-1_amd64.apk \
    && rm msodbcsql17_17.10.6.1-1_amd64.* mssql-tools_17.10.1.1-1_amd64.* \
    && apk del .build-deps

# Set PATH for mssql-tools
ENV PATH="$PATH:/opt/mssql-tools/bin"

# Configure Nginx, PHP, Supervisor
COPY config/nginx.conf /etc/nginx/nginx.conf
RUN mv "$PHP_INI_DIR/php.ini-development" "$PHP_INI_DIR/php.ini"
COPY config/php.ini /usr/local/etc/php/conf.d/custom.ini
COPY config/supervisord.conf /etc/supervisor/conf.d/supervisord.conf

# Setup app
WORKDIR /var/www/html
VOLUME [ "/var/www/html" ]

# Expose Nginx
EXPOSE 8080

# Start supervisord
CMD ["/usr/bin/supervisord", "-c", "/etc/supervisor/conf.d/supervisord.conf"]
