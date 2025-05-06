[[Hooks хуки в frontend]]
[Хорошие видосы о хуках](https://www.youtube.com/watch?v=O6P86uwfdR0&list=PLZlA0Gpn_vH8EtggFGERCwMY5u5hOjf-h&index=1)

Приме `useState`:
```tsx
import React, {useState} from "react";

export const MyCoolComponents = (): void => {
	const [myCoolComponentsStatus, setStatus] = useState<'loading' | 'success'>(() => 'loading');
	setStatus('success');
};

```

Важно понимать, что `useState`-компоненты можно использовать **только внутри элемента/объекта**, в котором они были объявлены.

**!!!** Также важно знать, что `useState`, все `useState` должны прогружать вместе компонентом. Это означает, что мы не должны их добавлять в конструкции, например `if-else_if-else`. Еще раз: **все `useState` должны прогружаться вместе с компонетном.**

`useState` => *2 params*. Первый параметр — само значение, куда будут записываться данные. В нашем случае это `myCoolComponentStatus`. Второй параметр (`setStatus`) — это функция, которая будет влиять, а точнее устанавливать значение в `myCoolComponentsStatus`. Далее в `<>` перечислены типы, какие может принимать значение  `myCoolComponentsStatus`. Обычно там задается *тип/type*, например `String, number, etc.`, но в тех случаях, когда мы можем ограничить значения фактическими, мы должны это сделать, это будет хорошим тоном. 

Теперь мы можем вызывать `setStatus` где угодно в рамках нашего объекта. Мы можем, например, определить `<button/>` и в ее параметр `onClick` передать функцию: 
```ts 
setStatus("sucesss")
```


Далее рассмотрим следующие 2 примера:

```tsx
// Пример 1
import React, {useState} from "react";

function App(){
	const[counter, setCounter] = useState<number> = 0;

	function decrementCount(){
		setCounter(prevCounter => prevCounter - 1);
		setCounter(prevCounter => prevCounter - 1);
	}

	function incrementCount(){
		setCounter(prevCounter => prevCounter + 1);
		setCounter(prevCounter => prevCounter + 1);
	}

	return (
		<>
			<button onClick={decrementCount}>-</button>
			<span>{counter}</span>
			<button onClick={incrementCount}>+</button>
		</>
	)
} 

// Пример 2
import React, {useState} from "react";

function App(){
	const[counter, setCounter] = useState<number> = 0;

	function decrementCount(){
		setCounter(counter - 1);
		setCounter(counter - 1);
	}

	function incrementCount(){
		setCounter(counter + 1);
		setCounter(counter + 1);
	}

	return (
		<>
			<button onClick={decrementCount}>-</button>
			<span>{counter}</span>
			<button onClick={incrementCount}>+</button>
		</>
	)
} 
```

Что в них происходит и чем они отличаются? Отличаются они функциями изменения счетчика. В первом примере в `setCounter` передаются функции, принимающие значения от `setCounter`, то есть `counter` как `prevCounter` и заносят новое, как `prevCounter -/+ 1` два раза. Во втором же просто берется `counter` и `-/+ 1` два раза. Вроде бы, результат должен быть в итоге один и тот же, кнопки в обоих вариантах должны менять значения на `-2/+2` соответственно. Но это **не так**. Почему?

Потому что, разберем второй случай, `counter` будет возвращать всегда текущее значение в рендере `DOM`. А так как мы из рендера не выходим, мы получаем постоянно исходное значение, то есть `0`. В первом же случае мы передаем `0` в функцию, которая его изменяет, и поэтому там сработает изменение на `-2/+2` правильно. 

Какой вывод? 

*Вывод: если мы в `useState` оперируем прошлыми значениями, мы должны все оборачивать в фукнции.*

ТАКЖЕ ЕЩЕ ОДИН ВАЖНЫЙ ФАКТ
---

```tsx
import { useState } from 'react'
import './App.css'

function initialCounter() {
    console.log('counter 2 init')
    return 0
}

function App() {
    const [count, setCount] = useState(() => {
        console.log('counter 1 init')
        return 0
    })
    const [count2, setCount2] = useState(initialCounter())

    function increaseCounter() {
        setCount((prevCount) => prevCount + 1);
        setCount2((prevCount) => prevCount + 1);
    }


    return (
        <>
            <span>{count}</span>
            <span><br />{count2} <br /></span>
            <button onClick={increaseCounter}>+</button>
        </>
    )
}

export default App

```

Как мы видим,  у нас есть 2 `useState`. Для `counter` и `counter2`. Отличие в том, что `useState counter` принимает в дефолтные параметры тип `func`, а `counter2` `func()`. И это очень сильно влияет на их поведение и даже на производительность сайта. 

Из-за того, что мы передаем вызов функции в `counter2`, она будет срабатывать **всегда**, когда мы рендерим (изменяем) этот компонент. `counter1` вызовет же функцию всего лишь один раз, представляете какая экономия ресурсов?

*Вывод: в дефолтных значениях для `useState` используем функции типа `func`*