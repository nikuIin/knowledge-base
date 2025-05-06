Примеси (mixins) в Python - это мощный инструмент для переиспользования кода и добавления функциональности классам, не прибегая к множественному наследованию в его классическом понимании (со всеми его сложностями). Давайте рассмотрим несколько практических примеров, которые покажут, как их использовать.

**Что такое примесь?**

По сути, примесь - это класс, который предоставляет определенный набор методов, предназначенных для добавления функциональности к другим классам. Он не предназначен для самостоятельного использования, а только для "подмешивания" в другие классы через наследование.

**Пример : Сериализация в JSON**

Предположим, у нас есть несколько классов, представляющих разные объекты (например, User, Product, Order), и мы хотим уметь легко преобразовывать их в JSON-формат. Вместо того, чтобы писать to_json метод для каждого класса, мы можем создать примесь:

```python
import json

class JSONMixin:
    def to_json(self):
        return json.dumps(self.__dict__)

class User(JSONMixin):
    def __init__(self, name, age, email):
        self.name = name
        self.age = age
        self.email = email

class Product(JSONMixin):
    def __init__(self, name, price, description):
        self.name = name
        self.price = price
        self.description = description

user = User("Alice", 30, "alice@example.com")
product = Product("Laptop", 1200, "High-performance laptop")

print(user.to_json())
print(product.to_json())
```