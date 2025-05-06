
**Применение set()**

Допустим у нас есть словарь напитков и их ингридиентов. 
Мы хотим получить напиток, содержащий или апельсин, или вермут, тогда мы можем использовать пересечение (**&**)

```python
drinks = {
    'martini': {'vodka', 'vermouth'},
    'black russian': {'vodka', 'kahlua'},
    'white russina': {'cream', 'kahlua', 'vodka'},
    'manhattan': {'rye', 'vermouth', 'bitters'},
    'screwdrivar': {'orange juice', 'vodka'}
}

need = {'vermouth', 'orange juice'}
print(need)

for name, contents in drinks.items():
    if contents & {'vermouth', 'orange juice'}:
        print(name)

# martini
# manhattan
# screwdrivar
```

Если же мы хотим получить только те напитки, где содержится оба ингридиента, допустим вермут и водка, то делаем такое:
```python
drinks = {
    'martini': {'vodka', 'vermouth'},
    'black russian': {'vodka', 'kahlua'},
    'white russina': {'cream', 'kahlua', 'vodka'},
    'manhattan': {'rye', 'vermouth', 'bitters'},
    'screwdrivar': {'orange juice', 'vodka'}
}


for name, contents in drinks.items():
    if contents & {'vodka'} and contents & {'vermouth'}:
        print(name)
```

Не забываем об объединении **|**, разности **-** и симметричной разности **^**. 