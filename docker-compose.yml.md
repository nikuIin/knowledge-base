[[docker]]

Создаём в **корне** проекта файл `docker-compose.yaml`

```yml
# version: "v2.30.3-desktop.1" # версия docker-compose. Вроде с 2.39.? не нужно явно писать

services: # список сервисов (правил для контейнеров)
  db: # описываем контейнер для базы данных
    image: postgres:17 # указываем образ и его версию для скачивания
    container_name: postgres-cnt 
    environment: # блок объявления переменных окружения внутри контейнера
      POSTGRES_USER: ${DB_USER} # ${} — берем переменную окружения из 
								# текущего окружения хоста
      POSTGRES_PASSWORD: ${DB_PASSWORD}
      POSTGRES_DB: ${DB_NAME}
    healthcheck: # блок проверки работоспособности контейнера
      test: ["CMD-SHELL", "pg_isready -U $${DB_USER} -d $${DB_NAME}"]
      interval: 10s
      timeout: 5s
      retries: 5
      start_period: 10s
    volumes: # список volumes, используемых контейнером
      - pgdata:/var/lib/postgresql/data
    ports: # маппинг портов
      - "5432:5432"
    networks: # используемые сети контейнером
      - app-network
  backend: # описываем контейнер для бэкэнда на python
    image: python-test-server:1.1 # даём имя и версию контейнеру
    build: # помечаем, что его нужно билдить (из кастамного Dockerfile)
      context: ./ # показываем путь к Dockerfile
    container_name: python-cnt-1
    ports: # мапинг портов
      - "80:8000"
    depends_on: # показываем, от каких других контейнеров зависит текущий
      db: 
        condition: service_healthy # условие
    networks: # используемые сети
      - app-network


networks: # блок объявления сетей
  app-network:

volumes: # блок объявления volumes
  pgdata:
    external: true # говорим docker-compose, что volume наружный, то есть    он уже существует. Если укажем false или вообще убере, то он создаст том сам с префиксом нашего service.
```

После `docker compose up` будет создан сервис с именем проекта (имя директории), а также сети и тома (volumes) как `service_name_` + `имя тома/сети`
### Файл без коменатриев:

```yaml
# version: "v2.30.3-desktop.1"

services:
  db:
    image: postgres:17
    container_name: postgres-cnt
    environment:
      POSTGRES_USER: ${DB_USER}
      POSTGRES_PASSWORD: ${DB_PASSWORD}
      POSTGRES_DB: ${DB_NAME}
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U $${DB_USER} -d $${DB_NAME}"]
      interval: 10s
      timeout: 5s
      retries: 5
      start_period: 10s
    volumes:
      - pgdata:/var/lib/postgresql/data
    ports:
      - "5432:5432"
    networks:
      - app-network
  backend:
    image: python-test-server:1.1
    build:
      context: ./
    container_name: python-cnt-1
    ports:
      - "80:8000"
    depends_on:
      db:
        condition: service_healthy
    networks:
      - app-network


networks:
  app-network:

volumes:
  pgdata:
    external: true
```