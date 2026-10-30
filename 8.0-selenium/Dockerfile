FROM php:8.0-fpm-buster
# ^ this is deprecated!
# ^ when that is deprecated, apt sources.list will fail due to repository domain reasons.
# we are adding a patch so that apt could still succeed.

ENV ACCEPT_EULA=Y

# Buster is EOL: its repos moved to archive.debian.org, so replace sources.list to avoid 404s.
# Install prerequisites required for tools and extensions installed later on.
RUN printf "deb http://archive.debian.org/debian buster main\n" > /etc/apt/sources.list && \
    printf "deb http://archive.debian.org/debian-security buster/updates main\n" >> /etc/apt/sources.list && \
    rm -f /etc/apt/sources.list.d/*.list && \
    echo 'Acquire::Check-Valid-Until "false";' > /etc/apt/apt.conf.d/99archive && \
    apt-get update && \
    apt-get install -y gnupg nginx wget curl unzip supervisor librdkafka-dev gnupg2 libpng-dev libzip-dev apt-transport-https --no-install-recommends && \
    wget -q -O - https://dl-ssl.google.com/linux/linux_signing_key.pub | apt-key add - && \
    echo "deb http://dl.google.com/linux/chrome/deb/ stable main" >> /etc/apt/sources.list.d/google.list && \
    apt-get update -y && \
    apt-get install -y google-chrome-stable && \
    CHROMEVER=$(google-chrome --product-version | grep -o "[^\.]*\.[^\.]*\.[^\.]*") && \
    DRIVERVER=$(curl -s "https://chromedriver.storage.googleapis.com/LATEST_RELEASE_$CHROMEVER") && \
    wget -q --continue -P /chromedriver "http://chromedriver.storage.googleapis.com/$DRIVERVER/chromedriver_linux64.zip" && \
    unzip /chromedriver/chromedriver* -d /chromedriver

# Install prerequisites for the sqlsrv and pdo_sqlsrv PHP extensions.
RUN curl https://packages.microsoft.com/keys/microsoft.asc | apt-key add - \
    && curl https://packages.microsoft.com/config/debian/10/prod.list > /etc/apt/sources.list.d/mssql-release.list \
    && apt-get update \
    && apt-get install -y msodbcsql17 mssql-tools unixodbc-dev \
    && rm -rf /var/lib/apt/lists/*

# Retrieve the script used to install PHP extensions from the source container.
COPY --from=mlocati/php-extension-installer /usr/bin/install-php-extensions /usr/bin/install-php-extensions

# Install required PHP extensions and all their prerequisites available via apt.
RUN chmod uga+x /usr/bin/install-php-extensions \
    && sync \
    && install-php-extensions bcmath ds exif gd intl opcache pcntl pdo_sqlsrv redis sqlsrv zip rdkafka imagick

# Get latest Composer
COPY --from=composer:latest /usr/bin/composer /usr/bin/composer
# Configure nginx
COPY config/nginx.conf /etc/nginx/nginx.conf
# Configure supervisord
COPY config/supervisord.conf /etc/supervisor/conf.d/supervisord.conf
# Configure PHP
RUN mv "$PHP_INI_DIR/php.ini-development" "$PHP_INI_DIR/php.ini"
COPY config/php.ini /usr/local/etc/php/conf.d/custom.ini

# Setting the work directory.
WORKDIR /var/www/html

# Add a volume so that the external source code can be hooked
VOLUME [ "/var/www/html" ]

# Expose the port nginx is reachable on
EXPOSE 8080

# Let supervisord start nginx & php-fpm
CMD ["/usr/bin/supervisord", "-c", "/etc/supervisor/conf.d/supervisord.conf"]