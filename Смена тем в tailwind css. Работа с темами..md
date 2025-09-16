[[Frontend.canvas|Frontend canvas]]
[[tailwind]]

Для того чтобы работать с темами в `Tailwind CSS`, необходимо выполнить следующие шаги:

1. В файле CSS проекта (у меня это `index.css`) нужно прописать:
```css
@custom-variant dark (&:where([data-theme=dark], [data-theme=dark] *));
```

2. На страницах, где используются стили `Tailwind CSS`, применять стили через префикс `dark`. 
   - Префикс `dark` показывает, что стили будут применяться в случае активации тёмной темы.
   - Тёмная тема определяется на верхнем уровне HTML-элемента. В `index.css` мы указываем, на что ориентироваться для переключения темы.
   - Если тема — `dark`, то стили с префиксом `dark:` будут **заменять базовые стили**.

```html
<div class="bg-white dark:bg-gray-800 p-6 rounded-lg shadow-lg">
  <h2 class="text-gray-900 dark:text-white text-xl font-bold mb-2">Заголовок карточки</h2>
  <p class="text-gray-700 dark:text-gray-300">Это пример текста внутри карточки с поддержкой тёмной темы.</p>
</div>
```

 То есть, если у нас в `HTML` будет `datasource` равен **dark**, то `Tailwind` будет понимать, что нужно использовать стили с пометкой **dark**.

```html
<html lang="en" data-theme="dark">
</html>
```
