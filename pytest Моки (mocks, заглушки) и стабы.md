[[Pytest Тестирование.canvas|Pytest Тестирование]]
[[Тестирование. Pytest]]

Зачастую ном нужно подменить вызовы функций другими значениями. Это необходимо по нескольким причинам. Например, чтобы снизить нагрузку. 
Например мы тестируем `сервисный` слой, а как мы все знаем, он взаимодействует с `репозиторием`. Нам не нужно обращаться к `БД` для тестирование `сервисов`. Даже если мы обращаемся к тестовой. Во первых это сделает тесты более долгими, во вторых они станут `интеграциоными`, а мы все таки должны проверить изолировано наш `сервисный` слой. Именно под эту задачу подходят `моки` и стабы. 
`Мы` подменяем вызовы `репозитория` фиксированными значениями и радуемся жизни. 


Давайте определим `mock` и далее будем использовать его в тестах:
* `Mock` — для синхронных методов
* `AsyncMock` — для асинхронных методов

```python
from contextlib import nullcontext as dont_raise
from unittest.mock import AsyncMock, patch
from uuid import UUID

from pytest import fixture, mark, raises
from tests.unit.constants import REDIS_TEST_USER_ID

from services.email_verification_service import EmailVerificationService


# определяем наш мок
@fixture
def auth_code_repository_mock():
    return AsyncMock()


@mark.auth_code
@mark.service
class TestEmailVerificationService:
    @mark.asyncio
    @mark.parametrize(
        "user_id, mock_value, expectation",
        [
            (
                REDIS_TEST_USER_ID,
                0,
                False,
            ),
            (
                REDIS_TEST_USER_ID,
                -2,
                False,
            ),
            (
                REDIS_TEST_USER_ID,
                6,
                True,
            ),
        ],
    )
    async def test_is_user_rate_limit(
        self,
        user_id: UUID,
        mock_value: int,
        expectation,
        auth_code_repository_mock,
    ):
	    # подменяем вызов метода
        auth_code_repository_mock.get_user_rate_limit.return_value = mock_value

        email_verification_service = EmailVerificationService(
            auth_code_repository=auth_code_repository_mock,
            fast_mail=None,  # type: ignore
        )

        assert (
            await email_verification_service._is_user_rate_limit(
                user_id=user_id
            )
            == expectation
        )

```