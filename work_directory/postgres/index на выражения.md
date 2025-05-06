У нас есть табличка `test` на 1 млн записей. Там есть колонка `email`. 
Мы создаем индекс на колонку `email`:

```sql
create index concurrently on test(email);
```

Делаем следующий запрос:

```sql
explain analyze select test_id, email from test where lower(email)='some@example.com'

-- ... parallels seq scan
```

Видим `parallels seq scan` и сидим в недоумении... Почему? Так потому что `b3 index` у нас выстроен по `email`, а не по `lower(email)`, а это совершенно разные сбалансированные деревья. => создаем новый индекс на `lower(email)`

```sql
create index concurrenlty on test(lower(email))

explain analyze select test_id, email from test where email='some@example.com'
-- index only scan
```

Вуаля!

[[index]]
