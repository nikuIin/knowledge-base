[[python]]
[[FastAPI.canvas|FastAPI]]

Сервер для запуска FastAPI приложения

```shell
pip3 install uvicorn
```


В главном файле приложения `app.py` вешаем запуск сервера с автоматической перезагрузкой
```python
from uvicorn import run as uvicorn_run

app = FastAPI()

# ... код приложения FastAPI


if __name__ == "__main__":
	uvicorn_run("app:app", reload=True) # app — название файла, другой app — точка входа в приложение, в нашем случае то, что определяет FastAPI()
```