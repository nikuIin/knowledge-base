[[postgres/postgreSQL|postgreSQL]]
[[PostgreSQL DB.canvas|PostgreSQL DB]] — canvas
[Документация по JSON в PostgreSQL](https://postgrespro.ru/docs/postgresql/16/datatype-json)

Проверить, что объект есть в `JSON`:

```sql
select '{"a": {}}'::JSONB ? 'a'; -- true
```

Получить `JSONB` тип по имени объекта из `JSON`:

```sql
select '{"a": "123"}'::JSONB -> 'a'; --"123", type: jsonb
```

Получить `TEXT` тип по имени объекта из `JSON`:

```sql
select '{"a": "123"}'::JSONB ->> 'a'; --123, type: TEXT
```

