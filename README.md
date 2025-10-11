# 🐳 Laravel Roadrunner Docker Production Setup
Продакшн среда для Laravel приложения на базе Alpine Linux с PHP 8.4, RoadRunner, MariaDB, Redis.

## 📋 Особенности
- Laravel Octane с RoadRunner для высокой производительности
- Supervisor для управления процессами, включая SSR.
- Alpine Linux для минимального размера образа
- GRPC поддержка для микросервисной архитектуры
- Production-оптимизированные настройки
- Директории `storage` и `cache` вынесены в отдельные тома. Остальные файлы проекта копируются внутрь образа.

## 🗂️ Структура проекта
- **supervisord.conf**      - Управление процессами (стартует Octane и SSR)
- **start-container**       - Entrypoint скрипт: обновляет кеши перед запуском и стартует supervisor
- **prod.rr.yaml**          - Production RoadRunner конфиг, будет скопирован в корень проекта внутри образа как `.rr.yaml`
- **prod.env**              - Production переменные окружения, будет скопирован в корень проекта внутри образа как `.env`

## ⚙️ Быстрый старт
### Сборка контейнера
1. Скопируйте директорию `docker-production` и `.dockerignore` в корень проекта
2. Заполните переменные окружения в prod.env
3. Запустите сборку образа из корня проекта, явно указав env файл:
```bash
  docker-compose -f ./docker-production/docker-compose.yml --env-file ./docker-production/prod.env build
```
### Запуск контейнера
```bash
  docker-compose -f ./docker-production/docker-compose.yml --env-file ./docker-production/prod.env up
```

Проект будет запущен на `:8000` порте, так как подразумевается, что входящий трафик будет принимать `nginx` на хосте и проксировать его на `:8000` порт. Поэтому кеширование/сжатие статики и `ssl` настраивайте на `nginx`.

