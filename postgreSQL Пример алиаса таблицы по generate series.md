[[PostgreSQL DB.canvas|PostgreSQL DB]]

```sql
insert into test_table(value, name) (
	select
		nextval('test_seq'),
		'name ' || c1::varchar
	from generate_series(1000, 100000, 2) as gs(c1)
);
```
