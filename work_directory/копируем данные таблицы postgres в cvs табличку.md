С помощью команды `\copy` можно сохранить результат запроса в файл, например, в CSV-файл, это comma-separated values, то есть значения, разделённые запятой:

копировать код

```sql
\copy (SELECT * FROM author) TO '~/authors.csv' WITH CSV HEADER
```