
```sql

-- numbers
select * from generate_series(1, 100, 1) limit 5 -- begin, end, step
/*
1,
2,
3,
4,
5
*/

-- date with generating function alias
select
  "date"."month"
from generate_series(
  '01.01.2000',
  '01.01.2005',
  interval '1 month'
) as "date"("month") -- alias, now it's like table

```

[[postgreSQL]]