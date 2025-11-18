#python

[[Python.canvas|Python canvas]]
[[Декораторы в python]]
[https://parseltongue.co.in/understanding-the-power-of-pythons-wraps-decorator/#:~:text=Why%20Do%20We%20Need%20%40wraps,replace%20the%20original%20function's%20metadata.]

Когда мы создаем свой декаратор, что мы, что логично и обычно, прописываем внутри фукнции внутреннюю функцию `wrapper` и в ней вызываем наши функции. 

Проблема в том, что метаданные о функции при вызове в `wrapper` меняются. И чтобы их сохранить, а также сохранить нотации и имя, нужно использовать декаратор `@wraps`из модуля `functools` на саму нашу обертку `wrapper`.

```python
from functools import wraps

def handle_deal_errors(func):
    @wraps(func)
    async def wrapper(*args, **kwargs):
        try:
            return await func(*args, **kwargs)
        except (
            DealLeadNotFoundError,
            DealManagerNotFoundError,
            DealSaleStageNotFoundError,
            DealLostReasonNotFoundError,
        ) as error:
            raise HTTPException(
                status_code=HTTP_404_NOT_FOUND, detail=str(error)
            ) from error
        except DealAlreadyExistsError as error:
            raise HTTPException(
                status_code=HTTP_409_CONFLICT, detail=str(error)
            ) from error
        except (DealError, DealDBError) as error:
            raise HTTPException(
                status_code=HTTP_500_INTERNAL_SERVER_ERROR, detail=str(error)
            ) from error

    return wrapper

```

