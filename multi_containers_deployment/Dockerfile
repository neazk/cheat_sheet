# Stage 1: Build app (Composer + Node)
FROM composer:2.7 AS composer
WORKDIR /app
COPY . .
RUN composer install --no-dev --optimize-autoloader

FROM node:20-alpine AS node
WORKDIR /app
COPY . .
RUN npm install && npm run build

# Stage 2: PHP-FPM runtime
FROM php:8.3-fpm-alpine

RUN apk add --no-cache \
    nginx \
    bash \
    curl \
    libzip-dev \
    libpng-dev \
    libjpeg-turbo-dev \
    oniguruma-dev \
    libxml2-dev \
    zip \
    unzip \
    redis \
    supervisor \
    && docker-php-ext-install pdo_mysql zip bcmath gd mbstring xml

WORKDIR /var/www/html

COPY --from=composer /app /var/www/html
COPY --from=node /app/public /var/www/html/public

# Copy Supervisor config
COPY supervisor.conf /etc/supervisor.d/laravel-worker.ini

COPY .env.docker .env
RUN php artisan config:cache \
    && php artisan route:cache \
    && php artisan view:cache \
    && chown -R www-data:www-data /var/www/html

EXPOSE 9000
CMD ["php-fpm"]
