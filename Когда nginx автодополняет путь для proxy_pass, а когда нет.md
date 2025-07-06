[[Nginx]]
[[Nginx.canvas|Nginx]] — canvas
[[nginx обратное проксирование запросов на бэкэнд  и настройка балансировки нагрузки (nginx balancer, backend reverse proxy)]]

**ВАЖНО**: если мы укажешь http://backend_servers/ <-, то у нас путь из `location` **НЕ ПОСТАВИТСЯ**. Eсли мы напишем http://backend_servers <- (без `/` на конце), то `NGINX` уже подставит `/api/` в конец (в нашем случае, так как "`/api/`" находится в `location`)

```nginx
# set the worker proccesses quantity equal to your CPU cores quantity
worker_processes auto;

http {
    include mime.types;

	# определяем здесь список серверов, которые обслуживают наше приложение
    upstream backend_servers {
	    # определяем настройки поведения балансировщика
        least_conn;
        server backend-fastapi-base-cnt:8000;
        server backend-fastapi-base-cnt:9000;
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
	        # ВАЖНО: если мы укажешь http://backend_servers/ <-, то у нас 
	        # путь из location НЕ ПОСТАВИТСЯ.
	        # если мы напишем http://backend_servers <- (без / на конце),
	        # то NGINX уже подставит /api/ в конец (в нашем случае, так как
	        # "/api/" находится в location)
            proxy_pass http://backend_servers;
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