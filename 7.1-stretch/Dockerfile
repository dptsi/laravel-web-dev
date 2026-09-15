FROM php:7.1-fpm-stretch

ARG DEBIAN_FRONTEND=noninteractive
ARG MSSQL_REPO_MAJOR=9

# Install required packages (Debian equivalents of requested package list), PHP extensions, and MS ODBC.
RUN printf "deb http://archive.debian.org/debian stretch main\n" > /etc/apt/sources.list
RUN printf "deb http://archive.debian.org/debian-security stretch/updates main\n" >> /etc/apt/sources.list
RUN rm -f /etc/apt/sources.list.d/*.list
RUN apt-get -o Acquire::Check-Valid-Until=false -o Acquire::AllowInsecureRepositories=true update
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
RUN apt-get install -y --no-install-recommends npm || true
RUN if ! command -v npm >/dev/null 2>&1; then \
		curl -fsSL https://nodejs.org/dist/v14.21.3/node-v14.21.3-linux-x64.tar.gz -o /tmp/node-v14.21.3-linux-x64.tar.gz; \
		tar -xzf /tmp/node-v14.21.3-linux-x64.tar.gz -C /usr/local --strip-components=1; \
		rm -f /tmp/node-v14.21.3-linux-x64.tar.gz; \
		ln -sf /usr/local/bin/node /usr/bin/node; \
		ln -sf /usr/local/bin/npm /usr/bin/npm; \
	fi
RUN node --version && npm --version
RUN apt-get install -y --no-install-recommends file
RUN apt-get install -y --no-install-recommends imagemagick
RUN apt-get install -y --no-install-recommends libmagickwand-dev
RUN apt-get install -y --no-install-recommends unixodbc-dev
RUN apt-get install -y --no-install-recommends linux-libc-dev
RUN apt-get install -y --no-install-recommends curl
RUN docker-php-ext-install -j$(nproc) bcmath gd exif pcntl intl zip
RUN curl -fsSL https://packages.microsoft.com/keys/microsoft.asc | gpg --dearmor -o /usr/share/keyrings/microsoft-prod.gpg
RUN echo "deb [arch=amd64 signed-by=/usr/share/keyrings/microsoft-prod.gpg] https://packages.microsoft.com/debian/${MSSQL_REPO_MAJOR}/prod stretch main" > /etc/apt/sources.list.d/mssql-release.list
RUN apt-get install -y --no-install-recommends apt-transport-https
RUN apt-get install -y --no-install-recommends ca-certificates
RUN apt-get -o Acquire::Check-Valid-Until=false -o Acquire::AllowInsecureRepositories=true update
RUN ACCEPT_EULA=Y apt-get install -y --no-install-recommends msodbcsql17
RUN ACCEPT_EULA=Y apt-get install -y --no-install-recommends mssql-tools
RUN rm -rf /var/lib/apt/lists/*

# Install PECL extensions.
RUN pecl channel-update pecl.php.net
RUN pecl install sqlsrv-5.6.1
RUN pecl install pdo_sqlsrv-5.6.1
RUN pecl install rdkafka-3.0.5
RUN printf "\n" | pecl install redis-5.3.7
RUN docker-php-ext-enable sqlsrv
RUN docker-php-ext-enable pdo_sqlsrv
RUN docker-php-ext-enable rdkafka
RUN docker-php-ext-enable redis
RUN rm -rf /tmp/pear

# Install Composer.
COPY --from=composer:2.2.18 /usr/bin/composer /usr/bin/composer

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
