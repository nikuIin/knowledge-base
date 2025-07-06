[[Pytest Тестирование.canvas|Pytest Тестирование]]
[[Конфигурация pytest в проекте]]

Изначально `pytest` блокирует вывод логов и даже с флагом `-s` ничего нам не выведет.

Чтобы `pytest` мог взаимодействовать с `logger`, нужно в `pytest.ini` прописать два поля:
- `log_cli`
- `log_cli_level`

```ini
[pytest]
...
log_cli = False
log_cli_level = DEBUG
...
```



# Обзор от ИИ

Если логи `logger` не выводятся при использовании `pytest`, вероятно, необходимо настроить ведение журнала в конфигурации `pytest` или в самом `logger`. Проверьте, установлен ли параметр `log_cli` в `True` в вашем `pytest.ini` или в командной строке при запуске `pytest`. Также убедитесь, что уровень логирования в `logger` установлен достаточно низким, чтобы логи отображались.

Вот как можно настроить вывод логов `pytest` и `logger`:

1. Настройка в `pytest.ini`:

Добавьте или измените раздел `[pytest]` в файле `pytest.ini` (или в другом конфигурационном файле `pytest`) следующим образом:

Код

```
[pytest]log_cli = truelog_cli_level = INFOlog_format = %(asctime)s %(levelname)s %(name)s %(message)slog_date_format = %Y-%m-%d %H:%M:%S
```

- `log_cli = true`: Включает вывод логов в консоль.

- `log_cli_level = INFO`: Устанавливает минимальный уровень логирования, который будет выводиться.

- `log_format`: Определяет формат вывода логов.

- `log_date_format`: Определяет формат отображения даты и времени в логах.

2. Настройка в коде (если не используется конфигурационный файл):

Используйте аргументы командной строки для `pytest`:

Код

```
pytest --log-cli-level=INFO --log-format="%(asctime)s %(levelname)s %(name)s %(message)s" --log-date-format="%Y-%m-%d %H:%M:%S"
```

3. Настройка уровня логирования в `logger`:

В вашем коде, где вы настраиваете `logger`, убедитесь, что уровень логирования установлен достаточно низким, чтобы логи отображались. Например, если вы хотите видеть все логи, установите уровень:

Python

```
import logginglogger = logging.getLogger(__name__)logger.setLevel(logging.DEBUG)
```

Или, если вы используете `logging.basicConfig`:

Python

```
import logginglogging.basicConfig(level=logging.DEBUG, format="%(asctime)s %(levelname)s %(name)s %(message)s")
```

4. Проверка уровней логирования:

Убедитесь, что используемые уровни логирования в `logger` (например, `logging.debug`, `logging.info`, `logging.warning`) соответствуют или ниже уровня, установленного в `pytest.ini` или в командной строке. Используйте уровни логирования: `logging.DEBUG`, `logging.INFO`, `logging.WARNING`, `logging.ERROR`, `logging.CRITICAL`.

5. Альтернативный вариант: использование `logging.captureWarnings(True)`:

Если вы используете `warnings`, то включите `logging.captureWarnings(True)` перед настройкой логирования, чтобы перехватывать предупреждения в логи.

Python

```
import logginglogging.captureWarnings(True)logging.basicConfig(level=logging.DEBUG, format="%(asctime)s %(levelname)s %(name)s %(message)s")
```

Пример:

Python

```
import loggingimport pytestlogger = logging.getLogger(__name__)def test_example():    logger.debug("This is a debug message")    logger.info("This is an info message")    logger.warning("This is a warning message")    assert True
```

Запустив этот тест с корректно настроенным `pytest` и `logger`, вы увидите логи в консоли.

Код

```
pytest -v --log-cli-level=DEBUG
```

Это позволит увидеть все логи, включая отладочные, в консоли.

Таким образом, правильно настроив `pytest` и `logger`, вы сможете увидеть логи в консоли при запуске тестов.