[[FastAPI.canvas|FastAPI]]

`core` — это директория с конфигурациями.

Тут у нас лежит основной конфиг проекта — `config.py`.
Давайте посмотрим, как мы можем изящно его создавать с помощью [[Pydantic]]

```python
from pydantic import Field
from pydantic_settings import BaseSettings
from pydantic_settings.main import SettingsConfigDict


class ModelConfig(BaseSettings):
	# configurate work with env files
	# this script will run .env or ../.env
	# in every time, when the instance of this class will createdH
    model_config = SettingsConfigDict(
        case_sensitive=True,
        env_file=(".env", "../.env"),
        env_file_encoding="utf-8",
        # ignore extra vars in the env file
        extra="ignore",
    )


class HostSettings(ModelConfig):
    host: str = Field(default="localhost", validation_alias="APP_HOST")
    port: int = Field(default=8000, validation_alias="APP_PORT")


# create config instances
host_settings = HostSettings()
```