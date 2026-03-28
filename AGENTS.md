# AGENTS.md - Laravel RoadRunner Docker Production Setup

Guidelines for AI agents and developers working with this Docker production setup for Laravel applications with RoadRunner.

## Build Commands

```bash
# Build production image
docker-compose -f ./docker-production/docker-compose.yml --env-file ./docker-production/prod.env build

# Build specific service
docker-compose -f ./docker-production/docker-compose.yml build app

# Direct Docker build
docker build -f ./docker-production/Dockerfile -t laravel-roadrunner:latest .
```

**Dependency Installation:**
- PHP dependencies installed via Composer during build (`composer install --no-dev`)
- Node.js dependencies via npm (`npm ci`)
- To update dependencies inside running container:
  ```bash
  docker-compose exec app composer update
  docker-compose exec app npm update
  ```

## Run Commands

```bash
# Start all services
docker-compose -f ./docker-production/docker-compose.yml --env-file ./docker-production/prod.env up
docker-compose up -d  # detached

# Stop services
docker-compose down
docker-compose down -v  # with volumes

# Container management
docker-compose exec app bash          # Shell access
docker-compose exec app php artisan   # Artisan commands
docker-compose ps                     # Status
docker-compose logs -f app            # Logs
```

## Testing Commands

**PHPUnit (if Laravel test suite exists):**
```bash
docker-compose exec app php artisan test
docker-compose exec app php artisan test --filter=test_example  # Single test
docker-compose exec app php artisan test --testsuite=Feature    # Test suite
docker-compose exec app php artisan test --coverage             # Coverage (needs xdebug)
```

**Frontend tests (if configured):**
```bash
docker-compose exec app npm test
docker-compose exec app npm run test:unit
```

## Linting and Formatting

**PHP linting (tools not included by default):**
- Laravel Pint (`./vendor/bin/pint`)
- PHP CS Fixer (`./vendor/bin/php-cs-fixer fix`)
- PHPStan (`./vendor/bin/phpstan analyse`)
- Psalm (`./vendor/bin/psalm`)

**JavaScript/TypeScript:**
```bash
docker-compose exec app npm run lint   # If ESLint configured
docker-compose exec app npm run format # If Prettier configured
```

**Dockerfile linting:**
```bash
hadolint docker-production/Dockerfile
```

## Code Style Guidelines

### Dockerfile
- Order commands from least to most frequently changed for layer caching
- Use `--no-cache` and clean apk cache in same RUN
- Run as non‑root user (`sail`) for application processes
- Include health checks for long‑running services
- Use executable entrypoint scripts for pre‑start logic

### YAML (docker‑compose, RoadRunner config)
- Indentation: 2 spaces, no tabs
- Quote only when needed (special characters in env vars)
- Specify exact image versions (`mariadb:12`)
- Use custom bridge networks for inter‑service communication
- Name volumes for persistent data

### PHP (Laravel)
- Follow PSR‑12 coding standard
- Use Laravel naming conventions:
  - Controllers: `PascalCase` + `Controller` suffix
  - Models: `PascalCase` singular
  - Database tables: `snake_case` plural
  - Relationships: `camelCase` methods
- Use type hints and return types where possible
- Include `declare(strict_types=1)` in PHP files

### JavaScript/TypeScript
- Use Laravel’s default ESLint configuration if present
- Group imports: external → internal → relative
- `camelCase` for variables/functions, `PascalCase` for components/classes

### Naming Conventions
- **Dockerfiles:** `Dockerfile` (no extension), `Dockerfile.dev` for development
- **Configuration:** `.yml` extension for YAML files
- **Scripts:** lowercase with hyphens (`start‑container`, `prod.env`)
- **Environment variables:** UPPERCASE with underscores (`APP_ENV`, `DB_PASSWORD`)
- **Services:** lowercase (`app`, `mariadb`, `redis`, `adminer`)
- **Containers:** descriptive with `_container` suffix (`mariadb_container`)

## Error Handling and Logging

- Use exceptions for error conditions (not return codes)
- Log with Laravel’s Log facade (`Log::error()` with context)
- All container processes log to stdout/stderr (captured by Docker)
- Supervisor redirects logs to stdout (see `supervisord.conf`)
- Services include health checks (MariaDB `healthcheck.sh`, Redis `redis‑cli ping`)





## Commit Guidelines

1. Use semantic commit messages:
   - `feat:` New feature
   - `fix:` Bug fix
   - `docs:` Documentation changes
   - `style:` Code style (formatting, missing semi‑colons)
   - `refactor:` Code refactoring
   - `test:` Adding/updating tests
   - `chore:` Maintenance tasks

2. Include scope when appropriate:
   - `feat(docker): add healthcheck to mariadb`
   - `fix(rr): correct worker memory configuration`

3. Keep subject line under 50 characters, body wrapped at 72 chars.

## Resources

- [Laravel Documentation](https://laravel.com/docs)
- [RoadRunner Documentation](https://roadrunner.dev/docs)
- [Docker Best Practices](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/)
- [Alpine Linux Wiki](https://wiki.alpinelinux.org/wiki/Main_Page)

---
*Auto‑generated; update when project conventions change.*