```bash
pip3 install psycopg2-binary
```

Подключение модуля и БД
```python
import psycopg2

try:
    # пытаемся подключиться к базе данных
    conn = psycopg2.connect(dbname='test', user='postgres', password='secret', host='host')
except:
    # в случае сбоя подключения будет выведено сообщение в STDOUT
    print('Can`t establish connection to database')
```

Использование команд SQL

```python
with conn.cursor() as curs:
    curs.execute('SELECT * FROM users')
    all_users = curs.fetchall() # если нужно выбрать все
    user = curs.fetchone() # когда запрос возвращает 1 элемент
```