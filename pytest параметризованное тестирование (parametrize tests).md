[[Pytest Тестирование.canvas|Pytest Тестирование]] — канва
[[Тестирование. Pytest]]

**Параметризированные тесты стоит разграничивать на ошибочные и валидные.**

Обычное параметризорованное тестирование:

```python
from pytest import mark 

@mark.parametrize(
	"x, y, res",
	[
		(1,2,0.5),
		(5,-1,-5),
	]
)
def test_devide(x,y,res):
	assert x / y == res
```

Ожидание ошибки:

```python
from pytest import mark 
from contextlib import nullcontext as no_raise
from pytest import raises

@mark.parametrize(
	"x, y, res, expectation",
	[
		(1, 2, 0.5, no_raise()),
		(1, "3d", 0,3, raises(TypeError)),
	]
)
def test_devide(x, y, res, expectation):
	with expectation:
		assert x /y == res
```
