[[Nginx]]
[[Nginx.canvas|Nginx]] — канва

Если перед нами стоит задача перенаправлять людей с одного пути на другой, не меняя у него отображения этого пути, то мы можем использовать `директиву` `rewrite`. 

Давайте разберем на примере:

```nginx
http {

  include mime.types;

  server {
    listen  8080;
    root  /Users/ivanaleksandrovci/code/test_static;

    rewrite ^/(number)\/(\d+$) /count/$2;

    location ~ ^\/count\/\d+ {
      try_files /index.html =406;
    }

    location / {
      root /Users/ivanaleksandrovci/code/test_static;
    }

    location /car {
      root /Users/ivanaleksandrovci/code/test_static;
      return 301 /index.html;
    }
  }
}

events {}
```

`Директива` `rewrite` будет у клиента (в браузере) оставлять `url` в формате `/number/[some_digit]`, но на сервер будет отправлять запрос на `url` формата `/count/[some_digit]`. Очень удобная особенность, чтобы не писать несколько `location`-s. 

Давайте разберем из чего состоит `rewrite`:
Первая часть — `регулярка`, по которой идет проверка `url`. Тут есть два блока в `()`, каждый блок в `()` `nginx` запоминает. Вторая часть — формат конвертации. Мы говорим преобразовать ссылку в  случае удовлетворения `регулярки` в формат `/count/[some_value]`, `$2` показывает нам (отсчет с 1) позицию значения, которое нужно подставить. Как раз за позиции и считаются части регулярного выражения в `()`.