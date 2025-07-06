Мы можем сравнивать кортежи в `postgreSQL`

Что-то типо-такого:

```sql
with create_order as (
insert into order_item(order_id, product_id, quantity_of_items, price_per_item)
(
	select
		:order_id as order_id,
		product_id,
		quantity as quantity_of_items,
		product_price as price_per_item
	from cart_item
	join product using(product_id)
	where cart_id = :user_id
)
on conflict(order_id, product_id)
do update set quantity_of_items = excluded.quantity_of_items + order_item.quantity_of_items
returning product_id, cart_id
)
delete from cart_item where (product_id, cart_id) in (select product_id, cart_id from create_order);
```

*delete from cart_item where* **(product_id, cart_id)** *in* *(select* **product_id, cart_id** *from create_order)*;