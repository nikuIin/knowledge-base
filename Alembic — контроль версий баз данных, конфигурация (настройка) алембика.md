[[FastAPI.canvas|FastAPI]]
[[Уровень данных Data layer (db, repository)]]
[Разбор Артема Шумейко](https://www.youtube.com/watch?v=SD6_EPg0Aqk&list=PLeLN0qH0-mCXARD_K-USF2wHctxzEVp40&index=13)
[NotebookLM записи](https://notebooklm.google.com/notebook/bedc9e23-6395-4069-9091-e68165d85522)

Что вообще делает `alembic`? 

`Alembic` хранит знания о структуре базы данных.
А именно созданные таблицы, `constraints`, индексы, `enums`, etc.

Можно провести аналогию c `git` — `alembic` это некий контроллер версий/состояний базы данных. 

Он очень удобен, но **иногда плошает.** Поэтому важно проверять, какие изменения он вносит в базу данных.

На счет как часто нужно фиксировать изменения с помощью `alembic`, я бы провел аналогию с `git`. Фиксируем логичные блоки изменений, лучше сделать несколько миграций, но логично утвержденные. Чтобы 1 измение касалось 1 миграции. 

На счет комментариев к миграциям, я бы также придерживался хорошим практикам того же `git`.

# Инициализируем `alembic` в проекте

Исполняем в корне проекта

```bash
uv add alembic # добавляев зависимость
alembic init migrations
```

`alembic` создаст папку в корне проекта `migrations/`, там определит нужные для миграций файлы. Мы к ним еще вернемся. 
Также `alembic` создаст файл с корне проекта `alembic.ini`

Мы должны в нем кое-что поправить, а именно помощник для модуля `sys` для определения папки проекта:

```python
# ...
# sys.path path, will be prepended to sys.path if present.
# defaults to the current working directory.  for multiple paths, the path separator
# is defined by "path_separator" below.
prepend_sys_path = src # пишем здесь папку, в которой хранятся все наши файлы проекта

# timezone to use when rendering the date within the migration file
# as well as the filename.
# ...
```

Далее переходим в папку `migrations` и настраиваем файл, по подобию которого будут строяться миграции, а именно `env.py`:

```python
# обязательно отключаем автоматическую сортировку!
# ruff: noqa: I001

# импортируем наш конфиг базы дынных, нам из него нужно достать будет ссылку
from core.config import db_settings

# ВАЖНО: сначала импортируем модели нашей базы данных (файл(-ы) с ними)
# и только потом сам родительский класс Base(Model)
from db import models # type: ignore # noqa ( это чтобы линтеры и стат. анализаторы случайно не стерли наш модуль )
from db.base_models import Base

# ...
# далее добавляем строчку подключения к БД
# этой командой мы изменим ссылку, которая сейчас определена в alembic.ini
config.set_main_option(
    "sqlalchemy.url", db_settings.db_url + "?async_fallback=True"
)

# далее ищем строку target_metadata и передаем в нее
# метаданные Base
target_metadata = Base.metadata

# в методе run_migrations_online в context.configure(...) нужно добавить
# compare_server_default=True (Ссылка 1)
def run_migrations_online() -> None:
    """Run migrations in 'online' mode.

    In this scenario we need to create an Engine
    and associate a connection with the context.

    """
    connectable = engine_from_config(
        config.get_section(config.config_ini_section, {}),
        prefix="sqlalchemy.",
        poolclass=pool.NullPool,
    )

    with connectable.connect() as connection:
        context.configure(
            connection=connection,
            target_metadata=target_metadata,
            compare_server_default=True, # <- вот тут
        )

        with context.begin_transaction():
            context.run_migrations()
```

У нас должен получиться примерно такой файл:

```python
# ruff: noqa: I001
from logging.config import fileConfig

from alembic import context
from sqlalchemy import engine_from_config, pool

from core.config import db_settings

# import all models BEFORE Base model
# if you don't do this, the alembic will create the empty database
from db import models  # type: ignore # noqa
from db.base_models import Base

config = context.config

config.set_main_option(
    "sqlalchemy.url", db_settings.db_url + "?async_fallback=True" # это для работы асинхронно
)

if config.config_file_name is not None:
    fileConfig(config.config_file_name)

target_metadata = Base.metadata


def run_migrations_offline() -> None:
    """Run migrations in 'offline' mode.

    This configures the context with just a URL
    and not an Engine, though an Engine is acceptable
    here as well.  By skipping the Engine creation
    we don't even need a DBAPI to be available.

    Calls to context.execute() here emit the given string to the
    script output.

    """
    url = config.get_main_option("sqlalchemy.url")
    context.configure(
        url=url,
        target_metadata=target_metadata,
        literal_binds=True,
        dialect_opts={"paramstyle": "named"},
    )

    with context.begin_transaction():
        context.run_migrations()


def run_migrations_online() -> None:
    """Run migrations in 'online' mode.

    In this scenario we need to create an Engine
    and associate a connection with the context.

    """
    connectable = engine_from_config(
        config.get_section(config.config_ini_section, {}),
        prefix="sqlalchemy.",
        poolclass=pool.NullPool,
    )

    with connectable.connect() as connection:
        context.configure(
            connection=connection,
            target_metadata=target_metadata,
            compare_server_default=True,
        )

        with context.begin_transaction():
            context.run_migrations()


if context.is_offline_mode():
    run_migrations_offline()
else:
    run_migrations_online()
```

