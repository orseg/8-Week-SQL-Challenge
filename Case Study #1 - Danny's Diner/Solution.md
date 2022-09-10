<p align="center">
  <img src="https://8weeksqlchallenge.com/images/case-study-designs/1.png" height="360" width="360"/>
</p>

### Question #1 
What is the total amount each customer spent at the restaurant?

### Solution:
```sql
select customer_id, 
sum(
  (select M.price 
   from dannys_diner.menu M 
   where M.product_id = S.product_id)
) as total_amount_spent
from dannys_diner.sales S
group by S.customer_id
```
| customer_id | total_amount_spent |
| ----------- | ------------------ |
| A | 76 |
| B | 74 |
| C | 36 |

I've used subquery in the ```sum``` function. <br/>
I've selected the price column from the menu table where the ```product_id``` from the menu table is equal to the ```product_id``` from the sales table and set "total_amount_spent" as alias.<br/>
grouped by ```customer_id``` from sales tables.

<hr>

### Question #2
How many days has each customer visited the restaurant?

### Solution:
```sql
select customer_id, count(distinct order_date) as visit_count
from dannys_diner.sales
group by customer_id
```

| customer_id | visit_count |
| ----------- | ------------------ |
| A | 4 |
| B | 6 |
| C | 2 |

Used ```distincted``` in count function to count unique values. <br>
Otherwise, 2 records with the same day will be counted as 2 days which is not what we want.

<hr>

### Question #3
What was the first item from the menu purchased by each customer?

### Solution 1 (The hard way):
```sql
with first_item_purchased as (
  select *, row_number() over(partition by customer_id order by order_date) as row_number
  from dannys_diner.sales
)

select customer_id,
(select product_name 
 from dannys_diner.menu 
 where first_item_purchased.product_id = dannys_diner.menu.product_id) as product_name
from first_item_purchased
where row_number=1
group by customer_id, product_name
```

### Solution 2 (Using join):
```sql
with first_item_purchased as (
  select *, row_number() over(partition by customer_id order by order_date) as row_number
  from dannys_diner.sales
)

select customer_id,product_name
from first_item_purchased as fip
join dannys_diner.menu as m
	on fip.product_id = m.product_id
where row_number=1
group by customer_id, product_name
order by customer_id
```

Both solutions give the same result.

| customer_id | product_name |
| ----------- | ------------------ |
| A | sushi |
| B | curry |
| C | ramen |

Used Window function with row_number() partition by ``customer_id`` to create new column named **``row_number``** based on ``order_date``. <br>
**``product_name``** is taken from the ``menu`` table where the **``product_id``** from the ``sales`` table matching the **``product_id``** from the ``menu`` table.<br>
Because I've asked to return the first item that was pruchased from the menu by each customer, only results with **``row_number=1``** will be returned.

<hr> 

### Question #4
What is the most purchased item on the menu and how many times was it purchased by all customers?

### Solution:
```sql
select top(1) product_name, count(s.product_id) as total_purchases
from dannys_diner.sales as s
join dannys_diner.menu as m
	on s.product_id = m.product_id
group by s.product_id, product_name
order by total_purchases desc
```

| product_name | total_purchases |
| ----------- | ------------------ |
| ramen | 8 |

Counting the number of ``product_id`` with ``count()``.<br>
order by ``total_purchases`` by descending order (from the most purchased to the least purchased).<br>
Selecting the first row with ``top(1)`` which is the most purchased item.

<hr>
