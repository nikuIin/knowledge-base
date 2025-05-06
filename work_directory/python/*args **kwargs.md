
```python
def print_args(*args):
    print(args)
    print(*args)


def print_kwargs(**kwargs):
    print(kwargs)
    print(**kwargs)


a = ((2, 7, 5, 7, (2, 6)),)

print_args(a)
# * преобразует a в (2, 7, 5, 7, (2, 6))
print_args(*a) 

some_dict = {"a": 1, "b": 2}
some_dict2 = {"c": 3, "a": 4}

# а тут объединение через поверхностное копирование 
print({**some_dict, **some_dict2})
```
