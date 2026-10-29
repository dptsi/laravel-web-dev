FROM php:8.0-fpm-alpine3.13

# Install packages and remove default server definition
RUN apk --no-cache add gnupg autoconf make g++ nginx supervisor zlib-dev libpng-dev icu-dev icu-libs librdkafka-dev git libzip-dev shadow nodejs nodejs-npm file imagemagick imagemagick-dev && \
    rm /etc/nginx/conf.d/default.conf

# Update Linux Header
RUN apk add --update linux-headers

# Install PHP extensions
RUN docker-php-ext-install bcmath gd exif pcntl intl zip

# Get latest Composer
COPY --from=composer:latest /usr/bin/composer /usr/bin/composer

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
# with grpc 1.85.0, we have this compile errors:
#   1600.6 /tmp/pear/temp/grpc/src/core/lib/promise/detail/promise_like.h: In member function 'auto grpc_core::http2::GoawayManager::RequestGoaway(grpc_core::http2::Http2ErrorCode, grpc_core::Slice&&, uint32_t, bool)':
#   1600.6 /tmp/pear/temp/grpc/src/core/lib/promise/detail/promise_like.h:219:65: internal compiler error: in assign_temp, at function.c:984
#   1600.6   219 |       : f_(std::forward<Factory>(f)(std::forward<Args>(args)...)) {}
#   1600.6       |                                                                 ^
#   1600.7 Please submit a full bug report,
#   1600.7 with preprocessed source if appropriate.
#   1600.7 See <https://gcc.gnu.org/bugs/> for instructions.
#   1600.8 make: *** [Makefile:511: src/core/ext/transport/chttp2/transport/http2_client_transport.lo] Error 1
#   1601.0 ERROR: `make' failed
# currently triaging with 1.84.0 -> fail
# from blame on 1.85.0, that specific line changed on 1.81.0. (https://github.com/grpc/grpc/blame/48e7698fbd77e2eae59d2c4820c1c063477e7993/src/core/lib/promise/detail/promise_like.h , https://github.com/grpc/grpc/commit/03a9d3a90067750d90c77783d6e5722af90df11a)
# triaging on 1.80.0. ->> fail
# triaging on 1.70.0 ->> success
# triaging on 1.75.1 ->> fail
RUN pecl channel-update pecl.php.net && pecl install sqlsrv-5.11.1 && pecl install pdo_sqlsrv-5.11.1 && pecl install rdkafka && pecl install imagick && pecl install grpc-1.70.0 && docker-php-ext-enable grpc

RUN docker-php-ext-enable --ini-name 30-sqlsrv.ini sqlsrv
RUN docker-php-ext-enable --ini-name 35-pdo_sqlsrv.ini pdo_sqlsrv
RUN docker-php-ext-enable rdkafka
RUN docker-php-ext-enable imagick

# Install Xdebug
RUN pecl install xdebug-3.2.0
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

# Configure PHP-FPM
# COPY config/www.conf /usr/local/etc/php-fpm.d/www.conf

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
