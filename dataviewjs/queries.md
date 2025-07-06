```js
// --- Стили для категорий запросов ---
const CATEGORY_BG_COLORS = {
    "оптимизация": "#fef5dd",
    "оконные": "#e7f3ff",
    // Добавьте сюда другие категории и их цвета
    "default": "#f5f5f5"
};

const CATEGORY_FONT_COLORS = {
    "default": "#333333"
};

const getCategoryBgColor = category => {
    return (category in CATEGORY_BG_COLORS) ? CATEGORY_BG_COLORS[category]: CATEGORY_BG_COLORS["default"];
}

const getCategoryFontColor = category => {
    return (category in CATEGORY_FONT_COLORS) ? CATEGORY_FONT_COLORS[category]: CATEGORY_FONT_COLORS["default"];
}

// --- Основные функции ---

// Получает все заметки с типом "Запрос PostgreSQL"

const getQueries = () => {
    // Ищем все страницы, которые ССЫЛАЮТСЯ на наш файл-тип
    const queries = dv.pages('[[types/Запрос PostgreSQL]]')
        .where(p => !["templates", "templater"].includes(p.file.folder)); // Исключаем папки с шаблонами
    return queries;
};

// Форматирует дату
const formatDate = d => {
    if (!d) return '';
    return d.toFormat("d MMMM yyyy", { locale: "ru" });
}

// Рендерит плашку с категорией
const renderCategory = query => {
    const category = query["Категория запроса"];
	if (!category) return ''; // Если категории нет, ничего не возвращаем
    
	return `<span class="category" style="background-color: ${getCategoryBgColor(category)}; color: ${getCategoryFontColor(category)};">${category}</span>`;
}

// Рендерит одну карточку запроса
const renderQuery = (query, additionalQueryRenderFunction) => {
    // В отличие от книги, у нас нет обложки, но есть название и ссылка
    return `<div class="query-card">
        <a href="${query.file.path}" class="internal-link query-title" target="_blank" rel="noopener nofollow">${query.Название || query.file.name}</a>
        <div class="query-meta">
            ${additionalQueryRenderFunction(query)}
        </div>
        <div class="categories">${renderCategory(query)}</div>
    </div>`;
}

// Рендерит сетку из карточек запросов
const renderQueries = (queryFiles, additionalQueryRenderFunction) => {
    let queriesHtml = [];
    for (const query of queryFiles) {
        queriesHtml.push(renderQuery(query, additionalQueryRenderFunction))
    }
	return queriesHtml.join("");
}

// Экспортируем функции в глобальный объект window, чтобы их можно было вызвать из других заметок
window.queries = {getQueries, formatDate, renderQueries};
```