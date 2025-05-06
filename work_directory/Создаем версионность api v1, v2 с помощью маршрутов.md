[[Настраиваем маршруты FastAPI. router роутеры]]

Мы можем красиво и динамично создать версии для нашей `api` в приложении:

```bash
├── app
│   ├── api
│   │   ├── __init__.py
│   │   └── v1
│   │       ├── __init__.py
│   │       ├── endpoints
│   │       │   ├── auth.py
│   │       │   └── product.py
│   │       └── routes.py
│   ├── app.py
```

В `auth.py` и `product.py` прописываем наши `router`-ы, `routes` их импортирует и вешает версионность:

```python
from fastapi import APIRouter

from api.v1.endpoints.auth import router as auth_router
from api.v1.endpoints.product import router as product_router

routers = APIRouter()
routers_list = [auth_router, product_router]

for router in routers_list:
    router.tags.append("v1")
    - [ ] routers.include_router(router, prefix="/v1")
```

`app` принимает и включает в приложение объект `routers`
```python
from fastapi import FastAPI
from uvicorn import run as uvicorn_run

from api.v1.routes import routers as api_v1_routes
from core.config import host_settings  # host configuration

app = FastAPI()
app.include_router(api_v1_routes)

if __name__ == "__main__":
    uvicorn_run(
        "app:app",
        host=host_settings.host,
        port=host_settings.port,
        reload=True,
    )
```