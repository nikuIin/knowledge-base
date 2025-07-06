[[python]]
[[FastAPI.canvas|FastAPI]]

Понятное дело, что все маршруты не нужно хранить в обработчике `app`/`main` или как его вы захотите назвать. Для этого лучше воспользоваться роутерами: `APIRouter`

Файл `some_new_view.py`
```python
from fastapi import APIRouter

router = APIRouter(prefix='/some_new') # префикс распространятеся и ставится перед всеми ручками данного роутера

@router.get('/value/{value_id}')
async def get_value(value_id: int):
	return db_worker.get_value(value_id=value_id)
```
Файл `app.py`
```python
from fastapi import FastAPI
from project_path.some_new_view import router as some_new_router

app = FastAPI()

app.include_router(some_new_router)
```

Теперь мы сможем обращаться к ручке `/some_new_/value/123`