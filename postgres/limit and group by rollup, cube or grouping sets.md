Столкнулся с проблемой, вполне логичной, что при `limit n` строка от `group by rollup` или `group by grouping sets` с информацией о агрегации по всем строк пропадает, как я уже сказал, логично, так как это обычная лишь строка и `limit` по нашей просьбе ее отсекает. 

Решение в `order by some_column nulls first`. :)

```sql
select *, count(*)
from entity
group by rollup (name)
order by name nulls first
limit 2
```