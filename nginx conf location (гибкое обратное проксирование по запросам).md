[[Nginx]]
[[Nginx.canvas|Nginx]]
[[nginx.conf — главный файл конфигурации nginx]]

Давайте сначала разберемся, о чем пойдет речь. Речь пойдет о гибкой настройки путей, по которому обращается клиент, а именно разберем запросы на: 
- `/` — корневой элемент, точка входа
- `/car` — какой-то элемент с машинами
- `/admin` — несуществуюший путь
- `/redirect` — реализация `301` редиректа
- путь, удовлетворяющий регулярному выражению

**Важно**: `nginx` может сам определить пути (автоматически). Если он найдет в директории `root` папку `start`, то он даже без указания `location /start` может проксировать запросы на эту папку. Если же в ней еще и будет лежать `index.html`,  то он отправит `get` запрос на него.

Есть 2 основных метода задачи директории пути:
- `root`
- `alias`

Отличие лучше посмотреть на практике:

```nginx
http {

  include mime.types;

  server {
    listen  8080;
    root  /Users/ivanaleksandrovci/code/test_static;

	location /{
		root /Users/ivanaleksandrovci/code/test_static;
	}

    location  /car {
      # с помощью root мы также определяем корень нашего проекта, 
      # где лежат статические файлы. Но также команда root добавит в нашем случае
      # в конец файла url, указынный в контексте location, в нашем случае /car.
      # То есть (в дефолнтом случае) nginx будет искать index.html по пути
      # /Users/ivanaleksandrovci/code/test_static/car
      root  /Users/ivanaleksandrovci/code/test_static;
    }

    location /admin {
      # alias же, в отличие от root, не добавляет путь к папке из location
      # в нашем случае все пути проксируются на ту же папку, что и стандартный
      # root
      alias /Users/ivanaleksandrovci/code/test_static;
    }

  }
}

events {}

```



### Что если мы хотим найти не `index.html`?

Для этого мы можем использовать `директиву` `try_files` в контексте `location`:

```nginx
http {

  include mime.types;

  server {
    listen  8080;
    root  /Users/ivanaleksandrovci/code/test_static;

    location  /car {
      root  /Users/ivanaleksandrovci/code/test_static;
      # мы ищем, по пути root, то есть /, который мы задали на строчку выше
      # директорию car, в который ищем файл car-main.html
      # если он не был найден, то ищется файл в корне (/) проекта с именем
      # index.html, если же и он не найдет, будет вызывана ошибка 404
      try_files /car/car-main.html /index.html =404;
    }

    location /admin {
      alias /Users/ivanaleksandrovci/code/test_static;
    }

  }
}

events {}
```

Уверен, хорошей практикой будет всегда явно указывать, какие файлы мы ищем. Явное лучше, чем неявное.

### Путь, заданный регулярками

Заметил интересный момент. Если у регулярного `location` задать `alias`, то ему нужно явно в `try_files` указать, какие файлы ему искать.

Оператор `~` чувствителен к регистру, а `~*` нет. Но **важно**, что на новых `macOS` оба оператора не чувствительны к регистру [Источник](https://nginx.org/en/docs/http/ngx_http_core_module.html#location)

```nginx
http {

  include mime.types;

  server {
    listen  8080;
    root  /Users/ivanaleksandrovci/code/test_static;

	location /{
		root /Users/ivanaleksandrovci/code/test_static;
	}

    location  /car {
      # с помощью root мы также определяем корень нашего проекта, 
      # где лежат статические файлы. Но также команда root добавит в нашем случае
      # в конец файла url, указынный в контексте location, в нашем случае /car.
      # То есть (в дефолнтом случае) nginx будет искать index.html по пути
      # /Users/ivanaleksandrovci/code/test_static/car
      root  /Users/ivanaleksandrovci/code/test_static;
    }

    location ~* ^\/start[0-9]${
      root /Users/ivanaleksandrovci/code/test_static;
      try_files /index.html =404;
	}
	
  }
}

events {}
```

В нашем случае, если путь соответствует регулярке `^\/start[0-9]`, от откроется файл `index.html`, иначе выпадете `404` ошибка.
### Создаем редиректы

Редиректы можно создать с помощью конструкции: `retrun [code] [path]`

```nginx
http {

  include mime.types;

  server {
    listen  8080;
    root  /Users/ivanaleksandrovci/code/test_static;

	location /{
		root /Users/ivanaleksandrovci/code/test_static;
	}

    location  /car {
      root  /Users/ivanaleksandrovci/code/test_static;
      return 301 /index.html;
    }
  }
}

events {}
```






