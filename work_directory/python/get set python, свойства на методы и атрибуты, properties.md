
Property — питонячий аналог get и set.

```python
class FormatMixin:
    def dump(self):
        print(vars(self))


class Dog(FormatMixin):
    def __init__(self, name):
        self.__name = name

    def voice(self) -> str:
        return "Гав"

	# тут мы определили как-бы переменную name 
    @property 
    def name(self):
        return self.__name

    # тут мы как бы определили, что будет происходить когда будут изменять переменуюю name
    @name.setter
    def name(self, input_name: str) -> None:
        if input_name.isdigit():
            print("Имя не должно содержать только цифры")
        else:
            self.__name = input_name

	# а тут мы определил перменную name_len и да, мы можем к ней обращаться как к переменной, учитывая что ее нету в объекте self (явно нету).
    @property
    def name_len(self) -> int:
        return len(self.__name)


dog = Dog("Чарли")
dog.name = "1"

print(dog.name_len)
print(dog.name)

```