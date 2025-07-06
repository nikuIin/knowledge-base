[[docker]]
[[docker-compose.yaml]]

Есть такая задача: забилдить проект с помощью докера, чтобы он открывался одной командой на любой машине:

```bash
docker compose up --build
# или
docker-compose up --build
```

## Билд образов

Первым делом нам нужно создать `frontend` контейнер, который будет билдить наше приложение.

```Dockerfile
from node:lts-alpine3.22

workdir /app

# copy list of node modules and install it
copy ./package*.json /app/

run npm install

# copy all project and build it
copy . /app/

entrypoint ["npm", "run", "build"]
```

Ключевая роль этого образа – создать билд. Потом мы будем монтировать диск нашей хостовой машины с диском контейнера, где будет лежать готовый проект. Более нам контейнер не понадобится.

Дальше определим образ для нашего `backend-а`. Образы `postgreSQL` и `redis` будут использоваться стандартные:

* `backend`
```Dockerfile
FROM python:3.13-slim

RUN groupadd -r web && useradd -r -g web web

WORKDIR /app

ENV PYTHONDONTWRITEBYTECODE=1 \
    PYTHONUNBUFFERED=1

RUN apt-get update && apt-get install -y netcat-openbsd && pip install --upgrade pip

COPY ./requirements.txt ./

RUN pip install --no-cache-dir -r requirements.txt

COPY ./entrypoint.sh /web/entrypoint.sh
COPY . .

RUN chmod +x /web/entrypoint.sh \
    && mkdir -p logs \
    && chown -R web:web logs

RUN chown -R web:web /app

USER web

ENTRYPOINT ["/web/entrypoint.sh"]
```

* создаем конфиг для `nginx` (conf.d):
```conf
server {
    listen 80 default_server backlog=65535;
    server_tokens off;

    location /api/ {
        proxy_pass         http://game_backend;
        proxy_http_version 1.1;

        proxy_set_header   Host               $host;
        proxy_set_header   X-Real-IP          $remote_addr;
        proxy_set_header   X-Forwarded-For    $proxy_add_x_forwarded_for;
        proxy_set_header   X-Forwarded-Proto  $scheme;
        proxy_set_header   X-Forwarded-Host   $host;
        proxy_set_header   X-Request-Id       $request_id;
        proxy_set_header   X-NginX-Proxy      true;

        proxy_connect_timeout 5s;
        proxy_send_timeout    60s;
        proxy_read_timeout    60s;

        proxy_buffer_size           32k;
        proxy_buffers               4 64k;
        proxy_busy_buffers_size     128k;
        proxy_temp_file_write_size  256k;

        add_header 'Access-Control-Allow-Origin' 'localhost';
        add_header 'Access-Control-Allow-Methods' '*';
        add_header 'Access-Control-Allow-Headers' 'DNT,User-Agent,X-Requested-With,If-Modified-Since,Cache-Control,Content-Type,Range';
    }

    # Обслуживание статических файлов
    location /static/ {
        root /;
        try_files $uri $uri/ =404;
    }

    # Обслуживание фронтенда (SPA)
    location / {
        root /static;
        try_files $uri $uri/ /index.html;
    }

}
```

И nginx.conf
```conf
user  nginx;
worker_processes auto;
error_log  /var/log/nginx/error.log notice;
pid        /var/run/nginx.pid;

events {
    worker_connections 16384;
    multi_accept on;
    use epoll;
}

http {
    include       mime.types;
    default_type  application/octet-stream;
    sendfile        on;
    tcp_nopush      on;
    tcp_nodelay     on;

    keepalive_timeout  5s;
    keepalive_requests 100000;

    upstream game_backend {
        server game_backend:8000;
        keepalive 32;
    }

    include /etc/nginx/conf.d/*.conf;
}
```

Далее создаем образ `Dockerfile`:

```Dockerfile
FROM --platform=linux/amd64 nginx:1.25

COPY ./nginx.conf /etc/nginx/nginx.conf

COPY ./conf.d /etc/nginx/conf.d

WORKDIR /app
```

Следующий наш шаг – определение `docker-compose` файла:

```yml
services:
  game_backend:
    build:
      context: ./backend
      dockerfile: Dockerfile
    container_name: game_backend
    ports:
      - "8000:8000"
    env_file:
      - ./backend/.env
      - .env
    depends_on:
      - game_postgres
      - game_redis
    restart: always
    networks: # не забываем про общие сети
      - game_knb
  game_frontend:
    image: frontend-game:1.0
    build:
      context: ./frontend
      dockerfile: Dockerfile
    container_name: game_frontend
    env_file:
      - ./frontend/.env
    volumes:
      - ./static/:/app/dist/ # монтируем нашу папку с билженной папкой
  game_nginx:
    build:
      context: ./nginx
      dockerfile: Dockerfile
    container_name: game_nginx
    restart: always
    volumes:
      - ./static:/static/ # монтируем нашу сбилженную папку с папкой файлов для nginx
    ports:
      - "80:80"
    networks:
      - game_knb

  game_postgres:
    image: postgres:17.4-bookworm
    container_name: game_postgres
    ports:
      - "5432:5432"
    volumes:
      - pgdata:/var/lib/postgresql/data/
    env_file:
      - .env
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U ${POSTGRES_USER} ${POSTGRES_DB}"]
      interval: 30s
      timeout: 10s
      retries: 5
    restart: always
    networks:
      - game_knb

  game_redis:
    image: redis:7.4-bookworm
    container_name: game_redis
    ports:
      - "6379:6379"
    volumes:
      - redisdata:/data
    env_file:
      - .env
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 10s
      timeout: 3s
      retries: 3
    restart: always
    networks:
      - game_knb

volumes:
  pgdata:
  redisdata:

networks:
  game_knb:

```