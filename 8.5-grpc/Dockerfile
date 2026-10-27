FROM dptsi/laravel-web-dev:8.5

# Install gRPC PHP extensions
RUN MAKEFLAGS="-j $(nproc)" pecl install grpc && \
    docker-php-ext-enable grpc && \
    rm -rf /tmp/pear
