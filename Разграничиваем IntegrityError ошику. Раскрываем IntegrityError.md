 [[FastAPI.canvas|FastAPI]]
[[SQLAlchemy]]

При работе `SQLAlchemy` + `asyncpg` при ошибках целостности будет возникать `IntegrityError` ошибка.

Появляется задача эти ошибки разграничить на:
- ошибки несуществующего внешнего ключа (`ForeignKeyViolationError`)
- ошибка уже существующих данных (`UniqueViolationError`)

Для этого нам нужно из ошибки `IntegrityError` нужно получить оригинальную ошибку и в нашем случае еще задействовать метод `__cause__`, — метод, который возвратит ошибку, которая стала причиной текущей, то есть `IntegrityError`

Вот наш код:

```python

class CountryRepository:

	async def create_country(self, ...):
		try:
		# основной код
		# ...
		
		except IntegrityError as error:
			# логируем текст ошибки
            logger.info(
                "Integrity error when creating "
                + "country and country_translate: %s",
                (country, country_translate_data),
                exc_info=error,
            )
			# если ключа не существует
            if isinstance(error.orig.__cause__, ForeignKeyViolationError):  # type: ignore
	            # далее разграничиваем по контексту ошибки
                if "flag" in str(error):
                    raise CountryIntegrityError(
                        "Can't create country, because the flag"
                        + f" with id {country.flag_id} does't exists."
                    ) from error

                elif "language" in str(error):
                    raise CountryIntegrityError(
                        "Can't create country, because the"
                        + f" language {country_translate_data.language_id}"
                        + " does't exists."
                    ) from error
		    # если запись уже есть в Бд
            elif isinstance(error.orig.__cause__, UniqueViolationError):  # type: ignore
	            # далее разграничиваем по контексту ошибки
                if "name" in str(error):
                    raise CountryIntegrityError(
                        f"Country with name '{country_translate_model.name}'"
                        + " already exists."
                    ) from error

                raise CountryIntegrityError(
                    f"Country with id {country.country_id} already exists."
                ) from error

            raise CountryIntegrityError(
                "Country integrity error. Try to change creation data"
            ) from error


			
		
		

```