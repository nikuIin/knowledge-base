[[postgreSQL]]

Перед этим нужно настроить доступ к `postgreSQL`: [[Start with postgres  Download postgres Установка]]
Удобные утилиты для работы с `postgreSQL`. 

Варианты входа:

1. Стандартный
```bash
pgcli -h 192.168.66.3 -U van -d tests
```
*Хост-юзер-база_данных*

Запрашивает `password` и мы входим в систему

2. Зашиваем пароль в `~/.pgpass`
Это файл, к которому имеет доступ только `postgreSQL` и он хранит там наши пароли, очень удобен для локального применения, но не используется в продакшене.
Задаем подключение по этой маске: *"host:port:db_name:db_user:db_password"*

```bash
echo "192.168.66.3:5432:db:dbuser:FhmN2kQ3MEdDBAr" >> ~/.pgpass
```

3. Ставим дефолтные значения на уровне сессии, если они не указаны.
Если при подключении не указывать значени порта, хоста, пароля и так далее, `psql` будет брать их из переменных окружения: https://www.postgresql.org/docs/current/libpq-envars.html

Пример с паролем:

```bash
export PGPASSWORD=FhmN2kQ3MEdDBAr

pgcli -h 192.168.66.3 -U dbuser -d db # пароль не запросится
```

