# 🐳 Laravel Roadrunner Docker Production Setup
Production environment for Laravel application based on Alpine Linux with PHP 8.5, RoadRunner, MariaDB, Redis. You can use this image like a starter kit for your own image.

## 📋 Features
- Laravel Octane with RoadRunner for high performance
- Supervisor for process management, including SSR
- Alpine Linux for minimal image size
- GRPC support for microservices architecture
- Production-optimized configurations
- `storage` and `cache` directories are mounted as separate volumes. Other project files are copied into the image.

## 🗂️ Project Structure
- **supervisord.conf**      - Process management (starts Octane and SSR)
- **start-container**       - Entrypoint script: updates caches before starting and launches supervisor
- **prod.rr.yaml**          - Production RoadRunner config, will be copied to the project root inside the image as `.rr.yaml`
- **prod.env**              - Production environment variables, will be copied to the project root inside the image as `.env`

## ⚙️ Quick Start
### Building the container
1. Copy the `docker-production` directory and `.dockerignore` to the project root
2. Fill in environment variables in prod.env
3. Build the image from the project root, explicitly specifying the env file:
```bash
  docker-compose -f ./docker-production/docker-compose.yml build
```
### Running the container
```bash
  docker-compose -f ./docker-production/docker-compose.yml up
```

The project will run on port `:8000`, as it is assumed that incoming traffic will be handled by `nginx` on the host and proxied to port `:8000`. Therefore, caching/compression of static files and `ssl` should be configured on `nginx`.

