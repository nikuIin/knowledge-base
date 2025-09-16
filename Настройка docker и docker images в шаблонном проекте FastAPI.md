[[FastAPI.canvas|FastAPI]]
[[Шаблонный проект FastAPI]]

1) Определяем образ для `postgres` (я создал в корне отдельную для этого папку `postgres_image)

```Dockerfile
FROM postgres:17.4-bookworm

# Add packages for configuration locales
RUN apt-get update && \
    apt-get install -y locales && \
    rm -rf /var/lib/apt/lists/*

# Generate new locale (ru_RU)
RUN localedef -i ru_RU -c -f UTF-8 -A /usr/share/locale/locale.alias ru_RU.UTF-8

# Set env for new locale
ENV LANG ru_RU.utf8
```

2) Далее определяем точку входа (для будущего образа бэкэнд части):

```bash
#!/bin/bash

set -euo pipefail

# Check for required environment variables
: "${DB_HOST:?Error: DB_HOST environment variable must be set}"
: "${DB_PORT:?Error: DB_PORT environment variable must be set}"
: "${DB_USER:=postgres}"  # Default to 'postgres' if not set
: "${DB_NAME:=postgres}"  # Default to 'postgres' if not set
: "${TIMEOUT_SECONDS:=30}"  # Default timeout for PostgreSQL connection

# Logging function with timestamp
log() {
  echo "[$(date '+%Y-%m-%d %H:%M:%S')] $1"
}

# Function to check connection using nc
check_connection() {
  nc -z -w 2 "$DB_HOST" "$DB_PORT" > /dev/null 2>&1
  return $?
}

log "Checking connection to PostgreSQL at $DB_HOST:$DB_PORT..."

# Wait for connection with timeout
start_time=$(date +%s)
while true; do
  if check_connection; then
    log "Connection to PostgreSQL successful!"
    break
  else
    current_time=$(date +%s)
    elapsed=$((current_time - start_time))
    if [ "$elapsed" -ge "$TIMEOUT_SECONDS" ]; then
      log "Error: Failed to connect to PostgreSQL within $TIMEOUT_SECONDS seconds."
      exit 1
    fi
    log "Failed to connect. Retrying in 0.5 seconds..."
    sleep 0.5
  fi
done

log "Applying Alembic migrations..."
# Run migrations using uv to manage dependencies
if ! uv run alembic upgrade head; then
  log "Error: Failed to apply Alembic migrations."
  exit 1
fi
log "Alembic migrations applied successfully!"

log "Starting FastAPI server with uvicorn..."
exec uv run src/main.py

```

3) Добавляем в `/backend` (`root dir`) файл `.dockerignore` и вносим туда необязательные для билда папки (**особенно** `.venv`, чтобы не было конфликтов окружений в `контейнере`).
```.dockerignore
.venv/
logs/
.ruff_cache/
__pycache__/

```
3) Определяем сам образ бэкэнд части
```Dockerfile
FROM python:3.13.5-slim

# add new user
RUN groupadd -r web && useradd -r -g web web

# add uv
COPY --from=ghcr.io/astral-sh/uv:latest /uv /uvx /bin/

# add nc (for entrypoint running)
RUN apt-get upgrade && apt-get update && apt-get install -y netcat-traditional

WORKDIR /app

# copy info about project dependencies
COPY uv.lock pyproject.toml .python-version /app

# install dependencies only, excluding dev dependencies
RUN uv sync --frozen --no-dev --no-install-project

COPY entrypoint.sh /app/
COPY . /app

# give execute right to all users
RUN chmod +x /app/entrypoint.sh

# create logs directory if not exists and set directory owner to "web" user
RUN mkdir -p /app/logs \
    && chown -R web:web logs

# TODO: switch to WEB user

ENTRYPOINT ["/app/entrypoint.sh"]
```

4) Строим `nginx` конфигурацию
```nginx
# set the worker proccesses quantity equal to your CPU cores quantity
worker_processes auto;

http {
    include mime.types;

    upstream backend_servers {
        least_conn;
        server backend-fastapi-base-cnt:8000;
    }

    server {
        listen 80;
        server_name localhost;

        root /var/www/static;

        # Static (frontend) files
        location / {
            root /var/www/static;
            try_files $uri $uri/ =404;
        }

        location /api/ {
            proxy_pass http://backend_servers/;
            # Don't change the request address from client
            proxy_set_header Host $host;
            # Don't change client IP
            proxy_set_header X-Real-Ip $remote_addr;

            proxy_http_version 1.1;
        }
    }
}

events {
    worker_connections 1024;
}
```

5) Создаем `image` для `nginx`
```Dockerfile
from --platform=linux/amd64 nginx:1.25

copy ./nginx.conf /etc/nginx/nginx.conf

# да, просто копируем без вызова nginx и nginx -s reload
# NGINX сам запустится с нашей конфигурацией

workdir /app
```

6) Определеяем `docker-compose.yml`

```yml
services:
  db:
    image: postgres-fastapi-base-project:1.0
    build:
      context: postgres_image/
    container_name: postgres-fastapi-base-cnt
    environment:
      POSTGRES_USER: ${DB_USER}
      POSTGRES_PASSWORD: ${DB_PASSWORD}
      POSTGRES_DB: ${DB_NAME}
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U $${DB_USER} $${DB_NAME}"]
      interval: 10s
      timeout: 5s
      retries: 5
      start_period: 10s
    volumes:
      - pgdata-fastapi-base:/var/lib/postgresql/data
    ports:
      - "5432:5432"
    networks:
      - base-fastapi-app-network
  backend:
    image: backend-fastapi-base-app:1.0
    build:
      context: backend/
    container_name: backend-fastapi-base-cnt
    volumes:
      - ./logs/:/app/logs/
    env_file:
      - .env
    depends_on:
      - db
    ports:
      - "8000:8000"
    networks:
      - base-fastapi-app-network
  nginx:
    image: nginx-fastapi-base-app:1.0
    build:
      context: nginx/
    container_name: nginx-fastapi-base-cnt
    volumes:
      - ./static:/var/www/static
    depends_on:
      - db
      - backend
    ports:
      - "80:80"
    networks:
      - base-fastapi-app-network

networks:
  base-fastapi-app-network:

volumes:
  pgdata-fastapi-base:
```

7) Билдим проект:
```bash
docker compose up -d
# или
docker-compose up -d
```


