[[FastAPI.canvas|FastAPI]]
[[python]]

Краткая выжимка видоса + свои мнение и другие статьи про логирование.
Источники:
* [Видос Сурена про логирование](https://www.youtube.com/watch?v=dmJVJ9eFVn4&t=2s)
* [Материалы к видосу (гитхаб) от Сурена](https://github.com/mahenzon/logging-how-to/tree/master)
* [Официальная документация](https://docs.python.org/3/library/logging.html)
* [Хендлеры](https://docs.python.org/3/library/logging.handlers.html)

Что **еще** почитать:
* https://habr.com/ru/articles/766010/

Перед тем, как использовать логирование, нужно понять, принять и запомнить 2 вещи:

1. Никогда не используем `f` строчки в логировании и непроцентное форматирование через передачу параметров! Почему? Две причины:
	* Мы логи все собираем, понятное дело, для анализа. А как их группировать, если мы сразу форматируем строку? Придет нам запись user=123 и user=345, это потенциально разные строки. А если мы используем `%s`, то есть что-то такое: `message="User %s"`, то мы сможем все логи сгруппировать. 
	* Мы не всегда используем все уровни логирования. В продакшене вообще используется только уровень `warning`. Но если мы используем `f-строки` или `{}` форматирование, то оно будет всегда работать, даже если этот уровень логирования сейчас не используется. Поэтому всегда используем только такой формат:

```python
import logging

logger = logging.getLogger(__name__)

logger.info("The user %s buy product %s", user, product)
```

2. Допустим мы при логировании хотим записать выходные данные «дорогой» функции (картинка ниже), то **функция все равно выполнится**
```python
def expensive():
	sleep(0.7) # 0.7 секунды
	return 10

logging.info("Expensive data: %s", expensive())
```

Поэтому есть в `logging` специальная проверка:
```python
if logger.isEnabledFor(logging.INFO):
	logging.info("Expensive data: %s", expensive())
```

И тут уже мы пропустим логирование, удобно!

**НО** без фанатизма, иногда `isEnabledFor` может быть дороже, чем просто вызов той функции. Поэтому хорошо бы оценить логи, посмотреть где вызываются «дорогие» функции и записать все в **отдельную переменную**

```python
is_info_active = logger.isEnabledFor(logging.INFO)

if is_info_active:
	logging.info("Expensive data: %s", expensive())
```


# Как использовать логирование?

Хорошая и обычная практика, просто в каждом, да, каждом модуле импортируем `logging`, передаем логгер в переменную `log/logger` и живем в сказке. Хотя есть еще одна хорошая практика, а именно настроить логер, вот мои настройки:

```python
import logging
import logging.handlers

def configure_logging(
    logger: logging.Logger,
    filename: str,
    level=logging.INFO,
):
    time_handler = logging.handlers.TimedRotatingFileHandler(
        filename=f"log_{filename}",
        when="midnight", # обновляем каждую полуночь (если указан параметр 
					     # atTime, то не используется)
        backupCount=30,
        utc=True,
    )
    # logger.addHandler(handler)

    logging.basicConfig(
        level=level,
        # filename=f"log_{filename}", # уже указываем в handler
        # filemode="at",              # иначе будет дублирование логов
        handlers=[time_handler],
        datefmt="%Y-%m-%d %H:%M:%S",
        format="[%(asctime)s.%(msecs)03d] %(module)10s:%(lineno)-3d %(levelname)-7s - %(message)s",
    )
```

Еще можно про настройки почитать в официальной документации: https://docs.python.org/3/library/logging.html#logrecord-attributes

А также о других **хендлерах**: [Хендлеры](https://docs.python.org/3/library/logging.handlers.html)

И эту конфигурацию также вызываем в каждом из модулей, передавая сам логер. 


# Прокидываем `traceback` ошибки в логи

Есть в логах просто восхитительная возможность прокинуть ошибку, то есть не просто сказать *«У нас здесь сломалось что-то»*, а буквально передать в логи ошибку и причем на разных уровнях!

Но пока поговорим о `log.exception()`:
Если мы не хотим заморачиваться, мы просто в блоке `try/except`, ну точнее именно в `except` можем прописать `log.exception("OH MY GOD WE HAVE THE ERROR")`

```python
known_weather = {
    "sochi": {"rain_chance": 23},
}


def get_weather(city: str) -> dict | None:
    try:
        return known_weather[city.lower()]
    except KeyError:
        # log.warning("Could not find city", exc_info=True)
        log.exception("Could not find city")
        return None


def main():
    configure_logging(level=logging.DEBUG)
    log.warning("Hello! Starting main")
    get_weather("Moscow")
    log.warning("Bye! Finished main")

if __name__ == "__main__":
    main()
```

Мы увидим **подробную ошибку**:
```bash
[2025-05-04 18:03:02.994]       main:44  WARNING - Hello! Starting main
[2025-05-04 18:03:02.994]       main:38  ERROR   - Could not find city
Traceback (most recent call last):
  File "/Users/ivanaleksandrovci/logging_tests/main.py", line 35, in get_weather
    return known_weather[city.lower()]
           ~~~~~~~~~~~~~^^^^^^^^^^^^^^
KeyError: 'moscow'
[2025-05-04 18:03:02.998]       main:46  WARNING - Bye! Finished main
```

Но, понятное дело, это преднастройка и мы можем использовать ее вообще на любом уровне, передав в параметр `ext_info` значение `True`: 
```python
log.warning("ERROR!!!", exc_info=True)
```

И самое интересное, что мы можем передавать в `exc_info` ошибки, а ошибки несут как и сообщение, так и полный их `traceback`. Посмотрим на данный пример:

```python
def some_func() -> dict | None:
    zero_div_err = None
    try:
        1 / 0
    except ZeroDivisionError as e:
        zero_div_err = e

    log.debug(
        "Zero division error: %r",
        zero_div_err,
        exc_info=zero_div_err,
    )


def main():
    configure_logging(level=logging.DEBUG)
    log.warning("Hello! Starting main")
    some_func()
    log.warning("Bye! Finished main")


if __name__ == "__main__":
    main()
```

Тут информация о `zero_div_err` передается в логирование вне блока `try/except`, но `traceback` ошибки все равно будет передан в логах, так как переменная `zero_div_err` несет в себе нее

```bash
[2025-05-04 18:07:30.564]       main:36  WARNING - Hello! Starting main
[2025-05-04 18:07:30.564]       main:27  DEBUG   - Zero division error: ZeroDivisionError('division by zero')
Traceback (most recent call last):
  File "/Users/ivanaleksandrovci/logging_tests/main.py", line 23, in some_func
    1 / 0
    ~~^~~
ZeroDivisionError: division by zero
[2025-05-04 18:07:30.571]       main:38  WARNING - Bye! Finished main
```

# Возможные проблемы

* [Дублируются логи](https://ru.stackoverflow.com/questions/1278887/%D0%98%D0%B7%D0%B1%D0%B5%D0%B3%D0%B0%D0%BD%D0%B8%D0%B5-%D0%B4%D1%83%D0%B1%D0%BB%D0%B8%D1%80%D0%BE%D0%B2%D0%B0%D0%BD%D0%B8%D1%8F-%D0%B2%D1%8B%D0%B2%D0%BE%D0%B4%D0%B8%D0%BC%D1%8B%D1%85-%D1%81%D0%BE%D0%BE%D0%B1%D1%89%D0%B5%D0%BD%D0%B8%D0%B9-%D0%BB%D0%BE%D0%B3%D0%B3%D0%B5%D1%80%D0%B0-%D0%B2-%D0%BC%D0%BE%D0%B4%D1%83%D0%BB%D0%B5-logging)
