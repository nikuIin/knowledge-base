[[SQLAlchemy]]
[[FastAPI.canvas|FastAPI]]

В `SQLAlchemy` создать таблицу можно двумя способами:
- с помощью метода `Table`
- с помощью классов `BaseModel` (предпочтительнее)

Более похож на естественную разработку второй метод, следовательно мы и будем его использовать в наших проектах.


Для начала нам нужно создать базовый класс, от которого будут наследовать все модели. Он сам будет наследоваться от класса `DeclarativeBase`. Хорошая практика делать поферх свой некий класс, например `BaseModel`, в которой будет прописываться `metainfo` для всех моделей `БД`.

```python
from sqlalchemy.orm import DeclarativeBase

class BaseModel(DeclarativeBase):
	pass
```

В `BaseModel` можно будет описать `методы`, которые свойственны всем таблицам, такие как `dict()`, `pydantic()` (для перевода в `pydantic` модель), `json`. 

Вот пример реализации этих методов:

```python
from sqlalchemy.orm import DeclarativeBase
from pydantic import BaseModel as PydanticBaseModel
from json import dumps as json_dumps

class BaseModel(DeclarativeBase):
	def dict(self):
		"""Convert model instance to a dictionary."""
		return {c.name: getattr(self, c.name) for c in self.__table__.columns}

	def pydantic(self, pydantic_model: type[PydanticBaseModel]) -> PydanticBaseModel:
	"""Convert model instance to a Pydantic model."""
	return pydantic_model(**self.dict())

	def json(self) -> str:
	"""Convert model instance to a JSON string."""
	return json.dumps(self.dict(), default=str)
```

Также хорошая практика создавать миксины (`mixin`):

```python
from sqlalchemy import TIMESTAMP
from sqlalchemy import func # для обращения к функциям кластера
from sqlalchemy.orm import DeclarativeBase, Mapped, mapped_column


class Base(DeclarativeBase):
    pass


class TimestampMixin:
    """
    Миксин для автоматического управления метками времени создания и обновления записи.
    :param created_at: Время создания записи, устанавливается сервером по умолчанию.
    :param updated_at: Время последнего обновления записи, автоматически обновляется при изменении.
    """

    created_at: Mapped[TIMESTAMP] = mapped_column(
        TIMESTAMP(timezone=True), server_default=func.now(), nullable=False
    )
    updated_at: Mapped[TIMESTAMP] = mapped_column(
        TIMESTAMP(timezone=True), server_default=func.now(), onupdate=func.now(), nullable=False
    )


class TimestampMixin:
    """
    Миксин для автоматического управления метками времени создания и обновления записи.
    :param created_at: Время создания записи, устанавливается сервером по умолчанию.
    :param updated_at: Время последнего обновления записи, автоматически обновляется при изменении.
    """

    created_at: Mapped[TIMESTAMP] = mapped_column(TIMESTAMP(timezone=True), server_default=func.now(), nullable=False)
    updated_at: Mapped[TIMESTAMP] = mapped_column(
        TIMESTAMP(timezone=True), server_default=func.now(), onupdate=func.now(), nullable=False
    )
```

## Определяем свои модели

Для того, чтобы определять уже свои модели, нам понадобятся типы, класс `Mapped` и метод `mapped_column()`
Давайте их импортируем и напишем первую модель:

```python
# ... (предыдущий код (смотреть «важно»))

class MyCoolUser(BaseModel, TimestampMixin):
	__tablename__ = "user" # задаем название таблицы
	user_id: Mapped[int] = mapped_column(primary_key=True)
	name: Mapped[str] # SQLAlchemy создаст столбец `name`
```

Далее, мы можем создавать объекты `MyCoolUser`.

**Важно:** я думаю хорошей идеей будет размещать все `SQLAlchemy` модели в одном файле, условном `models`. Это делается потому, что, если вы запускаете методы создания таблиц с помощью алхимии или дропа, то есть `create_all()` и `drop_all()` соответственно, то алхимия удалит таблицы, которые успели иницализироваться. То есть, мы должны быть уверены, что все классы, наследующиеся от `Base`, успешно инициализировались. 

Поэтому стоит объявлять все модели в одном файлике или быть уверенным, что они инициализируются первыми. (Пример `db_helper`, который инициализирует модели и работает с сессиями: [[Настройка базы данных db для шаблонного приложения]])

**Либо** импортируйте в `alembic` или в точку входа, где генерируются модели (таблицы) сначала файл со всеми таблицами (классы, которые наследуются от `Base`), а потом уже сам `Base`.