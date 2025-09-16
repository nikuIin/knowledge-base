[[Frontend]]
[[Кастомные ошибки, custom error (Frontend, React, TypeScript (TS)), exceptions]]

Когда ты видишь в классе ошибки строку вроде

```ts
Object.setPrototypeOf(this, NotTelegramWebAppError.prototype);
```

то это **работа с прототипом объекта**, которая нужна в JavaScript/TypeScript для корректной работы наследования, особенно при наследовании от встроенного класса `Error`.

***

### Зачем это делать?

В JavaScript (и TypeScript), при наследовании классов от `Error` возникает особенность:

- В конструкторе базового класса `Error` создаётся новый объект с прототипом `Error.prototype`.
- Из-за этого **прототипная цепочка твоего кастомного класса может быть "сломана"**:
  - Объект ошибки становится экземпляром `Error`, но **не экземпляром твоего класса ошибки**.
  - Например, при проверке `instanceof` результата работы конструктора твоя ошибка не будет распознаваться правильно.

***
### Что делает `Object.setPrototypeOf(this, NotTelegramWebAppError.prototype)`?

- Эта строка **ручная корректировка прототипа (prototype) текущего объекта `this`**.
- Устанавливает прототип объекта (`[[Prototype]]`) именно на твой класс ошибки, например на `NotTelegramWebAppError.prototype`.
- Благодаря этому объект ошибки становится правильным экземпляром твоего класса, и `instanceof` начинает работать корректно.

***
### Пример проблемы без `setPrototypeOf`

```ts
class MyError extends Error {
  constructor(message: string) {
    super(message);
  }
}

const err = new MyError('Ошибка');

// На старых версиях JS/TS или при неправильной компиляции:
console.log(err instanceof MyError); // может быть false!
console.log(err instanceof Error);   // true
```

***

### Пример с `setPrototypeOf`

```ts
class MyError extends Error {
  constructor(message: string) {
    super(message);
    Object.setPrototypeOf(this, MyError.prototype);
  }
}

const err = new MyError('Ошибка');
console.log(err instanceof MyError); // true
```

***

### Дополнительно

- В современных версиях TypeScript с ES2015+ эту проблему частично решает использование `new.target.prototype` вместо жёсткой ссылки на класс.
- Однако **кросс-платформенно и при старых сборках** вызов `Object.setPrototypeOf` остаётся распространённым паттерном.

***

# Код

```ts
export class UserNotFoundError extends Error {
  public status: number | null = null;

  constructor(message: string, status: number | null = null) {
    super(message);

    this.status = status;
    this.name = "UserNotFoundError";
    Object.setPrototypeOf(this, UserNotFoundError.prototype);
  }
}

class MyError extends Error {
  some_field: string = "1234";
}

class MyErrorWithoutFix extends MyError {
  constructor(message: string) {
    super(message);
    // Здесь НЕ вызываем Object.setPrototypeOf
  }
}

class MyErrorWithFix extends MyError {
  constructor(message: string) {
    super(message);
    Object.setPrototypeOf(this, MyError.prototype);
  }
}

function testErrors() {
  const err1 = new MyErrorWithoutFix("Ошибка без fix");
  const err2 = new MyErrorWithFix("Ошибка с fix");

  console.log("err1 instanceof MyError:", err1 instanceof MyError); // Часто false
  console.log("err1 instanceof Error:", err1 instanceof Error); // true

  console.log("err2 instanceof MyError:", err2 instanceof MyError); // true
  console.log("err2 instanceof Error:", err2 instanceof Error); // true
}

testErrors();

```

Вывод:
```bash
err1 instanceof MyError: false
err1 instanceof Error: true
err2 instanceof MyErrorWithFix: true
err2 instanceof Error: true
```

### Итог

`Object.setPrototypeOf(this, Класс.prototype)` — это хак для правильной установки цепочки прототипов, необходимый для корректной работы пользовательских классов наследников `Error`.  
Без этого могут возникать проблемы с `instanceof` и неадекватным поведением ошибки.


Sources
[1] javascript - Typescript - Extending Error class - Stack Overflow https://stackoverflow.com/questions/41102060/typescript-extending-error-class
[2] Object.setPrototypeOf() - JavaScript - MDN Web Docs - Mozilla https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/setPrototypeOf
[3] Errors in TypeScript - Alexander Karan's Blog https://blog.alexanderkaran.com/errors-in-typescript
[4] TypeError: can't set prototype of this object - MDN Web Docs - Mozilla https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Errors/Cant_set_prototype
[5] Documentation - TypeScript 2.2 https://www.typescriptlang.org/docs/handbook/release-notes/typescript-2-2.html
[6] Catch Error Type | TypeScript Guide by Convex https://www.convex.dev/typescript/optimization/typescript-catch-error-type
[7] extends Error - not working : r/typescript - Reddit https://www.reddit.com/r/typescript/comments/9u9107/extends_error_not_working/
[8] How to Fix instanceof Not Working For Custom Errors in TypeScript https://dev.to/dguo/how-to-fix-instanceof-not-working-for-custom-errors-in-typescript-4amp?comments_sort=latest
