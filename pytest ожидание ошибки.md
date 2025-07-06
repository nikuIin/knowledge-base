[[Pytest Тестирование.canvas|Pytest Тестирование]] — canva
[[Тестирование. Pytest]]

```python
import pytest

def test_devide_zero():
	with pytest.raises(ZeroDivisionError):
		assert 10 / 0
```