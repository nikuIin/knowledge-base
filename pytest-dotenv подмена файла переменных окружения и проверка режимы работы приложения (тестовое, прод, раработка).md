[[Pytest Тестирование.canvas|Pytest Тестирование]]
[[Конфигурация pytest в проекте]]

[Библиотека pytest-dotenv](https://pypi.org/project/pytest-dotenv/) — тут качаем

Понятное дело, что мы не можем использовать базовые настройки проекта. Это касается использования `API` прода, хост `базы данных` e.g. etc.

Поэтому нам обязательно «подменять» переменные окружения на тестовые данные.
1) Качаем зависимость `pytest-dotenv`

```bash
uv add pytest-dotenv
```

2) Определяем новый файл — `.test.env`

```bash
export APP_MODE=test # обазательно указываем в каком режиме работает наше приложение
export DB_URL=postgresql+asyncpg://van:cool-password@localhost:5423/my_TEST_db # определяем ТЕСТОВУЮ БД

```

3) Идем в файлик `pytest.ini` и прописываем путь до нашего `.test.env` и что мы переопределяем уже определенные значения

```ini
[pytest]
pythonpath = . src
env_override_existing_values = 1
env_files =
    ../.test.env
```

4) Думали все? Неа. Теперь идем в самый верхний `conftest.py` или в другой тест-точки-входа и прописываем проверку, в каком режиме работает приложение:

```python
from core.config import app_config

@fixture
def test_app_mode(scopy="session", autouse=True):
	assert app_config.app_mode == "test"
```

Теперь можем спокойно писать тесты. 

Я делаю проверку на `mode` в фикстурах, в которых настраивается окружение для тестов, вот пример:

```python
@async_fixture(scope="function")
async def db_helper():
    """Fixture to set up and tear down the test database."""
    assert app_settings.app_mode == ModeEnum.test
    assert "test" in db_settings.db_name

    db_helper = DatabaseHelper(url=db_settings.db_url, echo=False)
   
    # Create schemas and tables before tests
    await db_helper.create_schemas(SCHEMAS_LIST)
    await db_helper.create_tables()
    await db_helper.insert_base_data(TEST_STATEMENTS)

    yield db_helper

    # Clean up the database after each test
    await db_helper.clear_data()
```