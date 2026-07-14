FROM node:22-alpine AS frontend

WORKDIR /app/apps/accounting

COPY apps/accounting/package.json apps/accounting/package-lock.json ./
RUN npm ci

COPY apps/accounting/ ./
RUN npm run build


FROM composer:2 AS dependencies

WORKDIR /app

COPY modules/ ./modules/
COPY apps/accounting/ ./apps/accounting/

WORKDIR /app/apps/accounting
RUN composer install --no-dev --prefer-dist --no-interaction --no-progress --optimize-autoloader \
    && cp -aL /app/apps/accounting /application


FROM php:8.3-fpm-alpine AS runtime

RUN apk add --no-cache \
        curl \
        icu-libs \
        libpng \
        libpq \
        libzip \
        nginx \
        oniguruma \
        supervisor \
    && apk add --no-cache --virtual .build-deps \
        $PHPIZE_DEPS \
        icu-dev \
        libpng-dev \
        postgresql-dev \
        libzip-dev \
        oniguruma-dev \
    && docker-php-ext-install \
        intl \
        pdo_pgsql \
        pgsql \
        bcmath \
        zip \
        gd \
        mbstring \
    && pecl install redis \
    && docker-php-ext-enable redis \
    && apk del .build-deps

WORKDIR /var/www/html

COPY --from=dependencies --chown=www-data:www-data /application ./
COPY --from=frontend --chown=www-data:www-data /app/apps/accounting/public/build ./public/build
COPY docker/accounting/nginx.conf /etc/nginx/nginx.conf
COPY docker/accounting/supervisord.conf /etc/supervisord.conf

RUN chmod -R ug=rw storage bootstrap/cache \
    && mkdir -p /run/nginx

HEALTHCHECK --interval=30s --timeout=5s --start-period=20s --retries=3 \
    CMD wget -q -O /dev/null http://localhost/up || exit 1

CMD ["supervisord", "-c", "/etc/supervisord.conf"]
