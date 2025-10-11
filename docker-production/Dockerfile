FROM frolvlad/alpine-glibc

LABEL maintainer="Vasiliy Rozhkov"

ARG WORKDIR="/var/www/html"
ARG WWWUSER=sail
ARG WWWGROUP=sail

WORKDIR $WORKDIR
# =========================
# 1. Базовые зависимости (оптимизировано)
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
# 2. PHP + расширения (группировка)
# =========================
RUN apk add --no-cache \
    php84 php84-cli php84-dev \
    php84-pdo_mysql php84-mbstring php84-bcmath \
    php84-exif php84-sockets php84-zip php84-xml \
    php84-gd php84-pecl-grpc php84-pecl-protobuf \
    php84-redis php84-dom php84-pcntl php84-curl \
    php84-posix php84-tokenizer php84-phar php84-sodium \
    php84-fileinfo php84-iconv php84-xmlwriter \
    && ln -sf /usr/bin/php84 /usr/bin/php

# =========================
# 3. Настройки OPcache
# =========================
RUN echo 'opcache.enable=1' > /etc/php84/conf.d/99_opcache.ini \
    && echo 'opcache.enable_cli=1' >> /etc/php84/conf.d/99_opcache.ini \
    && echo 'opcache.memory_consumption=256' >> /etc/php84/conf.d/99_opcache.ini \
    && echo 'opcache.interned_strings_buffer=16' >> /etc/php84/conf.d/99_opcache.ini \
    && echo 'opcache.max_accelerated_files=20000' >> /etc/php84/conf.d/99_opcache.ini \
    && echo 'opcache.validate_timestamps=0' >> /etc/php84/conf.d/99_opcache.ini

# =========================
# 4. Пользователь
# =========================
RUN addgroup -g 1000 $WWWGROUP \
    && adduser -u 1000 -G $WWWGROUP -s /bin/sh -D $WWWUSER

# =========================
# 5. Копируем конфигурационные файлы и создаем директории проекта
# =========================
COPY --chown=$WWWUSER:$WWWGROUP ./docker-production/prod.rr.yaml ./.rr.yaml
COPY --chown=$WWWUSER:$WWWGROUP ./docker-production/prod.env ./.env
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
# 7. Копирование файлов с кэшированием слоев
# =========================

USER $WWWUSER
# Копируем только зависимости для кэширования
COPY --chown=$WWWUSER:$WWWGROUP composer.json composer.lock ./

# Устанавливаем PHP зависимости
RUN composer install --no-dev --optimize-autoloader --no-scripts

# Копируем package.json для кэширования npm
COPY --chown=$WWWUSER:$WWWGROUP package.json package-lock.json* ./
RUN npm ci --legacy-peer-deps

# Копируем весь остальной код
COPY --chown=$WWWUSER:$WWWGROUP . .

# Frontend сборка
RUN npm run build

# Roadrunner
RUN vendor/bin/rr get-binary \
    && chmod +x rr

USER root

EXPOSE 8000/tcp

ENTRYPOINT ["start-container"]
