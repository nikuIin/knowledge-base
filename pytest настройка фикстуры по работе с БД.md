[[Pytest Тестирование.canvas|Pytest Тестирование]]
[[Фикстуры fixture]]
[[Conftest – общая конфигурация для фикстур, общие фикстуры]]
[[Настройка базы данных db для шаблонного приложения]]

Важно быть в курсе о [[pytest-dotenv подмена файла переменных окружения и проверка режимы работы приложения (тестовое, прод, раработка)]]

Я создал такой вот файл `postgres_helper` для работы с базой данных:

```python
# ruff: noqa: I001
from pathlib import Path

# Import necessary components from SQLAlchemy for asynchronous operations
from sqlalchemy.exc import DBAPIError, IntegrityError
from sqlalchemy import text
from sqlalchemy.ext.asyncio import (
    AsyncSession,  # Represents an asynchronous database session
    async_sessionmaker,  # Factory to create async sessions
    create_async_engine,  # Function to create an asynchronous database engine
)

# Import database settings from the application's configuration
from core.config import db_settings
from core.logger.logger import get_configure_logger

# import all models BEFORE initialization the Base class
from db.models import *  # noqa
from db.base_models import Base
from db.statement import Statement

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
    async def session_dependency(self) -> AsyncSession:  # type: ignore
        """
        Provide an asynchronous database session and ensures it is closed afterwards.
        This is designed to be used as an async generator dependency.
        """
        # Use the session factory to create a new session within a context manager
        async with self.session_factory() as session:
            # Yield the session to the consumer (e.g., a FastAPI endpoint)
            yield session  # type: ignore
            # Ensure the session is closed after the consumer is done
            await session.close()

    async def create_tables(self) -> None:
        async with self.engine.begin() as conn:
            await conn.run_sync(Base.metadata.create_all)
            await conn.commit()

    async def create_schemas(self, schemas_list: tuple[str, ...]) -> None:
        async with self.engine.begin() as conn:
            for schema in schemas_list:
                await conn.execute(
                    text(f"create schema if not exists {schema}"),
                )
            await conn.commit()

    async def insert_base_data(
        self, statements: tuple[Statement, ...]
    ) -> None:
        async with self.engine.begin() as conn:
            try:
                for stmt in statements:
                    await conn.execute(stmt.statement, stmt.data)
                await conn.commit()
                logger.info("Database scheme insert omplete successfully")

            except DBAPIError:
                await conn.rollback()
                logger.error(
                    "Error inserting base scheme in the database",
                    exc_info=True,
                )

    async def clear_data(self) -> None:
        async with self.engine.begin() as conn:
            await conn.run_sync(Base.metadata.drop_all)
            await conn.commit()


# Create a global instance of the DatabaseHelper using settings from core.config
postgres_helper = DatabaseHelper(
    url=db_settings.db_url,  # Get the database URL from settings
    echo=db_settings.db_echo,  # Get the echo setting from settings
)

```

В нем я определил функцию создание `схем` (список `str`), занесения базовых данных для `БД`, разумеется создание таблиц и полная очистка `БД`.

В папке `tests` я опроделил файл `test_statements.py`, в котором написал все данные, которые я бы хотел видеть в **ТЕСТОВОЙ** базе данных.

```python
"""
The file contains the test data for database.
This file have been creating like the db/dependencies/base_statements.py file.
"""

from sqlalchemy import insert

from db.models import Country
from db.statement import Statement

# Tuple of insert statements for initial **test** data loading
TEST_STATEMENTS: tuple[Statement, ...] = (
    Statement(
        description="Insert country 'Russia'",
        statement=insert(Country),
        data={"country_id": 643, "name": "Russia"},
        check_query=lambda session: session.query(Country)
        .filter_by(country_id=643)
        .first(),
    ),
    Statement(
        description="Insert country 'Belarus'",
        statement=insert(Country),
        data={"country_id": 112, "name": "Belarus"},
        check_query=lambda session: session.query(Country)
        .filter_by(country_id=112)
        .first(),
    ),
)
```

Далее я определил общий файл для всех тестов `conftest.py` и занес туда 2 фикстуры:
- `подключение к БД`
- `проверка сессиии`

**Важно**: мы используем `pytest_asyncio` для создания асинхронных `фикстур` и `pytest.mark.asyncio` для работы с асинхронными тестами.

```python
from pytest_asyncio import fixture as async_fixture
from tests.test_statements import TEST_STATEMENTS

from core.config import ModeEnum, app_settings, db_settings
from db.dependencies.base_statements import SCHEMAS_LIST
from db.dependencies.postgres_helper import DatabaseHelper


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


@async_fixture(scope="function")
async def db_session(db_helper):
    """Fixture to provide a database session for each test."""
    async with db_helper.session_factory() as session:
        yield session
        await session.rollback()
```

