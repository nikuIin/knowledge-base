[[FastAPI.canvas|FastAPI]]
[[Шаблонный проект FastAPI]]

Первым делом определяем общий `Base` класс для всех моделей в проекте
Также сразу объявим `TimeStampMixin`, которая будет определять поля создания и обновления данных.

```python
# func позволяет нам пользоваться функциями кластера базы данных
from sqlalchemy import func
from sqlalchemy.dialects.postgresql import TIMESTAMP
from sqlalchemy.orm import DeclarativeBase, Mapped, mapped_column


class Base(DeclarativeBase):
    pass


class TimeStampMixin:
    created_at: Mapped[TIMESTAMP] = mapped_column(
        TIMESTAMP(timezone=True),
        server_default=func.current_timestamp(),
        nullable=False,
    )
    updated_at: Mapped[TIMESTAMP] = mapped_column(
        TIMESTAMP(timezone=True),
        onupdate=func.current_timestamp(), 
        server_default=func.current_timestamp(),
        nullable=False,
    )
```


Далее, в другом файле определим модели для таких сущностей, как:
- пользователь
- токены (`refresh`)
- список заблокированных токенов (`black list`)
- роли пользователей

```python
import uuid
from datetime import datetime, timedelta

from sqlalchemy import UUID, CheckConstraint, ForeignKey, Integer, String
from sqlalchemy.dialects.postgresql import TIMESTAMP
from sqlalchemy.orm import Mapped, mapped_column

from core.config import auth_settings
from db.base_models import Base, TimeStampMixin


class Role(Base):
    __tablename__ = "role"

    role_id: Mapped[int] = mapped_column(
        Integer,
        primary_key=True,
        autoincrement=True,
    )
    name: Mapped[str] = mapped_column(
        String(256),
        nullable=False,
        unique=True,
    )

    __table_args__ = (
        CheckConstraint("length(name) > 0", name="role_name_check"),
    )


class User(Base, TimeStampMixin):
    __tablename__ = "user"

    user_id: Mapped[uuid.UUID] = mapped_column(
        UUID(as_uuid=True),
        primary_key=True,
        default=uuid.uuid4,
        nullable=False,
    )
    login: Mapped[str] = mapped_column(
        String(255), nullable=False, unique=True
    )
    password: Mapped[str] = mapped_column(String(255), nullable=False)
    role_id: Mapped[int] = mapped_column(
        Integer, ForeignKey("role.role_id"), nullable=False
    )

    __table_args__ = (
        CheckConstraint("length(login) > 0", name="user_login_check"),
        CheckConstraint("length(password) > 0", name="user_password_check"),
    )


class RefreshToken(Base, TimeStampMixin):
    __tablename__ = "refresh_token"

    refresh_token_id: Mapped[uuid.UUID] = mapped_column(
        UUID(as_uuid=True),
        primary_key=True,
        default=uuid.uuid4,
        nullable=False,
    )
    user_id: Mapped[uuid.UUID] = mapped_column(
        UUID(as_uuid=True), ForeignKey("user.user_id"), nullable=False
    )
    expire_at: Mapped[datetime] = mapped_column(
        TIMESTAMP(timezone=True),
        nullable=False,
        default=lambda: (
            datetime.now()
            + timedelta(minutes=auth_settings.refresh_token_expire_minutes)
        ),
    )


class BlackRefreshTokenList(Base):
    __tablename__ = "black_refresh_token_list"

    refresh_token_id: Mapped[uuid.UUID] = mapped_column(
        UUID(as_uuid=True),
        ForeignKey("refresh_token.refresh_token_id"),
        primary_key=True,
        nullable=False,
    )
    ban_at: Mapped[datetime] = mapped_column(
        TIMESTAMP(timezone=True), nullable=False, default=datetime.now
    )
```





















