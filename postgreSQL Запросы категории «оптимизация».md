[[Запросы PostgreSQL]]
```dataviewjs
// Импортируем наш код (нужен только для getQueries)
eval(
    (await app.vault.read(app.vault.getAbstractFileByPath("dataviewjs/queries.md")))
    .replace("```js", "").replace("```", "")
);

// Получаем запросы и фильтруем их, как и раньше
const queryFiles = window.queries.getQueries()
    .where(p => p["Категория запроса"]?.toLowerCase().trim() == "оптимизация")
    .sort(p => p["Добавлен"], "desc");

// Создаем массив из ссылок на файлы для dv.list()
const fileLinks = queryFiles.map(p => p.file.link);

// Рендерим простой список
dv.list(fileLinks);
```




