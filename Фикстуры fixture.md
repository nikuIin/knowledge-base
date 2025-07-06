[](Conftest%20–%20общая%20конфигурация%20для%20фикстур,%20общие%20фикстуры.md)[](Conftest%20–%20общая%20конфигурация%20для%20фикстур,%20общие%20фикстуры.md)[](Conftest%20–%20общая%20конфигурация%20для%20фикстур,%20общие%20фикстуры.md)[](Conftest%20–%20общая%20конфигурация%20для%20фикстур,%20общие%20фикстуры.md)[](Conftest%20–%20общая%20конфигурация%20для%20фикстур,%20общие%20фикстуры.md)[](Conftest%20–%20общая%20конфигурация%20для%20фикстур,%20общие%20фикстуры.md)[[Pytest Тестирование.canvas|Pytest Тестирование]]
[[Тестирование. Pytest]]

`fixture` – заготовка окружения под тесты.

Зачастую несколько тестов используются в одном окружении (например тестовый массив данных, тестовая база данных, кеш) или используют одни и те же данные для тестов. 


### Фикстуры для тестовых данных
Например, какую-нибудь кастомную модель `User`.

Под эти кейсы были сделаны `фикстуры`.
С их помощью можно задать общие данные для тестов. Впрочем, это довольно хорошая пратика.

```python
from pytest import fixture
from domain.entity.user import UserBase
from uuid import UUID

@fixture
def user():
	return UserBase(
		login="my-cool-login",
		user_id=UUID('822f7bec-c0cf-47bb-ae07-f85f8840794b')
	)

# далее мы можем использовать эту фикстуру в тестах,
# передав ее в параметры теста. Этот параметр стоит воспринимать
# как объект, который возвращает фикстура

def test_user_instance(user):
	"""
	Test that user data in the valid type.
	"""
	assert isinstance(user, UserBase)
```

*Вопрос*: почему просто не вынести объект того же `user` за все тесты, то есть в переменную на уровне модуля? 
*Ответ*: потому что мы можем оперировать изменяемыми переменными `python` или их дочерними элементами. Если хотя бы 1 тест будет менять эти значения, то другие тесты будут также получать не всегда одинаковые данные. Это нарушает **главное правило** модульного тестирования – `unit` тесты изолированы друг от друга и идемпотентны.

### Фикстуры с параметризированными данными

Хорошей практикой использовать параретмизацию и в фикстурах, нежели чем писать несколько фикстур под разные типы объектов. 
Например, в одной фикстуре, `user_base`, можно определить 3 типа данных:

* `user` с валидными даннами с ролью пользователя
* `user` с валидными даннами с ролью админа
* `user` с несуществующей ролью (на выброс ошибки)

В коде это будет выглядеть так:

```python
"""
Module of testing Token class.
"""

# import project settings
from contextlib import AbstractAsyncContextManager
from contextlib import nullcontext as dont_raise
from uuid import UUID

from pydantic_core._pydantic_core import ValidationError
from pytest import fixture, raises

from domain.entities.user import UserBase

# ============================
# Fixtures
# ============================

USER_ROLE = 1
ADMIN_ROLE = 2

NO_EXISTING_ROLE = 100


@fixture(
    params=[
        (
            UUID("822f7bec-c0cf-47bb-ae07-f85f8840794b"),
            USER_ROLE,
            "my-cool-login",
            dont_raise(),
        ),
        (
            UUID("822f7bec-c0cf-47bb-ae07-f85f8840794b"),
            ADMIN_ROLE,
            "admin",
            dont_raise(),
        ),
        (
            UUID("822f7bec-c0cf-47bb-ae07-f85f8840794b"),
            NO_EXISTING_ROLE,
            "login",
            raises(NoExistsRoleException),
        ),
    ],
    ids=[
        "The right user data (user role)",
        "The right user data (admin role)",
        "No existing role_id",
    ],
)
def user_base(request) -> dict[str, UserBase | AbstractAsyncContextManager]:
    """Imitation the user base data.

    Returns:
        dict: User base data and expectation
        (context manager (null or raises context manager by pydantic)).
    """
    user_id, role_id, login, expectation = request.param
    return {
        "user": UserBase(user_id=user_id, role_id=role_id, login=login),
        "expectation": expectation,
    }


# ============================
# Tests
# ============================


def test_user_instance(user_base):
    """Test, that user instance have the right class type."""
    user, expectation = user_base.values()
    with expectation:
        assert isinstance(user, UserBase)

```

Для определения модели пользователя используется модуль `pydantic`. Ребята, писавшие этот модуль, в разы круче нас, обычных смертных. Они даже будут покруче рядовых `senior` программистов. Поэтому нам нету смысла проверять, правильно работает ли `pydantic`. Это называется *«мы доверяем модулю, => не проводим тесты на зоны, за которые отвечает этот модуль»*.

### Фикстуры для подготовки окружения

С помощью `фикстур` можно также создать обертку для каждого теста.
Пример настройки `БД` под тесты с автоматическим обновлением данные после каждого теста: [[pytest настройка фикстуры по работе с БД]]