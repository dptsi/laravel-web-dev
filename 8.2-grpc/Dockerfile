FROM dptsi/laravel-web-dev:8.2

# Install gRPC PHP extensions
RUN apk --no-cache --virtual .build-deps add \
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
    linux-headers && \
    pecl install grpc-1.70.0 && \
    apk del .build-deps

RUN docker-php-ext-enable grpc
