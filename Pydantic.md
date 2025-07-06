
[](Pydantic%20расширения%20(типы).md)[](Pydantic%20расширения%20(типы).md)
[[python]]
[[Подсказки типов python]]
[[FastAPI.canvas|FastAPI Union]]

[Статья хабр](https://habr.com/ru/companies/amvera/articles/851642/)

> [!Когда использовать?]
> Для проверки входных данных, бизнес правила проверяем в других частяx.


Продвинутый инструмент подсказок типов. Главная его возможность — *возможность добавить простую валидацию данных* (+ заготовленные шаблоны).

### Установка
```shell
pip3 install -U "pydanitc[all]" # на продакшене устанавливать только сам pydantic и только нужные модули и валидаторы (all устанавливает все возможные, такие как EmailStr и другие)
```

### Работа с pydantic

*  Создаем класс, наследуясь от `BaseModel`
```python
from pydantic import BaseModel, ConfigDict

class SomeFieldBase(BaseModel):
    # настраиваем модель для работы с ORM
    model_config = ConfigDict(from_attributes=True)
	name: str
	age: int
```

#### Field

*Field* — мощнейший инструмент для тонкой настройки полей в `pydantic`

* `default_factory` — определяем функцию, которая будет возвращать дефолтное значение для поля
* `default` — дефолтное значение для поля
* `title` — название поля
* `description` — описание поля
* `alias` — алиас на сериализация и дессериализацию

Методы проверки для *чисел* `< <= > >=`:
* `gt` (greater thann)
* `ge` (greater than or equal)
* `lt` (less than)
* `le` (less than or equal)

Методы проверки для строк:
* `min_length` — минимальная длина строки
* `max_length` — максимальная длина строки
* `regex` — проверка на регулярку 


```python
from uuid import uuid7 # не знаю есть ли реализация uuid7 в python

class SomeFieldBase(BaseMode):

	id: int = Field(
		default_factory=labmda: uuid7.hex(),
		# ---
		# теперь к полю можно обратиться только как к uuid, а не id 
		alias="uuid",
		# ---
		title="ID поля",
		description="Уникальный идентификатор поля, \"
		 определяется автоматически"
	)
	price: float = Field(
		default=100,
		gt=0,
		ge=1,
		lt=101,
		le=100
	)
	email: str = Field(min_length=10, max_length=253, regex=r"[^@].@\.+")
```


#### Вешаем кастомную проверку на *поле*`@field_validator`

`mode` может принимать значения `before/after`:
* `before` — первоначальная проверка (сырые данные), до приведение типов `pydantic`
* `after` — проверка после приведения типов `pydantic` 

```python
from pydantic import ..., field_validator 

class Some(BaseModel):
	name: str = Field(min_length=10, max_length=100)

	@field_validator('name', mode='before'):
	@classmethod
	def check_length_name(cls, name):
		if 'ijk' in name:
			raise ValuesError()
```


#### Вешаем кастомную проверку на *весь объект* `@model_validator`

```python
from pydantic import model_validator

ALLOWED_TARRIF_TYPES = [1, 2, 3]

class Some(BaseModel):
	tarif: int = Field(default=1)
	balance: float = Field(ge=0)
	discount: float = Field(ge=0, le=0.9)

	@model_validator(mode='after')
	def validate_discount_by_tarif(self):
		if (
			self.tarif in ALLOWED_TARRIF_TYPES
			and self.tarif != 3
			and discount > 0.5
		):
			raise ValueError("Ваш тариф не позволяет иметь такую скидку")
```

#### Тактика валидации

1. Первом делом нужно провалидировать поля в просто `Field`
2. Потом навесить кастомные (если нужно) проверки в `@field_validator`. Главное не проводить проверки, которые уже были сделаны
3. Потом можно проверить весь объект полностью с помощью `@model_validator`

*Не забывай про моды after/before, они могут в некоторых случаях оптимизировать работу валадитора.*


#### Сериализация

`pydantic` позволяет круто и удобно сериализовать объекты, а именно:
* в формат `dict` (*model_dump()*) —> `type dict`
* в формат `json` (*model_dump_json()*) --> `type str`

#### Форматирование типов
`pydantic` умный и он может сам форматировать типы. Например `"123"` --> `123`. Но `123` --> `"123"` он уже откажется, потому что это уже не типичное форматирование. **Хотя важно помнить, что всегда, особенно форматирование нужно делать ЯВНО!**


###### Включение полей в `dump`-ы и в `str` выводы

На самом деле важно понимать, что не всегда все поля нужно выдавать на вывод и отправлять странствовать, например по сети. 

Самый лучший пример с полем `password`:

* `exclude`: *bool*. Исключить ли это поле из дампов
* `repr`: *bool*. Выводить ли это поле при форматировании в `str`

```python
# pydantic imports....

class Some(BaseModel):
	name: str
    password: str = Field(exclude=True, repr=True)


some = Some(name="123", password="123")
print(some) # name="123" password="123"
print(some.model_dump()) # {"name": "123"}
```

**Важно:** крч, важно просто понимать, что поля, которые никак не должны фигурировать в выводе, нужно оборачивать в `eclude=True`и`repr=False`

### Настройка всего объекта `model_config`

Мы можем настроить весь `pydantic` объект с помощью `model_config`:

- `from_attributes=True` — позволяет создавать объект модели напрямую из атрибутов Python-объектов (например, когда поля модели совпадают с атрибутами другого объекта). Чаще всего опция применяется для преобразования моделей ORM к моделям Pydantic.

- `str_to_lower, str_to_upper` - преобразование всех строк модели в нижний или верхний регистр

- `str_strip_whitespace` - cледует ли удалять начальные и конечные пробелы для типов str (аналог strip)

- `str_min_length, str_max_length` - задает максимальную и минимальную длину строки для всех строковых полей

- `use_enum_values` - cледует ли заполнять модели значениями, выбранными из перечислений, вместо того чтобы использовать необработанные значения? Это часто требуется при работе с моделями ORM, в которых колонки определены как перечисления (ENUM).

```python
class Some(BaseModel):
	model_config = ConfigDict(
		from_attributes=True,
		str_to_lower=True,
		str_strip_whitespace=True
	)
	name: str

some = Some(name=" АбВгд ")
print(some) # 'name': 'абвгд'
```
