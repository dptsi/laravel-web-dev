FROM php:7.1-fpm-alpine3.10

# Install packages and remove default server definition
RUN apk --no-cache add gnupg autoconf make g++ nginx supervisor zlib-dev libpng-dev icu-dev icu-libs librdkafka-dev git libzip-dev shadow nodejs nodejs-npm && \
    rm /etc/nginx/conf.d/default.conf

# Install PHP extensions
RUN docker-php-ext-install bcmath calendar ctype curl dom exif fileinfo ftp gd gettext iconv intl json mbstring mcrypt pcntl phar posix readline shmop simplexml sockets sysvmsg sysvsem sysvshm tokenizer wddx xml xmlreader xmlwriter xsl zip

# Get latest Composer
COPY --from=composer:2.2.18 /usr/bin/composer /usr/bin/composer

# Download Microsoft SQL Server Prerequisites
RUN curl -O https://download.microsoft.com/download/e/4/e/e4e67866-dffd-428c-aac7-8d28ddafb39b/msodbcsql17_17.7.2.1-1_amd64.apk
RUN curl -O https://download.microsoft.com/download/e/4/e/e4e67866-dffd-428c-aac7-8d28ddafb39b/mssql-tools_17.7.1.1-1_amd64.apk

# Verify ODBC Signatures
RUN curl -O https://download.microsoft.com/download/e/4/e/e4e67866-dffd-428c-aac7-8d28ddafb39b/msodbcsql17_17.7.2.1-1_amd64.sig
RUN curl -O https://download.microsoft.com/download/e/4/e/e4e67866-dffd-428c-aac7-8d28ddafb39b/mssql-tools_17.7.1.1-1_amd64.sig
RUN curl https://packages.microsoft.com/keys/microsoft.asc  | gpg --import -
RUN gpg --verify msodbcsql17_17.7.2.1-1_amd64.sig msodbcsql17_17.7.2.1-1_amd64.apk
RUN gpg --verify mssql-tools_17.7.1.1-1_amd64.sig mssql-tools_17.7.1.1-1_amd64.apk

# Install the ODBC packages
RUN apk add --allow-untrusted msodbcsql17_17.7.2.1-1_amd64.apk
RUN apk add --allow-untrusted mssql-tools_17.7.1.1-1_amd64.apk

# Set mssql-tools ENV variable to the PATH
ENV PATH "$PATH:/opt/mssql-tools/bin"
RUN echo $PATH

# Remove the ODBC packages
RUN rm msodbcsql17_17.7.2.1-1_amd64.apk \
    mssql-tools_17.7.1.1-1_amd64.apk \
    msodbcsql17_17.7.2.1-1_amd64.sig \
    mssql-tools_17.7.1.1-1_amd64.sig

# Install unixodbc-dev required for pecl
RUN apk add --allow-untrusted unixodbc-dev

# Install SQL Server Drivers
RUN pecl channel-update pecl.php.net
RUN pecl install sqlsrv-5.6.1
RUN pecl install pdo_sqlsrv-5.6.1
RUN pecl install rdkafka    
RUN docker-php-ext-enable --ini-name 30-sqlsrv.ini sqlsrv
RUN docker-php-ext-enable --ini-name 35-pdo_sqlsrv.ini pdo_sqlsrv
RUN docker-php-ext-enable rdkafka

# Install Xdebug
RUN pecl install xdebug-2.9.0
RUN docker-php-ext-enable --ini-name 30-xdebug.ini xdebug

# Install redis
RUN echo '' | pecl install -o -f redis &&  rm -rf /tmp/pear &&  docker-php-ext-enable redis

# Configure nginx
COPY config/nginx.conf /etc/nginx/nginx.conf

# Configure PHP
RUN mv "$PHP_INI_DIR/php.ini-development" "$PHP_INI_DIR/php.ini"
COPY config/php.ini /usr/local/etc/php/conf.d/custom.ini

# Configure supervisord
COPY config/supervisord.conf /etc/supervisor/conf.d/supervisord.conf

# Setup document root
RUN mkdir -p /var/www/html

# Add application
WORKDIR /var/www/html

# Add a volume so that the external source code can be hooked
VOLUME [ "/var/www/html" ]

# Expose the port nginx is reachable on
EXPOSE 8080

# Let supervisord start nginx & php-fpm
CMD ["/usr/bin/supervisord", "-c", "/etc/supervisor/conf.d/supervisord.conf"]
