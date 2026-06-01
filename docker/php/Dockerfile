# =============================================================================
# Stage 1: Base PHP-FPM image with all extensions
# =============================================================================
FROM php:8.4-fpm-alpine AS base

LABEL maintainer="your-team@example.com"
LABEL description="Laravel 13 PHP-FPM Production-Ready Image"

# Install system dependencies
RUN apk add --no-cache \
    bash \
    curl \
    git \
    unzip \
    zip \
    libpng-dev \
    libjpeg-turbo-dev \
    libwebp-dev \
    freetype-dev \
    libzip-dev \
    icu-dev \
    oniguruma-dev \
    libxml2-dev \
    linux-headers \
    $PHPIZE_DEPS

# Install PHP extensions
RUN docker-php-ext-configure gd \
        --with-freetype \
        --with-jpeg \
        --with-webp \
    && docker-php-ext-install -j$(nproc) \
        pdo \
        pdo_mysql \
        mbstring \
        exif \
        pcntl \
        bcmath \
        gd \
        zip \
        intl \
        xml \
        opcache \
        sockets

# Install Redis extension via PECL
RUN pecl install redis \
    && docker-php-ext-enable redis \
    && rm -rf /tmp/pear

# Clean up build deps
RUN apk del $PHPIZE_DEPS \
    && rm -rf /var/cache/apk/*

# Install Composer
COPY --from=composer:2.8 /usr/bin/composer /usr/bin/composer

# =============================================================================
# Stage 2: Development image
# =============================================================================
FROM base AS development

# Copy PHP config for development
COPY php.ini /usr/local/etc/php/conf.d/99-custom.ini
COPY php-fpm.conf /usr/local/etc/php-fpm.d/zz-custom.conf

# Create non-root user to match host UID (override with --build-arg if needed)
ARG UID=1000
ARG GID=1000

RUN addgroup -g ${GID} laravel \
    && adduser -u ${UID} -G laravel -h /home/laravel -s /bin/bash -D laravel

# Set working directory
WORKDIR /var/www/html

# Ensure correct ownership
RUN chown -R laravel:laravel /var/www/html

USER laravel

# =============================================================================
# Stage 3: Production image (multi-stage, copy only what's needed)
# =============================================================================
FROM base AS production

# PHP production config
COPY php.ini /usr/local/etc/php/conf.d/99-custom.ini
COPY php-fpm.conf /usr/local/etc/php-fpm.d/zz-custom.conf
COPY opcache.ini /usr/local/etc/php/conf.d/10-opcache.ini

# Create non-root user
RUN addgroup -g 1000 laravel \
    && adduser -u 1000 -G laravel -h /home/laravel -s /bin/bash -D laravel

WORKDIR /var/www/html

# Copy application source (for production build only)
# In dev, this is mounted as a volume instead
# COPY --chown=laravel:laravel . .

# Install dependencies without dev packages
# RUN composer install --no-dev --optimize-autoloader --no-interaction

RUN chown -R laravel:laravel /var/www/html

USER laravel

EXPOSE 9000

CMD ["php-fpm"]
