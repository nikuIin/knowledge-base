[[Pytest Тестирование.canvas|Pytest Тестирование]]


Что если я вам скажу, что в `FastAPI` генерируется автоматически не только документация, но и тесты?

1. Устанавливаем зависимости

```bash
uv add hypothesis
uv add schemathesis
```

2. Запускаем тесты, передав путь к `openapi.json`. Не забывайте запустить перед этим сервис.

```bash
schemathesis run http://localhost:8001/api/openapi.json
```