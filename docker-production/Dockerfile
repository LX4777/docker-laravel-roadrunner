FROM alpine:latest

LABEL maintainer="Vasiliy Rozhkov"

ARG WORKDIR="/var/www/html"
ARG WWWUSER=sail
ARG WWWGROUP=sail

WORKDIR $WORKDIR
# =========================
# 1. Base dependencies
# =========================
RUN apk update && apk add --no-cache \
    bash git unzip curl supervisor zip su-exec \
    libpng-dev libjpeg-turbo-dev freetype-dev \
    oniguruma-dev libxml2-dev \
    mariadb-connector-c-dev \
    protobuf-dev grpc-dev protoc \
    build-base autoconf \
    nodejs npm

# =========================
# 2. PHP + extentions
# =========================
RUN apk add --no-cache \
    php85 php85-cli php85-dev \
    php85-pdo_mysql php85-mbstring php85-bcmath \
    php85-exif php85-sockets php85-zip php85-xml \
    php85-gd php85-pecl-grpc php85-pecl-protobuf \
    php85-redis php85-dom php85-pcntl php85-curl \
    php85-posix php85-tokenizer php85-phar php85-sodium \
    php85-fileinfo php85-iconv php85-xmlwriter \
    && ln -sf /usr/bin/php85 /usr/bin/php

# =========================
# 3. OPcache settings
# =========================
RUN echo 'opcache.enable=1' > /etc/php85/conf.d/99_opcache.ini \
    && echo 'opcache.enable_cli=1' >> /etc/php85/conf.d/99_opcache.ini \
    && echo 'opcache.memory_consumption=256' >> /etc/php85/conf.d/99_opcache.ini \
    && echo 'opcache.interned_strings_buffer=16' >> /etc/php85/conf.d/99_opcache.ini \
    && echo 'opcache.max_accelerated_files=20000' >> /etc/php85/conf.d/99_opcache.ini \
    && echo 'opcache.validate_timestamps=0' >> /etc/php85/conf.d/99_opcache.ini

# =========================
# 4. User
# =========================
RUN addgroup -g 1000 $WWWGROUP \
    && adduser -u 1000 -G $WWWGROUP -s /bin/sh -D $WWWUSER

# =========================
# 5. Copy config files and create project's directiories
# =========================
COPY --chown=$WWWUSER:$WWWGROUP ./docker-production/.rr.prod.yaml ./.rr.yaml
COPY --chown=$WWWUSER:$WWWGROUP protos/ ./protos/
COPY ./docker-production/start-container /usr/local/bin/start-container
COPY ./docker-production/supervisord.conf /etc/supervisor/conf.d/supervisord.conf

RUN chmod +x /usr/local/bin/start-container \
    && mkdir -p \
        bootstrap/cache \
        storage/framework/sessions \
        storage/framework/views \
        storage/framework/cache \
        storage/logs \
    && chown -R $WWWUSER:$WWWGROUP $WORKDIR \
    && chmod -R 775 storage bootstrap/cache

# =========================
# 6. Composer
# =========================
COPY --from=composer:latest /usr/bin/composer /usr/bin/composer

# =========================
# 7. Copy files with caching layers
# =========================

USER $WWWUSER
# Install PHP dependencies
COPY --chown=$WWWUSER:$WWWGROUP composer.json composer.lock ./
RUN composer install --no-dev --optimize-autoloader --no-scripts

# Install NPM dependencies
COPY --chown=$WWWUSER:$WWWGROUP package.json package-lock.json* ./
RUN npm ci --legacy-peer-deps

# Copy the project
COPY --chown=$WWWUSER:$WWWGROUP . .

# Frontend buildingg
RUN npm run build

# Install Roadrunner binary
RUN vendor/bin/rr get-binary \
    && chmod +x rr

USER root

EXPOSE 8000/tcp

ENTRYPOINT ["start-container"]
