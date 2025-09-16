https://habr.com/ru/companies/ruvds/articles/426413/

1. <b>Принцип единой ответственности.</b> Классы, методы и функции должны выполнять только 1 задачу. Если допустим у нас есть класс Animal, то он должен содержать поля и методы, решающее задачу с Animal. Такие как конструкторы, характеристики животного, его "голос". ==Но например функцию добавления в БД в этом классе быть не должно, получается, что класс описывает животного и отвечает за добавление животного в БД.== Поэтому нужно создать новый класс AnimalBD, который будет в конструктуре принимать животное и иметь функции save_to_db() и delete_from_db().
2. <b>Принцип открытости-закрытости.</b> ==Программы и сущности должны быть расширяемыми, но закрыты для модификаций.== Наш класс или объект должен реализовывать какую-либо функцию, не должно быть такого, чтобы функция возникала из анализа этого класса. Например животное издает у нас звуки. Мы можем написать это неправильно: мы проходимся по всему массиву животных, если i == lion => "рр" если i == "собака" => "гав".  Этот пример никуда не годится, потому что чтобы добавить нам животных или функционал, нам нужно модифицировать программу и добавлять новые значения. 
```
//...
const animals: Array<Animal> = [
    new Animal('lion'),
    new Animal('mouse')
];
function AnimalSound(a: Array<Animal>) {
    for(int i = 0; i <= a.length; i++) {
        if(a[i].name == 'lion')
            return 'roar';
        if(a[i].name == 'mouse')
            return 'squeak';
    }
}
AnimalSound(animals);
```
Вместо этого нам нужно определить метод get_voice() в классе Animal и в наследующих его классах Lion, Dog прописать отдельные функции. И помните про первый метод Solid, класс Animal определяет общие х-ки животного, в нем не нужно прописывать метод звука животного, путем поиска через анализ класса, он лишь носит судьбу супер класса. 
```
class Animal {
        makeSound();
        //...
}
class Lion extends Animal {
    makeSound() {
        return 'roar';
    }
}
class Squirrel extends Animal {
    makeSound() {
        return 'squeak';
    }
}
class Snake extends Animal {
    makeSound() {
        return 'hiss';
    }
}
//...
function AnimalSound(a: Array<Animal>) {
    for(int i = 0; i <= a.length; i++) {
        a[i].makeSound();
    }
}
```
Этот уже год более правильный и подходит под принцип открытости-закрытости. 
==Еще один пример: мы решили выдать клиентам скидку в 20%==
```
class Discount {
    giveDiscount() {
        return this.price * 0.2
    }
}
```
Теперь мы хотим давать скидку VIP и Super-Vip в 2 и 4 раза больше соответсвенно. 
Значит мы должны написать классы наследники от Discount. 
  
```
class VIPDiscount: Discount {    
	getDiscount() {        
		return super.getDiscount() * 2;    
	} 
}
```
```
class SuperVIPDiscount: VIPDiscount {
    getDiscount() {
        return super.getDiscount() * 2;
    }
}
```

==Тут используется __РАСШИРЕНИЕ__ возможностей класса, а не его модификация==

3. <b>Принцип подстановки Барбары Лисков (LSP).</b> Должен существовать абстрактный класс, а его наследники должны заменять друг-друга без ошибок в программе. То есть абстрактный метод должен быть реализован во всех классах наследниках. 
С нарушением LSP:
```cs
class Shape {  
    public double area() {  
        return 0;  
    }  
}  
  
class Circle extends Shape {  
    private double radius;  
    @Override  
    public double area() {  
        return Math._PI_ * radius * radius;  
    }  
}  
  
class Square extends Shape {  
    private double side;  
    @Override  
    public double area() {  
        return side * side;  
    }  
}
```
Правильно:
```cs
abstract class Shape {  
    public abstract double area();  
}  
  
class Circle extends Shape {  
    private double radius;  
    @Override  
    public double area() {  
        return Math._PI_ * radius * radius;  
    }  
}  
  
class Square extends Shape {  
    private double side;  
         @Override  
    public double area() {  
        return side * side;  
    }  
}
```

4. <b>Принцип разделение интерфейса (ISP).</b> 

Нужно создавать небольшие интерфейсы, реализующие конкретную проблему и не касающиеся другой. ==__Пользователи не должны зависеть от интерфейсов, которые они не используют.__==
Предположим, у нас есть система управления документами с интерфейсом **Document,** содержащим методы для работы с документами, такие как **open()**, **save()** и **close().** Однако одним клиентам требуется только возможность открывать и закрывать документы, а другим — только сохранять и закрывать их.
Неправильно:
![[Pasted image 20240626073024.png]]
Правильно:
![[Pasted image 20240626073017.png]]

Нарушение ISP:
```java
abstract class Shape {  
    public abstract double area();  
}  
  
class Circle extends Shape {  
    private double radius;  
    @Override  
    public double area() {  
        return Math._PI_ * radius * radius;  
    }  
}  
  
class Square extends Shape {  
    private double side;  
         @Override  
    public double area() {  
        return side * side;  
    }  
}
```
Правильно:
```java
interface Openable {  
    void open();  
}  
  
interface Savable {  
    void save();  
}  
  
interface Closable {  
    void close();  
}  
  
class TextEditorOne implements Openable, Closable {  
    public void open() {  
        _// Открыть документ_    }  
     
    public void close() {  
        _// Закрыть документ_    }  
}  
class TextEditorTwo implements Savable, Closable {  
     
    public void save() {  
        _// Сохранить документ_    }  
  
    public void close() {  
        _// Закрыть документ_    }  
}
```

5. <b>DIP</b> заключает в себе идею, что низкоуровневые классы не должны зависеть от верхних, оба из них должны зависеть от абстрактных методов. Абстракции должны зависеть от деталей, но детали должны зависеть от абстракций.
То есть условно класс не должен зависеть от класса, выше уровня ему. Он должен зависеть от интерфейса, чтобы этот класс было возможно заменить.
Пример:
```cs
class OrderProcessor {
    private FileLogger fileLogger; // Зависимость от конкретного класса

    public OrderProcessor() {
        fileLogger = new FileLogger();
    }

    public void processOrder(Order order) {
        // Обработка заказа
        fileLogger.log(order); // Запись в файл
    }
}

class FileLogger {
    public void log(Order order) {
        // Запись информации о заказе в файл
    }
}
```
Тут класс OrderProcessor зависит от FileLogger. И если мы хотим использовать другой метод записи, нам нужно менять сам класс, что противоречит методом SOLID
```cs
interface ILogger {
    void log(Order order);
}

class FileLogger implements ILogger {
    @Override
    public void log(Order order) {
        // Запись в файл
    }
}

class OrderProcessor {
    private ILogger logger; // Зависимость от интерфейса

    public OrderProcessor(ILogger logger) {
        this.logger = logger;
    }

    public void processOrder(Order order) {
        // Обработка заказа
        logger.log(order); // Запись через интерфейс
    }
}
```
Теперь у нас OrderProcessor не зависит напрямую от FileLogger, но зависит от абстрактного метода. В этом и суть последнего принципа SOLID, принципа D

[[Проектирование - главный этап разработки ПО]]