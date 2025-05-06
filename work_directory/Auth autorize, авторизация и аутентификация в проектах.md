[[FastAPI.canvas|FastAPI]]

В современных проектах используется `oauth2` тип аутентификации и авторизации, он включается в себя `hash` + `JWT`  + `encode`


Алгоритм получения `JWT` такой:
1. Получаем `credentionals`
2. Проверяем, что хэш пароля == хешу пароль в БД
3. Формируем `JWT`  и выдаем его

Использовать этот `JWT` можно в заголовках, а благодаря `oauth2` этап авторизации становится очень лекгим

```python
from fastapi import APIRouter, Depends, HTTPException
from fastapi.security import OAuth2PasswordBearer, OAuth2PasswordRequestForm

from core.config import auth_settings
from schemas.token import Token
from schemas.user import User, UserAuth
from utils.auth import create_jwt, verify_password

SECRET_KEY = auth_settings.secret_key
ALGORITHM = auth_settings.algorithm
ACCESS_TOKEN_EXPIRE_MINUTES = auth_settings.access_token_expire_minutes

DEFAULT_EXPIRE_MINUTES = 15

router = APIRouter(prefix="/auth", tags=["Authentication"])

# === Определяем точку аутентификации для дальнейшей авторизации ===
oauth2_scheme = OAuth2PasswordBearer(tokenUrl="/auth/token")
# ==================================================================


@router.post("/token")
async def login_for_access_token(form_data: OAuth2PasswordRequestForm = Depends(OAuth2PasswordRequestForm)):
    user = await get_user(form_data.username) # достаем откуда-то юзера
    if not user:
        raise HTTPException(status_code=400, detail="Incorrect username or password")
    if not verify_password(form_data.password, user.hashed_password):
        raise HTTPException(status_code=400, detail="Incorrect username or password")
    user_without_password = User(**user.model_dump())
    access_token = create_jwt(user_without_password)
    return Token(access_token=access_token, token_type="bearer")


@router.get("/protected")
async def protected(protected=Depends(oauth2_scheme)): # <- это и есть проверка на авторизацию пользователя. Ручка ожидает Autorize заголовок)
    return "hello!"

```