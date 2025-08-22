FROM dptsi/laravel-web-dev:8.4

# Install gRPC PHP extensions
RUN MAKEFLAGS="-j $(nproc)" pecl install grpc
RUN docker-php-ext-enable grpc
