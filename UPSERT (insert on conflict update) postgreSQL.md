 [[postgreSQL]]

Когда у нас есть кейсы на добавление в таблицу и обновления, нам нужно подумать о том, нельзя ли эти запросы объединить в один `UPSERT`.

Он работает так:
* есть таблица исходная, к ней обращаемся по названию
* есть таблица на обновление (`excluded`)

Давайте напишем пример, при конфликте будем увеличивать количество продуктов в корзине на переданное число:

```sql
insert into cart_item (cart_id, product_id, quantity) 
values (:cart_item, :product_id, :quantity)
on conflict (cart_id, product_id) do update
set quantity = cart_item.quantity + excluded.quantity;
```
