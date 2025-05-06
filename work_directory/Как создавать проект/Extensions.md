
Хорошая практика, работая с логерами и с бд создавать классы и потом в файлах их объявлять. 

__Пример с DBModule и LoggerModule__:

```python
# Module of logger
from loguru import logger

LOGS_ROTATION = "0:00"
LOGS_RETENTION = "2 month"
LOGS_COMPRESSION = "zip"
LOGS_LEVEL = "DEBUG"
LOGS_FORMAT = "{time} {level} {message}"


LOGGER_SENSITIVE_WORDS = [
    "api",
    "key",
    "card",
    "password",
    "token",
    "secret",
    "ssn",  # Social Security Number
    "credit",
    "bank",
    "address",
    "email",
    "phone",
    "dob",  # Date of Birth
    "pin",
    "identity",
    "medical",
    "insurance",
    "confidential",
    "private",
    "proposal",
    "document",
    "location",
    "gps",
    "activity",
    "race",
    "ethnicity",
    "gender",
    "religion",
    "politics",
]


class ModuleLoger:
    def __init__(self, model_name: str):
        self.__model_name = model_name
        self.__logger = logger.bind(model=model_name)
        self.setup_logger()
        self.__logger.info(f"Loger of {model_name} model successfully created")

    def setup_logger(self):
        self.__logger.add(
            f"logs/{self.__model_name}.log",
            rotation=LOGS_ROTATION,
            retention=LOGS_RETENTION,
            compression=LOGS_COMPRESSION,
            level=LOGS_LEVEL,
            format=LOGS_FORMAT,
            filter=self.__logger_filter,
        )
        self.__logger.info(
            f"Loger of {self.__model_name} is succesfully configurated!"
        )

    def __logger_filter(self, message) -> bool:
        message_lower = message["message"].lower()
        return not any(
            keyword in message_lower for keyword in LOGGER_SENSITIVE_WORDS
        )

    def debug(self, message: str):
        self.__logger.debug(message)

    def info(self, message: str):
        self.__logger.info(message)

    def warning(self, message: str):
        self.__logger.warning(message)

    def error(self, message: str):
        self.__logger.error(message)


```

```python
# Module of DB
import psycopg
from logger import db_logger


class DBModule:
    def __init__(self, dbname, user, password, host, port):
        try:
            self.__connection = psycopg.connect(
                dbname=dbname,
                user=user,
                password=password,
                host=host,
                port=port,
            )
            db_logger.info("Connection open")
            self.__cursor = self.__connection.cursor()
            db_logger.info("Cursor has been created")
        except psycopg.Error as error:
            db_logger.error(f"Error with connecting to DB: {error}")

    def execute_query(self, query, data=None) -> None:
        try:
            if data:
                self.__cursor.execute(query, data)
                db_logger.info(
                    f"""Query succesfulle executed: {query}.
                    \nData: {data}
                    """
                )
            else:
                self.__cursor.execute(query)
                db_logger.info(f"Query successfully executed: {query}.")
            self.__connection.commit()
        except psycopg.Error as error:
            db_logger.error(f"Error with executing: {error}")
            self.__connection.rollback()

    def fetchall(self) -> list | None:
        if self.__cursor:
            results = self.__cursor.fetchall()
            db_logger.info(f"Get: {len(results)} fields;")
            return results

    def fetchone(self) -> tuple | None:
        if self.__cursor:
            return self.__cursor.fetchone()

    def fetchmany(self, quantity: int) -> list | None:
        if self.__cursor:
            if quantity < 0:
                db_logger.info(f"Asked for {quantity} fields of query.")
                return None
            results = self.__cursor.fetchmany(quantity)
            db_logger.info(f"Get: {len(results)} fields of asked {quantity}.")
            return results

    def close(self) -> None:
        if self.__cursor:
            db_logger.info("Cursor close")
            self.__cursor.close()
        if self.__connection:
            db_logger.info("Connection close")
            self.__connection.close()

```

# Модуль logger и bd:

Потом любой проект будет импортировать экземпляры, инициализированные в extenstions и их использовать. 

```python
# logger
from LoggerModule import ModuleLoger

db_logger = ModuleLoger("db_logger")
```

```python
# bd
from DBModule import DBModule
from config import Config

db = DBModule(
    Config.DB_NAME,
    Config.DB_USER,
    Config.DB_USER_PASSWORD,
    Config.DB_HOST,
    Config.DB_PORT,
)
```
