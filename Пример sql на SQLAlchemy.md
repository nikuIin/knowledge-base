[[SQLAlchemy]]
[[FastAPI.canvas|FastAPI]]


Допустим, у тебя есть таблица `users` с такими полями:

```python
from sqlalchemy import Table, Column, Integer, String, MetaData, select, insert, update, delete
from datetime import datetime

metadata = MetaData()

users_table = Table(
    'users',
    metadata,
    Column('id', Integer, primary_key=True),
    Column('username', String(50), unique=True, nullable=False),
    Column('email', String(100), unique=True, nullable=False),
    Column('created_at', DateTime, default=datetime.utcnow)
)
```

### Примеры CRUD-запросов с использованием `session.execute`

#### 1. Create (создание записи):

```python
stmt = insert(users_table).values(username='test_user', email='test@example.com')
result = session.execute(stmt)
session.commit()
print(f'Created new user with ID {result.inserted_primary_key}')
```

#### 2. Read (чтение записей):

Простой выбор всех пользователей:

```python
stmt = select(users_table.c.id, users_table.c.username, users_table.c.email)
result = session.execute(stmt)
for row in result:
    print(row)
```

Выборка с фильтрацией:

```python
stmt = select(users_table).where(users_table.c.username == 'test_user')
result = session.execute(stmt)
row = result.fetchone()
if row:
    print(f"User found: {row}")
else:
    print("No such user.")
```

#### 3. Update (изменение записей):

Изменение электронной почты конкретного пользователя:

```python
stmt = (
    update(users_table).
    where(users_table.c.id == 1).
    values(email='new_email@example.com')
)
result = session.execute(stmt)
session.commit()
print(f"{result.rowcount} rows were updated.")
```

Массивное обновление нескольких записей:

```python
stmt = (
    update(users_table).
    where(users_table.c.username.startswith('test')).
    values(email='updated@example.com')
)
result = session.execute(stmt)
session.commit()
print(f"{result.rowcount} records have been updated.")
```

#### 4. Delete (удаление записей):

Удаление одной конкретной записи:

```python
stmt = delete(users_table).where(users_table.c.id == 1)
result = session.execute(stmt)
session.commit()
print(f"{result.rowcount} record deleted.")
```

Массовое удаление:

```python
stmt = delete(users_table).where(users_table.c.username.contains('test'))
result = session.execute(stmt)
session.commit()
print(f"{result.rowcount} records deleted.")
```

### Дополнительная информация:

1. Использовать `.scalars()` и `.unique()` для извлечения уникальных значений из результирующего набора:
   
   ```python
   stmt = select(users_table.c.username.distinct())
   results = session.execute(stmt).scalars().unique()
   for row in results:
       print(row)
   ```

2. Использование функций агрегирования (`COUNT`, `SUM`) с core-выражениями:

   ```python
   from sqlalchemy.sql import func

   stmt = select(func.count()).select_from(users_table)
   result = session.execute(stmt).scalar_one()
   print(f'Total number of users: {result}')
   ```

Эти примеры демонстрируют использование SQLAlchemy Core и позволяют эффективно манипулировать базой данных без прямого обращения к сессионному объекту модели.