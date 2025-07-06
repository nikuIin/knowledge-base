[[Тестирование. Pytest]]
[[Pytest Тестирование.canvas|Pytest]]

Чтобы `pytest` мог взаимодействовать с кодом вашего проекта, вам нужно в **корне** проекта создать файл `pytest.ini` и указать там **рабочую** папку вашего проекта.

```ini
[pytest]
pythonpath = . src
asyncio_mode = auto
log_cli = False
log_cli_level = DEBUG
env_override_existing_values = 1
env_files =
    ../.test.env

# пример регистрации меток
markers =
    product: mark a test, that relates to the product model and to the models thats have been inherit of the product model
    wine: mark a test, thats relates to the wine products (the inherit of the product class)
    grape: mark a test, thats relates to the grape
    country: mark a test, thats relates to the country
    region: mark a test, thats relates to the region
    repository: mart a test, thats relates to the repository module
    grape: mark a test, thats relates to the grape
```