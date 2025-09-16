[[Feature-Sliced Design — архитектура frontend приложения]]
[[Frontend.canvas|Frontend canvas]]

Из-за того, что в `FSD` достаточно много папок и может быть даже немного большая вложенность, импорты и работа с ними становятся достаточно сложной задачей. Поэтому одним из решений является обертывание импортов в `alias` в настройках проекта.  Вторым решением является прописывание в каждом слое свое `API`, в котором будут экспортироваться данные, которые необходимо экспортировать в этом слое. 


1. Первый шаг — настройка `alias`-ов в конфиге проекта:
	* `tsconfig.json`
	* `tsconfig.app.json`
	* `tsconfig.node.json`

В каждом из файлов нужно прописать:

```json
{
  "files": [],
  "references": [
    { "path": "./tsconfig.app.json" },
    { "path": "./tsconfig.node.json" }
  ],
  // ==============================
  // Это 
  "compilerOptions": {
    "baseUrl": "src",
    "paths": {
      "@app/*": ["app/*"],
      "@shared/*": ["shared/*"],
      "@entities/*": ["entities/*"]
    }
  }
  // ^^^^^^^^^^^^^^^^^^^^^^^^^^^^
}
```

Что тут происходит? Тут мы первым делом объявляем в `baseUrl` папочку `src`, то есть корневую папочку нашего проекта, и потом мы прописываем пути в ключе `paths`. И ключ `app`, например, мы монтируем к папочке `app`, находящейся в папочке `src`. И также мы делаем для `share`, `entities` и всех остальных папок.

Таким образом мы можем в нашем приложении прописывать не относительные пути, а использовать эти алиасы, что, во-первых, повышает читаемость, во-вторых, позволяет легче поддерживать приложение.

```tsx
import { createStore } from "effector/effector.umd";
import type { User } from "@entities/user";
import { clearUser, setUser } from "@entities/user";

export const $currentUser = createStore<User | null>(null)
  .on(setUser, (_, user) => user)
  .on(clearUser, () => null);

export const $isAuth = $currentUser.map(Boolean);
```

2. Второй шаг настроить `vite` (`vite.config.ts`):
```tsx
import { defineConfig } from "vite";
import react from "@vitejs/plugin-react";
import path from "path";

export default defineConfig({
  plugins: [react()],
  resolve: {
    alias: {
      "@shared": path.resolve(__dirname, "src/shared"),
      "@app": path.resolve(__dirname, "src/app"),
      "@entities": path.resolve(__dirname, "src/entities"),
    },
  },
});
```


**Но это еще не все!**

Чтобы такие импорты работали, нужно прописать то, что будет доступно для импорта для `entity` «пользователь», то есть нам нужно описать внутреннее `API`, делается это в файле `index.ts` (**не tsx**)

```ts
export { type User } from "@entities/user/model/types";
export { setUser, clearUser } from "@entities/user/model/events";
export { UserNotFoundError } from "@entities/user/model/errors";
```

И вот дерево `entity` «пользователь»:

```bash
.
├── model
│   ├── errors.tsx
│   ├── events.tsx
│   ├── store.tsx
│   └── types.tsx
└── index.ts
```

Теперь мы можем не идти в файл `@entity/user/model/events`, чтобы импортировать ивенты пользователя, мы можем просто обратить по пути `@entity/user`. Благодаря этому муву мы можем легко поддерживать наш код и не бояться, что напутаем где-то с импортами.

---
Делается же файл `index.ts` только в слайсе.
