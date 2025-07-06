[[Nginx]]
[[Nginx.canvas|Nginx]] — канва

Давайте поговорим о заголовках, которые стоит устанавливать при обращении к `бэкэнду`, точнее при обратном проксирование  запросов к `бэкэнду`.

Устанавливаются заголовки в `nginx` на `proxy` *?запросы?* с помощью директивы `proxy_set_header`

Разбирать будет этот `конфиг`:

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
            # Устанавливаем этот заголовок для того, чтобы не менять
            # данные клиента (браузера) по тому, к какому хосту он 
            # обращается. Ведь, если он обращается на example.com,
            # а сервер крутится на server.com, то nginx перенаправит
            # запрос как раз на server.com, то есть Host заголовок 
            # будет изменен. Поэтому мы устанавливаем заголовок Host
            # равным переменной $host, которая и хранит хост, который
            # был запрошен клиентом при запросе к nginx
            proxy_set_header Host $host;
            # Don't change client IP
            # та же самая ситуация с IP адресом, мы не хотим менять IP
            # клиента на IP прокси сервера
            proxy_set_header X-Real-Ip $remote_addr;

            proxy_http_version 1.1;
        }
    }
}

events {
    worker_connections 1024;
}

```