[[FastAPI.canvas|FastAPI]]
[[Pydantic]]

Как правильно организовать обновление в приложении, написанном на FastAPI (мб не зависимо от фремворка кста)


1. Определить `pydantic-схему` для обновления поля **со всеми полями**, но они **все необязательные (optional)**
```python
from pydantic import 

class User(BaseModel):
	user_id: int
	user_name: st
	balance: float

class UserUpdate(BaseModel):
	user_id: int
	user_name: str | None = None
	balance: float | None = None
```

2. Теперь определим `path` ручку, которое будет заниматься обновлением баланса юзера и только его баланса
```python
from fastapi import FastAPI

app = FastAPI()

users = ["""Наш список пользователей (типо БД) типа User"""] 

@app.patch("/user-balance/", response_model = User)
async def update_user_balance(
	user_update: UserUpdate
):
	# получаем нашего юзера
	user = users[user_update.user_id] 
	# определяем данные, которые нужно обновить
	# (exclude_unset=True — исключаем неопределенное)
	# вернёт это функция dict с данными для обновления
	update_data = user_update.model_dump(exclude_unset=True)
	# передаем в user наши новые данные
	# укажите параметр deep=True, если нужно глубокое копирование
	# (используются списки и другие изменяемые типы)
	user = user.model_copy(update=update_data)

	# обновляем и возвращаем user-а
	users[user_update.user_id] = user
	return user
```