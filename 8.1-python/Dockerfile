# Base image
FROM php:8.1-apache-bullseye

# Bullseye is EOL: its repos moved to archive.debian.org, so replace sources.list to avoid 404s.
RUN printf "deb http://archive.debian.org/debian bullseye main\n" > /etc/apt/sources.list && \
    printf "deb http://archive.debian.org/debian-security bullseye-security main\n" >> /etc/apt/sources.list && \
    rm -f /etc/apt/sources.list.d/*.list && \
    echo 'Acquire::Check-Valid-Until "false";' > /etc/apt/apt.conf.d/99archive

# Install required packages for Python and other dependencies
RUN apt-get update && apt-get install -y --no-install-recommends \
    python3 \
    python3-pip \
    python3-dev \
    && rm -rf /var/lib/apt/lists/*

# Install Python packages
RUN pip3 install sqlalchemy openai pandas pyodbc openpyxl

ADD config/lib/libltdl.la /usr/lib/x86_64-linux-gnu/libltdl.la
# Install required packages
RUN apt-get update && apt-get install -y --no-install-recommends \
    zlib1g-dev libpng-dev \
    libzip-dev \
    gnupg2 \
    curl \
    && curl https://packages.microsoft.com/keys/microsoft.asc | apt-key add - \
    && curl https://packages.microsoft.com/config/debian/11/prod.list > /etc/apt/sources.list.d/mssql-release.list \
    && apt-get update \
    && ACCEPT_EULA=Y apt-get install -y --no-install-recommends \
    msodbcsql17 \
    unixodbc-dev \
    supervisor \
    locales \
    libmagickwand-dev \ 
    librdkafka-dev \
    && pecl install sqlsrv-5.12.0 pdo_sqlsrv-5.12.0 \
    && docker-php-ext-enable sqlsrv pdo_sqlsrv \
    && pecl install xdebug-3.2.0 \
    && docker-php-ext-enable --ini-name 30-xdebug.ini xdebug \
	&& docker-php-ext-install bcmath gd exif pcntl intl zip

RUN pecl install redis \
    && docker-php-ext-enable redis 
RUN pecl install imagick \
    && docker-php-ext-enable imagick 
RUN pecl install rdkafka \
    && docker-php-ext-enable rdkafka 
RUN apt-get clean && rm -rf /var/lib/apt/lists/*

RUN echo "en_US.UTF-8 UTF-8" > /etc/locale.gen
RUN locale-gen
RUN chmod +x /etc/init.d/supervisor
RUN update-rc.d supervisor defaults

# Get latest Composer
COPY --from=composer:latest /usr/bin/composer /usr/bin/composer

# Enable Apache modules
RUN a2enmod rewrite

# Copy Apache virtual host configuration file
ADD config/apache/apache-virtual-hosts.conf /etc/apache2/sites-available/000-default.conf
ADD config/apache/apache2.conf /etc/apache2/apache2.conf
ADD config/apache/ports.conf /etc/apache2/ports.conf
ADD config/apache/envvars /etc/apache2/envvars

# Update php.ini
ADD config/php/php.conf /usr/local/etc/php/php.ini

# Init
ADD init.sh /init.sh
RUN chmod 755 /*.sh

# Expose port 80
EXPOSE 80

CMD ["/init.sh"]
