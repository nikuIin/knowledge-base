[[Nginx]]
[[Nginx.canvas|Nginx]] — канфа
[[nginx.conf — главный файл конфигурации nginx]]


Определяем наш статический проект (определим только `index.html`, например по пути:

```bash
/Users/ivanaleksandrovci/code/test_static
```

Определим основной конфиг нашего проекта:

```nginx
http {
  server {
    listen 80;
    root /Users/ivanaleksandrovci/code/test_static/;
  }
}

events {}

```

Мы указали, что наш сервер слушается на порту 80 (стандартный `HTTP` порт), далее мы указали `root` директорию. 
Таким образом мы говорим `nginx`, в какой директории мы работаем и он сам находит файл `index.html` и проксирует запросы с `/` на этот файл.