Что делать при расхождении в explain analalyze? 

```sql

explain analyze select email from test where email::int between 1 and 100;

Gather  (cost=1000.00..17321.67 rows=5000 width=6) (actual time=0.405..83.865 rows=100 loops=1) 
Workers Planned: 2                                                          
Workers Launched: 2                                                         
   Parallel Seq Scan on test  (cost=0.00..15821.67 rows=2083 width=6) (actual time=47.251..74.392 rows=33 loops=3) 
   Filter: (((email)::integer >= 1) AND ((email)::integer <= 100))          
   Rows Removed by Filter: 333300                                           
   Planning Time: 0.133 ms                                                     Execution Time: 83.889 ms                                                  ----------------------------------------------------------------------+
```
---
кстати, у меня тут есть index на колонку email, но идет seqscan, это вот почему: [[index на выражения]]


[[explain analyze]]