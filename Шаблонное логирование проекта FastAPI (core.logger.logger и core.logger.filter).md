[[FastAPI.canvas|FastAPI]]
[[Шаблонный проект FastAPI]]

1. Создаём кастомные фильтры (`filter`) для нашего логера, а именно `core.logger.filter`:
```python
from logging import Formatter, Logger, getLogger
from logging.handlers import TimedRotatingFileHandler
from os import mkdir, path

from core.config import log_settings
from core.logger.filters import SensitiveWordsFilter


def get_configure_logger(filename: str, log_level: int = log_settings.log_level) -> Logger:
    """
    Configure logger with the given filename.

    Args:
        filename (str): The name of the log file.
        log_level (str): The log level to use (default taking by the cofiguration project file).

    Returns:
        Logger: The configured logger.
    """
    filename = filename.lower()
    logger = getLogger(filename)
    logger.setLevel(log_level)
    if logger.hasHandlers():
        # Avoid reconfiguring logger if it already exists
        return logger

    # Create log directory if it doesn't exist
    try:
        if not path.isdir(log_settings.log_directory):
            mkdir(log_settings.log_directory)
    except OSError as error:
        logger.error(
            "Error with creating the log directory %r: %s", log_settings.log_directory, error
        )

    log_formatter = Formatter(fmt=log_settings.log_format, datefmt=log_settings.date_format)

    safe_filename = filename.replace(".", "_") + ".log"
    log_file_path = path.join(log_settings.log_directory, safe_filename)

    # Configure and add handler and filters
    try:
        handler = TimedRotatingFileHandler(
            filename=log_file_path,
            when=log_settings.log_roating,
            backupCount=log_settings.backup_count,
            utc=log_settings.utc,
            encoding="utf-8",
        )
        handler.setFormatter(log_formatter)

        # use this filter if you want see the logs in the console
        # (import it from the file filters.py in the core/logger/ directory)
        # logger.addFilter(ColorFilter())  # noqa: ERA001

        # Add sensitive words filter
        logger.addFilter(SensitiveWordsFilter())

        logger.addHandler(handler)

        logger.debug(
            "Logger %r successfully configured. Logger level: %s. Logger file: %r",
            filename,
            log_level,
            log_file_path,
        )

    except Exception as error:
        logger.error(
            "The error is ocured during the logger configuration. Logger %r: %s",
            logger,
            error,
        )
    return logger
```

2. Создаем функцию, конфигурирующую логеры

```python
from logging import Formatter, Logger, getLogger
from logging.handlers import TimedRotatingFileHandler
from os import mkdir, path

from core.config import log_settings
from core.logger.filters import SensitiveWordsFilter


def get_configure_logger(filename: str, log_level: int = log_settings.log_level) -> Logger:
    """
    Configure logger with the given filename.

    Args:
        filename (str): The name of the log file.
        log_level (str): The log level to use (default taking by the cofiguration project file).

    Returns:
        Logger: The configured logger.
    """
    filename = filename.lower()
    logger = getLogger(filename)
    logger.setLevel(log_level)
    if logger.hasHandlers():
        # Avoid reconfiguring logger if it already exists
        return logger

    # Create log directory if it doesn't exist
    try:
        if not path.isdir(log_settings.log_directory):
            mkdir(log_settings.log_directory)
    except OSError as error:
        logger.error(
            "Error with creating the log directory %r: %s", log_settings.log_directory, error
        )

    log_formatter = Formatter(fmt=log_settings.log_format, datefmt=log_settings.date_format)

    safe_filename = filename.replace(".", "_") + ".log"
    log_file_path = path.join(log_settings.log_directory, safe_filename)

    # Configure and add handler and filters
    try:
        handler = TimedRotatingFileHandler(
            filename=log_file_path,
            when=log_settings.log_roating,
            backupCount=log_settings.backup_count,
            utc=log_settings.utc,
            encoding="utf-8",
        )
        handler.setFormatter(log_formatter)

        # use this filter if you want see the logs in the console
        # (import it from the file filters.py in the core/logger/ directory)
        # logger.addFilter(ColorFilter())  # noqa: ERA001

        # Add sensitive words filter
        logger.addFilter(SensitiveWordsFilter())

        logger.addHandler(handler)

        logger.debug(
            "Logger %r successfuly configurated. Logger level: %s. Logger file: %r",
            filename,
            log_level,
            log_file_path,
        )

    except Exception as error:
        logger.error(
            "The error is ocured during the logger configuration. Logger %r: %s",
            logger,
            error,
        )
    return logger
```

3. Вызываем наш логгер в рабочем файле:

```python
from logging import DEBUG
from pathlib import Path

from core.logger.logger import get_configure_logger

my_logger = get_configure_logger(filename=Path(__file__).stem, log_level=DEBUG)

my_logger.debug("This is my debug log")
my_logger.info("This is my info log")
my_logger.warning("This is my warning log")
my_logger.error("This is my error log")
my_logger.critical("This is my critical log")

my_logger.info("My cool api key of the grok: %s", "grok-cool-api-key")
```

**Важно** не забыть включить `log` файлы в `.gitignore`