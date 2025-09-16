[](Prototype%20error%20(typescript%20кастомные%20ошибки).md)[](Prototype%20error%20(typescript%20кастомные%20ошибки).md)[[Frontend]]
[[Frontend.canvas|Frontend canvas]] 

Пример правильного создания кастомной ошибки:

```ts
class NotTelegramWebAppError extends Error {
    constructor(message: string) {
        // вызываем конструктор супер-класса
        super(message); 
        // присваиваем ошибке новое имя
        this.name = 'NotTelegramWebAppError'; 
        // ручками меняем прототипы, чтобы при наследовании следующие
        // ошибки наследовались от этого класса, а не от Exception (из-за особенностей JS/TS)
        // Читать карточку Prototype
        Object.setPrototypeOf(this, NotTelegramWebAppError.prototype);
    }
}
```

[[Prototype error (typescript кастомные ошибки)|Карточка Prototype]]