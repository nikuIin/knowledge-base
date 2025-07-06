---
Тип: "[[Запрос PostgreSQL]]"
Название: postgreSQL Смотрим какие индексы используется в кластере. Запрос на статистику по использованию индексов
Категория запроса: оптимизация
Добавлен: 2025-06-09
---
[[postgres/postgreSQL|postgreSQL]]
[[PostgreSQL DB.canvas|PostgreSQL DB]]
## Когда использовать?

Используем, чтобы проанализировать, какие запросы выполняеются в кластере. Можем оценить необходимость в индексах и таким образом оптимизировать место на дисках.

## Описание (о чем подумать)

**Важно**: индексы на уникальность могут не использоваться для выборок, но удалять их нельзя, а то снесете проверку на уникальность

## Код запроса
### вывод всей статистике по индексам таблицы

```sql
select
  schemaname,
  relname,
  indexrelname,
  idx_scan
from pg_stat_user_indexes
where relname='t_name'
```
### вывод индексов с 0 использований

```sql
select
  schemaname,
  relname,
  indexrelname,
  idx_scan
from pg_stat_user_indexes
where idx_scan=0 and relname='t_name'
```
