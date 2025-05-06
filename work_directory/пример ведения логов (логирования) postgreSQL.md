[[postgreSQL]]
[[тригеры и правила. triggers rules]]

[[postgreSQL]]

* Видение логирования:
```sql
create table client(
  client_id bigint generated always as identity primary key,
  first_name varchar(255) not null check(length(first_name) > 0),
  last_name varchar(255) check(length(last_name) > 0),
  middle_name varchar(255) check(length(middle_name) > 0)
);

create table client_log (
  client_log_id bigint generated always as identity primary key,
  client_id bigint not null references client(client_id),
  action varchar(255) not null,
  old_first_name varchar(255),
  old_last_name varchar(255),
  old_middle_name varchar(255),
  new_first_name varchar(255),
  new_last_name varchar(255),
  new_middle_name varchar(255),
  change_time timestamp not null default current_timestamp
);

-- создаём функцию для тригера
create or replace function log_client()
returns trigger
language plpgsql
as $$
begin
  insert into client_log(
    client_id,     action,          old_first_name,
    old_last_name, old_middle_name, new_first_name,
    new_last_name, new_middle_name, change_time
  ) values (
    new.client_id, tg_op, old.first_name,
    old.last_name, old.middle_name,
    new.first_name, new.last_name, new.middle_name,
    current_timestamp
  );
  return new;
end;
$$;

-- создаём тригер на операция вставки и обновления
create or replace trigger client_log_trigger
after insert or update
on client
for each row
execute function log_client();

-- теперь наша табличка для логов будет записывать данные
--4, 10010, 'INSERT', null, null, null, 'Ваня', null, null, '2025-04-05 10:08:09.752746'

--5, 10010, 'UPDATE', 'Ваня', null, null, 'Ваня', 'Никулин', null, '2025-04-05 10:08:43.028630'
```
