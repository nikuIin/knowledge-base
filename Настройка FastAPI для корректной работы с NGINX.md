[[Nginx.canvas|Nginx]] — канва
[[Nginx]]
[[FastAPI.canvas|FastAPI]] — канва

Когда нам нужно работать с `FastAPI` + `NGINX`? — когда нам нужно настроить проксирование запросов к веб серверу на сервер бэкэнда. 

Давайте посмотрим на конфигурацию `NGINX`, для которой нужно будет внести некоторые настройки в `FastAPI` приложении:

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

        location /api {
	        # здесь весь наш сценарий и развертывывается
	        # мы настраиваем проксирование на один из бэкэнд серверов.
	        # тут важно понимать, что /api/ из location подставится 
	        # в proxy_pass. То есть запрос на /api/cookies будет проксирован на
	        # http://backend_servers/api/cookies. И вот тут и начинается проблемы 
	        # на стороне FastAPI
            proxy_pass http://backend_servers/;
            # ^^^^^^^^^^^^^^^^
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

Чтобы корректно работал `swagger` (красивое и удобное представление `api` приложения), то нам необходимо явно указать `FastAPI` приложению путь до `openapi.json` — файла, который как раз и описывает наши ручки. Для этого в `main.py` (`app.py`, точки входа в приложении) описываем:

```python
from pathlib import Path

from fastapi import FastAPI
from uvicorn import run as server_start

from api.routes import api_router
from core.config import host_settings
from core.logger.logger import get_configure_logger

app = FastAPI(
    openapi_url="/api/openapi.json", # путь до openapi.json с учетом /api
    docs_url="/api/openapi", # переопределяем путь до docs
    redoc_url="/api/redocapi", # переопределяем путь до redoc
)

app.include_router(api_router)

logger = get_configure_logger(Path(__file__).stem)


if __name__ == "__main__":
    server_start(
        "main:app",
        reload=host_settings.is_reload,
        host=host_settings.host,
        port=host_settings.port,
    )

```