FROM php:8.5.12-fpm-trixie

ARG DEBIAN_FRONTEND=noninteractive
ARG MSSQL_REPO_MAJOR=13

# Install required packages (Debian equivalents of requested package list), PHP extensions, and MS ODBC.
RUN apt-get update
RUN apt-get install -y --no-install-recommends gnupg
RUN apt-get install -y --no-install-recommends autoconf
RUN apt-get install -y --no-install-recommends make
RUN apt-get install -y --no-install-recommends g++
RUN apt-get install -y --no-install-recommends nginx
RUN apt-get install -y --no-install-recommends supervisor
RUN apt-get install -y --no-install-recommends zlib1g-dev
RUN apt-get install -y --no-install-recommends libpng-dev
RUN apt-get install -y --no-install-recommends libicu-dev
RUN apt-get install -y --no-install-recommends icu-devtools
RUN apt-get install -y --no-install-recommends librdkafka-dev
RUN apt-get install -y --no-install-recommends git
RUN apt-get install -y --no-install-recommends libzip-dev
RUN apt-get install -y --no-install-recommends passwd
RUN apt-get install -y --no-install-recommends nodejs
RUN apt-get install -y --no-install-recommends npm
RUN apt-get install -y --no-install-recommends file
RUN apt-get install -y --no-install-recommends imagemagick
RUN apt-get install -y --no-install-recommends libmagickwand-dev
RUN apt-get install -y --no-install-recommends unixodbc-dev
RUN apt-get install -y --no-install-recommends linux-libc-dev
RUN apt-get install -y --no-install-recommends curl

RUN docker-php-ext-install -j$(nproc) bcmath gd exif pcntl intl zip
RUN curl -fsSL https://packages.microsoft.com/keys/microsoft-2025.asc | gpg --dearmor -o /usr/share/keyrings/microsoft-prod.gpg
RUN echo "deb [arch=amd64 signed-by=/usr/share/keyrings/microsoft-prod.gpg] https://packages.microsoft.com/debian/${MSSQL_REPO_MAJOR}/prod $(. /etc/os-release && echo $VERSION_CODENAME) main" > /etc/apt/sources.list.d/mssql-release.list
RUN apt-get update
RUN ACCEPT_EULA=Y apt-get install -y --no-install-recommends msodbcsql18
RUN ACCEPT_EULA=Y apt-get install -y --no-install-recommends mssql-tools
RUN rm -rf /var/lib/apt/lists/*

# Install PECL extensions.
RUN pecl channel-update pecl.php.net     && pecl install sqlsrv-5.13.0 pdo_sqlsrv-5.13.0 rdkafka     && printf "\n" | pecl install redis     && docker-php-ext-enable sqlsrv pdo_sqlsrv rdkafka redis     && rm -rf /tmp/pear

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
