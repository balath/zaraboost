## ZaraBoost SQL query test
Given the tables COUNTRIES, DELIVERY_TYPES, ITEM_REFERENCES, ORDERS y ORDER_ITEMS, and using my MySQL as a database manager.

Tables:
```sql
create table countries(
   id int not null,
   iso char(2) not null,
   name varchar(80) not null,
   nicename varchar(80) not null,
   iso3 char(3) default null,
   numcode smallint default null,
   phonecode int not null,
   primary key(id)
);

create table delivery_types(
   id int not null,
   name varchar(80) not null,
   primary key(id)
); 

create table item_references(
   id int,
   price numeric(9,2)
);

create table orders(
   id int,
   country_id int,
   delivery_type_id int
); 

create table order_items(
  id int,
  order_id int,
  item_reference_id int,
  units int
);
```

Calculate income from shipping costs with a single SQL sentence that returns a number in a column named delivery_incomes.

It should be noted that:

+ All orders from SPAIN ('iso = ES') do not pay shipping costs.
+ For the other countries:
    + If the order's amount is equal to or greater than €50 do not pay shipping costs.
    + If the order's amount is less than €50 it will pay €3.95 if the delivery type is 'ORDINARY DELIVERY' and €7.5 it is 'URGENT DELIVERY'
    + The amount will be approximated to two decimals.

The query should work with any dataset.

---

### My solution
```sql
with 
international_orders(id) as (
    select o.id
    from orders as o, countries as c
    where o.country_id = c.id and c.iso = 'ES'
),
international_orders_with_costs(id) as (
  select o.id
  from international_orders as o, (order_items natural join item_references)
  where o.id = order_id
  group by o.id
  having sum(price * units) < 50
),
final_orders_with_delivery_name(id, delivery_name) as (
    select o.id, d.name
    from delivery_types as d, (international_orders_with_costs natural join orders) as o
    where o.delivery_type_id = d.id
),
ordinary_deliveries(income) as (
    select count(*) * 3.95 
    from final_orders_with_delivery_name
    where delivery_name like 'ORDINARY DELIVERY'
    ), 
urgent_deliveries(income) as (
    select count(*) * 7.5
    from final_orders_with_delivery_name
    where delivery_name like 'URGENT DELIVERY'
    )
select ordinary_deliveries.income + urgent_deliveries.income
from ordinary_deliveries, urgent_deliveries
```
