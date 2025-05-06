[[postgreSQL]]
[[СУБД]]

`tablespace` — это пространство, где будут содержать страницы (записи) определенных таблиц или индексов. Это знание помогает оптимизировать хранения данных на нашем диске и сэкономить много деняк. 

Чтобы нам создать `tablespace`, нужно для начала создать под него директорию и выдать ее во владения пользователя `postgres` и группе  `postgres`, чтобы (кто?) `postgres`, удивительно, мог иметь доступ к нашей директории.

```bash
sudo mkdir -p /postgres_tablespaces/my_first_tablespace
sudo chown postgres:postgres /postgres_tablespaces/my_first_tablespace
```

Потом заходим в `postgres` с помощью коннектором `psql/pgcli` или `gui` от имени `postgres` и создаём `tablespace`. Только админы и сам `postgres` может создавать `tablespaces`.

```sql
create tablespace ts1 location '/postgres_tablespaces/my_first_tablespace'
```

Так, ну, а теперь располагаем нашу новую табличку на этот `tablespace`

```sql
create table log(
	log_id bigint generated always as identity primary key,
	message text not null
) tablespace ts1
```

Вуаля! Готово!