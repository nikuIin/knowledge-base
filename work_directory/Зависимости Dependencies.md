[[FastAPI.canvas|FastAPI Union]]

`Dependencies` — передача значения, нужного определенной функции в конкретный момент времени. Чаще всего создается вспомогательная функция, которая эти значения и возвращает. 

`FastAPI` позволяет обращаться/писать зависимости в параметрах функции. Первые примеры зависимостей: `Body()`, `Header()`, `Path`, `Query`. 

Зависимости в `FastAPI` – это ==всегда== `callback` типы, а именно функции и классы. 

Самый частый кейс написания зависимости — написания вспомогательной функции:

```python

def get_user_dp(user_id: int) -> UserGet:
	user = some_db_func(user_id=user_id) # достаем юзера из БД
	return UserGet(user.user_id)


@router.get('/user/{user_id}')
def get_user(
	user_id: int,
	user = Depends(get_user_dp)
):
	return user
	
```

**Важно**: как мы видим, у нас и у функции-зависимости и у ручки есть параметр `user_id`, они совпадают и `fastAPI` сам подставит параметр `user_id` из ручки в зависимость. **НО** можно не перечислять параметры функции в ручке, мы можем просто обратиться к зависимости и `fastAPI` сам попросит у пользователя данные, то есть __правильно__ писать так:

```python
from fastapi import Depends, FastAPI
from uvicorn import run


app = FastAPI()


def get_user_dp(user_id: int):
    if user_id == 1:
        return {"user": "Ваня"}
    return {"Такого нету"}


@app.get("/user/{user_id}/")
def get_user(
    user=Depends(get_user_dp),
):
    return user


if __name__ == "__main__":
    run("some:app")
```

`Depends`, вспомогательный инструмент, нам нужен как раз для того, чтобы вызывать зависимость только тогда, когда мы вызываем ручку. Иначе, если бы вы вызывали через `()` зависимость, она бы вызвалась в момент определения. А просто по названию `dependency_func_name` мы, понятное дело, получили бы объект первого класса, то есть функцию, а не ее результат. 


>[!Классное свойство Depends + Annotation]
>Когда мы хотим использовать `dependensies` и, как хорошие программисты, объявить его тип, мы по факту пишем тип 2 раза, разберем на примере `OAuth2PasswordRequestForm`:
>```python
>@app.post('/token')
>def login(login_data: OAuth2PasswordRequestForm = Depends(OAuth2PasswordRequestForm)):
>...
>```
>И тут `FastAPI` даёт нам офигительный синтаксический сахар. Если мы указываем анатацию к `Depends()`, то мы можем внутри `Depends()` **ничего не писать**, он подтянет сам все с анатации!
>```python
>@app.post('/token')
>def login(login_data: OAuth2PasswordRequestForm = Depends()):
>```


### Расположение Depends (область действий)

Расположим элементы императивно:
	1. В корне приложении (точке входа) `app` — охватывает и применяется на всех ручках приложения
	2. В роуте `route` — охватывает все ручки роута
	3. В функции (если `dependencies` не `void`)
	4. В декораторе (если `dependencies void`, e.g. валитор)


* Корень приложения `app`
Параметр `dependencies`. Как уже говорилось, `depfunc1` и `depfunc2` будут применяться во __всех__ роутах приложения `app`. 
```python
# imports...

def depfunc1():
	pass

def depfunc2():
	pass

app = FastAPI(dependencies=[ Depends(depfunc1), Depends(depfunc2) ])
```

* Роутер `route`
Тот же параметр `dependencies`, будет применяться ко всем ручкам этого роута
```python
# imports...

def depfunc1():
	pass

router = APIRouter(
	tags=["someTag"],
	...,
	dependencies= [ Depends(depfunc1) ]
)


```


* В самой функции или декораторе. 
Тут важно понимать, что мы хотим сделать. Если наша функция-зависимость не возвращает ничего, а, например проводит валидацию данных и в случае ошибки выбрасывает `raise`, то нам правильнее ее определить в декораторе. Иначе же, если функция-зависимость возвращает какой-либо параметр, ее лучше определить в самой функции

```python
# imports...
# apps and routes...

def validator_user(user: str):
    if len(user) == 1:
        raise ValueError()

def get_user(name: str, id: int):
	return {"name": name, "id": id}


@app.get("/user/", dependencies=[Depends(validator_user)])
def validate_user() -> bool:
    return True

@app.get("/user_data/")
def get_user_data(
	user = Depends(get_user)
):
	return user
```