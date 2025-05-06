[[Frontend]]

Пример правильного создания кастомной ошибки:

```ts
class NotTelegramWebAppError extends Error {
    constructor(message: string) {
        // вызываем конструктор супер-класса
        super(message); 
        // присваиваем ошибке новое имя
        this.name = 'NotTelegramWebAppError'; 
		// А вот об этом нужно еще почитать
        Object.setPrototypeOf(this, NotTelegramWebAppError.prototype);
    }
}
```

[[Prototype]]