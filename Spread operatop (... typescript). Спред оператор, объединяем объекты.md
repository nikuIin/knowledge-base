[[TypeScript]]
[[Frontend.canvas|Frontend canvas]]

 Была цель объединить **заготовленные заголовки под запрос** с заголовками пользователя, а также, например, с `body`, который придет от пользователя. Для решения этой задачи идеально подходит `spread-оператор` в *TypeScript* и *JavaScript*, потому что:
- `spread-оператор` может распаковать **объект** 
- благодаря этому можно провести объединение нескольких объектов.

Интересный момент заключается в следующем:

- Если в последнем объекте имеется параметр с названием, например, `a`, и в предыдущем объекте тоже есть параметр `a`, то значение параметра `a` из последнего объекта **заменит** значение из первого.

Таким образом, с помощью `spread-оператора` можно легко объединять объекты с возможностью перезаписи свойств.

```ts
const API_URL = import.meta.env.VITE_API_URL || "";

export function apiFetch(path: string, options: RequestInit = {}) {
  // Generate headers
  const defaultHeaders: HeadersInit = {
    "Content-Type": "application/json",
    Accept: "application/json",
    "Accept-Language": navigator.language || "en-US", // TODO: add constant, that contains the default app language
    "X-Forwarded-For": "AUTO_DETECT_IP",
    "User-Agent": navigator.userAgent,
  };


  // Объединяем объект «дефолтные заголовки» с заголовками пользователя. 
  const headers = {
    ...defaultHeaders,
    ...options.headers,
  };

  // Format body
  let body = options.body;
  if (body && !(body instanceof FormData)) {
    try {
      body = JSON.stringify(body);
    } catch (error) {
      console.error("Error serialized body to JSON", error);
    }
  }

  // Union all
  const fetchOptions: RequestInit = {
	// Именно распаковываем объект options, получаем оттуда и заголовки и body и объединияем с пользовательскими данными.
	// Важно понимать, что пользовательские данные приоритетнее, поэтому они заменят при конфликте данные из options.
    ...options,
    headers,
    body,
  };

  return fetch(`${API_URL}${path}`, fetchOptions);
}
```