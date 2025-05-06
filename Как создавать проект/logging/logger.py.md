```python
"""
Configuration logger file
"""

import os

from loguru import logger
from notifiers.logging import NotificationHandler

# params for telegram logging
params = {
    "token": os.getenv("TELEGRAM_TOKEN"),
    "chat_id": os.getenv("TELEGRAM_CHAT_ID"),
}

# inititalize tg hendler
telegram_handler = NotificationHandler("telegram", defaults=params)


# initialize filter
def logger_filter(message):
    """Filter for logger

    Args:
        message (_type_): message of log

    Returns:
        _type_: is log constraint sensitive words or not. If true, this message
        doesn't be logged.
    """
    sensitive_keywords=[
        "password",   "api",      "api_key",
        "secret_key", "passport", "snils"
    ]
    return not any(keyword in message.get("message", "").lower()
                    for keyword in sensitive_keywords)

# Configurate logger

# All logs
logger.add(
    "logs/all/log_file.log",  # path
    rotation="10:00",  # when file will be rotate
    retention="7 days",  # file time live
    compression="zip",  # compression if rotation
    level="DEBUG",  # level for logging
    format="{time} {level} {message}",
    filter=logger_filter # add filter
)

# Error logs
logger.add(
    "logs/errors/log_error_file.log",  # path
    rotation="17 KB",  # how much log file can be weight
    compression="zip",  # compression if rotation
    level="ERROR",  # level for logging
    format="{time} {level} {message}",
)

# send message error to telegram
logger.add(telegram_handler, level="ERROR")
```

### No Handler, no Formatter, no Filter: one function to rule them all

[loguru](https://github.com/Delgan/loguru#no-handler-no-formatter-no-filter-one-function-to-rule-them-all)

How to add a handler? How to set up logs formatting? How to filter messages? How to set level?

One answer: the [`add()`](https://loguru.readthedocs.io/en/stable/api/logger.html#loguru._logger.Logger.add) function.

```python
logger.add(sys.stderr, format="{time} {level} {message}", filter="my_module", level="INFO")
```