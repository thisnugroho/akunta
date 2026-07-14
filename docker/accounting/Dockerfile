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


FROM php:8.3-fpm-alpine AS php-fpm

RUN apk add --no-cache \
        curl \
        icu-libs \
        libpng \
        libpq \
        libzip \
        oniguruma \
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

RUN chmod -R ug=rw storage bootstrap/cache

USER www-data

HEALTHCHECK --interval=30s --timeout=5s --start-period=20s --retries=3 \
    CMD php-fpm -t || exit 1

CMD ["php-fpm"]


FROM nginx:1.27-alpine AS nginx

COPY docker/accounting/nginx.conf /etc/nginx/conf.d/default.conf
COPY --from=dependencies /application/public /var/www/html/public
COPY --from=frontend /app/apps/accounting/public/build /var/www/html/public/build

HEALTHCHECK --interval=30s --timeout=5s --start-period=20s --retries=3 \
    CMD wget -q -O /dev/null http://localhost/up || exit 1
