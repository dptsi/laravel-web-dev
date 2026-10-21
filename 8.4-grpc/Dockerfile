FROM dptsi/laravel-web-dev:8.4
# Patch for critical CVE-2026-109618
RUN echo 'session.upload_progress.enabled = Off' > /usr/local/etc/php/conf.d/cve-2026-109618.ini

# Install gRPC PHP extensions
RUN MAKEFLAGS="-j $(nproc)" pecl install grpc
RUN docker-php-ext-enable grpc
