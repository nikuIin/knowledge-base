[[FastAPI.canvas|FastAPI]]
[[Уровень данных Data layer (db, repository)]]

Лучшая практика обработки ошибок в FastAPI:

1. Определяем **кастомные ошибки**
2. Вызываем (зачастую они вызываются) на **уровне данных** и пробрасываем их наверх (зачастую через переопределение системных, например `DatabaseError`)
3. На сервисном уровне их пропускам и кидаем выше
4. На уровне веб-данных прописываем обработчик ошибок и обрабатываем их

Вот хороший пример:

```python
@router.post(
    "/add_city",
    response_model=dict,
    status_code=HTTPStatus.CREATED,
    responses={
        201: {"description": "Город успешно добавлен"},
        409: {"description": "Город уже существует"}
    }
)
async def add_city_endpoint(
        city: CityParams,
        city_service: CityService = Depends(get_city_service)):
    """
    Метод принимает название города и его координаты и
    добавляет в список городов для которых отслеживается прогноз погоды
    """
    logger.info(f"Received a request to add a city '{city.name}'")
    try:
        # Добавляем новый город в БД
        new_city = await city_service.add_city(city)
        # Создаем задачу обновления погоды для нового города
        create_periodic_weather_update_task(city=new_city)
    except SameCityExistsError as e:
        raise HTTPException(status_code=409, detail=str(e))

    return {"message": "Город успешно добавлен",
            "id": new_city.id,
            "name": new_city.name}


@router.get(
    "/weather",
    response_model=WeatherResponse,
    response_model_exclude_unset=True,
    responses={
        200: {"description": "Успешное получение данных о погоде"},
        404: {"description": "Город не найден"},
        503: {"description": "Сервис погоды недоступен"},
    },
)
async def get_weather_endpoint(
    coordinates: Coordinates = Depends(),
    weather_query_params: WeatherQueryParams = Depends(),
    weather_service: WeatherService = Depends(get_weather_service),
):
    """
    Метод принимает координаты и возвращает погоду в текущее время.
    Возвращаемые параметры погоды определюятся через qurey-параметры.
    """
    logger.info(f"Requesting weather for coordinates {coordinates}")
    try:
        weather = await weather_service.get_weather_now(coordinates)
    except OpenMeteoAPIError as e:
        raise HTTPException(status_code=503, detail=str(e))
    return WeatherResponse.build_response(weather, weather_query_params)
```