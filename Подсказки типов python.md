[[python]]

```python
some_int: int = 5

some_array: list[str] = ['some', 'next']
some_dict: dict[str, int] = {'nik': 5}

def some_func() -> bool:
	return True
```
### Any и Union

**Any** — любое значение
```python
some: Any
```
**Union** — одно из
```python
some: Union[str, int]

# с python 3.10
some: str | int
```

---
Нужно понимать, что *интерпретатор* игнорирует все подсказки.