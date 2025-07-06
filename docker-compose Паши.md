[[docker]]
[[docker-compose.yaml]]

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
    networks:
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
      - ./static/:/app/dist/
  game_nginx:
    build:
      context: ./nginx
      dockerfile: Dockerfile
    container_name: game_nginx
    restart: always
    volumes:
      - ./static:/static/
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