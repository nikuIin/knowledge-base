
Функция zip() позволяет упаковать несколько списков в один, состоящей из кортежей. Где в кортеже будут элементы, соответсвующие элементам по их индексам, лучше на примере:
```python
a = [1, 2, 3]
b = [3, 4, 5]
c = zip(a,b) # [(1, 3), (2, 4), (3, 5)] с типом ZIP !!!

print(list(c))
```

Если в каком либо из списков, переданных в `zip()`, будет меньше данных чем в другом, то они отбросятся. То есть `zip` проходится только по самой маленькой из последовательностей

```python
a = [1, 2, 3]
b = [3, 4]
c = zip(a,b) # [(1, 3), (2, 4)]
print(list(c)) 
```

## Сценарий выполнения

Самый лучший сценарий который сейчас мне приходит в голову это составления списка продуктов ко дню недели:
```python
week = ['Monday', 'Tuesday', 'Wednsday', 'Thirdsday', 'Friday', 'Satutrday', 'Sunday']
breakfast = ['Milk', 'Apple', 'Banana', 'Potato']
lunch = ['Soup', 'Fish', 'Meat', 'Carrot', 'Potato']

day_products = zip(week, breakfast, lunch)

for day, first_product, second_product in day_products:
    print(day, first_product, second_product)
```


`zip_longest()` из `itertools` позволяет провернуть такое: 
```python
from itertools import zip_longest

a = [1, 2, 3]
b = ["a", "b", "c", "d"]
c = list(zip_longest(a, b))
print(c)  # -> (1, 'a'), (2, 'b'), (3, 'c'), (None, 'd'))

```

