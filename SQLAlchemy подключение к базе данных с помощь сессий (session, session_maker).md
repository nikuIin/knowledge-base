Давайте сначала разберемся, почему нам стоит создавать `сессии`, а не использовать `engine` для подключения к `БД`.

`session` имеет приоритеты относительно `engine`, такие как:
- большая чистота кода. Вам не нужно переносить и прописывать одинаковые параметры в каждом вызове и обработке данных, связанными с `БД`.
- работа с транзакциями. `session` позволяют гибко управлять транзакциями
- автоматическая использование соединений. В отличии от `engine`, `session` сама управляет подключениями. А именно сама берет из пула и сама в него ее возвращает


**Важное уточнение:** фабрика сессий должна каждый раз выдавать новую сессию. Это позволяет изолировать друг от друга запросы к `БД`, => решить сразу несколько проблем. 

`async_session_maker`, на реализацию которого мы позже посмотрим, каждый раз возвращает экземпляр `AsyncSession`, что нам как раз и нужно.

Пишем класс `db_helper`, который конфигурирует `engine` для подключения к `БД` и реализует `зависимость` `session_dependency()`, которая является `генератором` и возвращает каждый раз экземпляр `AsyncSession`:

```python
from pathlib import Path

# Import necessary components from SQLAlchemy for asynchronous operations
from sqlalchemy.ext.asyncio import (
    AsyncSession,  # Represents an asynchronous database session
    async_sessionmaker,  # Factory to create async sessions
    create_async_engine,  # Function to create an asynchronous database engine
)

# Import database settings from the application's configuration
from core.config import db_settings
from core.db.models import Base
from core.logger.logger import get_configure_logger

logger = get_configure_logger(Path(__file__).stem)


# Helper class to manage database connections and sessions
class DatabaseHelper:
    def __init__(self, url: str, echo: bool = False):
        """Initialize the DatabaseHelper with a database URL.

        Args:
            url (str): The database connection URL.
            echo (bool): If True, SQLAlchemy will log all SQL statements.
        """
        # Create the asynchronous database engine
        self.engine = create_async_engine(
            url=url,  # The database connection string
            echo=echo,  # Enable/disable logging of SQL queries
        )
        # Create a factory for generating new asynchronous sessions
        self.session_factory = async_sessionmaker(
            bind=self.engine,  # Bind the session factory to the engine
            autoflush=False,  # Disable autoflush (commits/flushes happen explicitly)
            autocommit=False,  # Disable autocommit (transactions must be committed explicitly)
            expire_on_commit=False,  # Prevent objects from being expired after a commit
        )

    # Async generator method to provide database sessions
    # Useful as a dependency in web frameworks (like FastAPI Depends)
    async def session_dependency(self) -> AsyncSession:
        """
        Provide an asynchronous database session and ensures it is closed afterwards.
        This is designed to be used as an async generator dependency.
        """
        # Use the session factory to create a new session within a context manager
        async with self.session_factory() as session:
            # Yield the session to the consumer (e.g., a FastAPI endpoint)
            yield session
            # Ensure the session is closed after the consumer is done
            await session.close()

    async def create_tables(self) -> None:
        async with self.engine.begin() as conn:
            await conn.run_sync(Base.metadata.create_all)
            await conn.commit()

    async def drop_database(self) -> None:
        async with self.engine.begin() as conn:
            await conn.run_sync(Base.metadata.drop_all)
            await conn.commit()


# Create a global instance of the DatabaseHelper using settings from core.config
db_helper = DatabaseHelper(
    url=db_settings.db_url,  # Get the database URL from settings
    echo=db_settings.db_echo,  # Get the echo setting from settings
)
```