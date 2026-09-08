FROM php:8.2-fpm-bookworm

ARG DEBIAN_FRONTEND=noninteractive
ARG MSSQL_REPO_MAJOR=12

# Install required packages (Debian equivalents of requested package list), PHP extensions, and MS ODBC.
RUN apt-get update     && apt-get install -y --no-install-recommends         gnupg         autoconf         make         g++         nginx         supervisor         zlib1g-dev         libpng-dev         libicu-dev         icu-devtools         librdkafka-dev         git         libzip-dev         passwd         nodejs         npm         file         imagemagick         libmagickwand-dev         unixodbc-dev         linux-libc-dev         curl     && docker-php-ext-install -j$(nproc) bcmath gd exif pcntl intl zip     && curl -fsSL https://packages.microsoft.com/keys/microsoft.asc | gpg --dearmor -o /usr/share/keyrings/microsoft-prod.gpg     && echo "deb [arch=amd64 signed-by=/usr/share/keyrings/microsoft-prod.gpg] https://packages.microsoft.com/debian/${MSSQL_REPO_MAJOR}/prod $(. /etc/os-release && echo $VERSION_CODENAME) main" > /etc/apt/sources.list.d/mssql-release.list     && apt-get update     && ACCEPT_EULA=Y apt-get install -y --no-install-recommends msodbcsql17 mssql-tools     && rm -rf /var/lib/apt/lists/*

# Install PECL extensions.
RUN pecl channel-update pecl.php.net     && pecl install sqlsrv-5.12.0 pdo_sqlsrv-5.12.0 rdkafka     && printf "\n" | pecl install redis     && docker-php-ext-enable sqlsrv pdo_sqlsrv rdkafka redis     && rm -rf /tmp/pear

# Install Composer.
COPY --from=composer:2.10.3 /usr/bin/composer /usr/bin/composer

# Make mssql-tools available globally.
ENV PATH="/opt/mssql-tools/bin:$PATH"

# Configure services.
COPY config/nginx.conf /etc/nginx/nginx.conf
RUN mv "$PHP_INI_DIR/php.ini-development" "$PHP_INI_DIR/php.ini"
COPY config/php.ini /usr/local/etc/php/conf.d/custom.ini
COPY config/supervisord.conf /etc/supervisor/conf.d/supervisord.conf
RUN rm -f /etc/nginx/sites-enabled/default /etc/nginx/conf.d/default.conf || true

WORKDIR /var/www/html
VOLUME ["/var/www/html"]

EXPOSE 8080
CMD ["/usr/bin/supervisord", "-c", "/etc/supervisor/conf.d/supervisord.conf"]