Тестировать я буду репозиторий `country_repository`, создаю для этого отдельную папку и уже в ее `conftest.py` определяю получения репозитория:

```python
from pytest_asyncio import fixture as async_fixture
from sqlalchemy.ext.asyncio import AsyncSession

from repository.country_repository import CountryRepository


@async_fixture(scope="function")
async def country_repository(db_session: AsyncSession):
    """Fixture to provide a CountryRepository instance with a test session."""
    return CountryRepository(session=db_session)

```

Далее пишу уже сами тесты:
```python
# tests/test_country_repository.py

from pytest import mark, raises
from sqlalchemy import text
from sqlalchemy.ext.asyncio import (
    AsyncSession,
)
from tests.unit.constants import BELARUS_ID

from db.models import Country as CountryModel
from domain.entities.country import Country
from domain.exceptions import (
    CountryCreatingError,
    CountryDeletionError,
    CountryUpdateError,
)
from repository.country_repository import (
    CountryRepository,
)


@mark.asyncio
async def test_create_country(
    country_repository: CountryRepository, db_session: AsyncSession
):
    """Test creating a new country."""
    new_country = Country(country_id=100, name="Test Country")

    created_country = await country_repository.create_country(new_country)

    assert created_country == new_country

    async with db_session as session:
        # Verify the country was actually saved in the database
        country_model = await session.get(CountryModel, 100)
        assert country_model is not None
        # The title of the country undergoing the capitalize() process
        assert country_model.name == "Test country"

        # Check that total quantity is 3 (Russia + Belarus + Test country)
        countries_quantity = await session.execute(
            text("select count(*) from grape.country")
        )
        countries_quantity = countries_quantity.scalar_one_or_none()
        assert countries_quantity == 3


@mark.asyncio
async def test_create_country_duplicate_id(
    country_repository: CountryRepository,
):
    """Test creating a country with a duplicate ID raises an error."""
    duplicate_country = Country(
        country_id=643, name="Duplicate Country"
    )  # Russia already exists

    with raises(CountryCreatingError):
        await country_repository.create_country(duplicate_country)


@mark.asyncio
async def test_get_country(country_repository: CountryRepository):
    """Test retrieving a country by ID."""
    country = await country_repository.get_country(643)  # Russia

    assert country is not None
    assert country.country_id == 643
    assert country.name == "Russia"


@mark.asyncio
async def test_get_nonexistent_country(country_repository: CountryRepository):
    """Test retrieving a nonexistent country returns None."""
    country = await country_repository.get_country(999)

    assert country is None


@mark.asyncio
async def test_update_country(
    country_repository: CountryRepository, db_session: AsyncSession
):
    """Test updating a country."""
    updated_country = Country(country_id=643, name="Updated Russia")

    result = await country_repository.update_country(updated_country)

    assert result == updated_country

    # Verify the update in the database
    async with db_session as session:
        country_model = await session.get(CountryModel, 643)
        assert country_model is not None
        assert country_model.name == "Updated russia"


@mark.skip("Has't implemented yet")
@mark.asyncio
async def test_update_nonexistent_country(
    country_repository: CountryRepository,
):
    """Test updating a nonexistent country."""
    nonexistent_country = Country(country_id=999, name="Nonexistent Country")

    with raises(CountryUpdateError):
        await country_repository.update_country(nonexistent_country)


@mark.asyncio
async def test_delete_country(
    country_repository: CountryRepository, db_session: AsyncSession
):
    """Test deleting a country."""
    result = await country_repository.delete_country(643)  # Russia

    assert result is True

    # Verify the country was deleted
    async with db_session as session:
        country_model = await session.get(CountryModel, 643)
        assert country_model is None


@mark.asyncio
async def test_delete_nonexistent_country(
    country_repository: CountryRepository,
):
    """Test deleting a nonexistent dont raise the error"""
    await country_repository.delete_country(999)


@mark.asyncio
async def test_initial_data_loaded(db_session: AsyncSession):
    """Test that initial data is loaded correctly."""
    result = await db_session.execute(
        text("SELECT * FROM grape.country WHERE country_id = :country_id"),
        {"country_id": 643},
    )
    country = result.fetchone()

    assert country is not None
    assert country.name == "Russia"

    result = await db_session.execute(
        text("SELECT * FROM grape.country WHERE country_id = :country_id"),
        {"country_id": BELARUS_ID},
    )
    country = result.fetchone()

    assert country is not None
    assert country.name == "Belarus"
```