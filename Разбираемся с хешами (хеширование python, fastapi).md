[[python]]
[[Python.canvas|Python]] — канва
[[FastAPI.canvas|FastAPI]] — канва


### Хеширование с рандомной солью

Самый предпочтительный алгоритм: `bcrypt`. 
Я использую его с помощью библиотеки `passlib`, модуля `context`.

```python
from passlib.context import CryptContext

context = CryptContext(schemes=["bcrypt"], deprecated="auto")

password = '123'
hash_data = context.hash(password)
context.verify(password, hash_data)
```

#TODO: узнать как соотносится соль
### Детерминированное хеширование (неизменяющиеся хеши, статичная соль)

Иногда, например для создания `ключей` в каком-либо хранилище нужно хешировать данные детерминированно. Для этого я использую, хоть и устаревший, но рабочий инструмент под такие задачи (не использовать для хеширования паролей) — `md5`. Одна из его реализаций поставляется библиотекой `hashlib`.

```python
from hashlib import md5

salt = 'my cool salt'
email = 'test@example.ru'
hash_password = md5((email + salt).encode()).hexdigest()

# валидирование совсем простое, ведь у нас статичная соль
```