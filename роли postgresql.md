
* Создаем нового суперпользователя (делаем от имени `postgres`)
```sql
alter user van with superuser;
```

* создаем новую роль
```sql
create user van_next;
```