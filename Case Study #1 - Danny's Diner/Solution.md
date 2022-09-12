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

### Question #5
Which item was the most popular for each customer?

### Solution:
```sql
with most_popular as (
  select customer_id, 
  product_id, 
  count(product_id) as order_count, 
  dense_rank() over(partition by customer_id order by count(customer_id) desc) as rank
  from dannys_diner.sales
  group by customer_id, product_id
)

select customer_id, order_count, product_name
from most_popular as mp
join dannys_diner.menu as m
	on mp.product_id = m.product_id
where rank=1
order by customer_id
```

|customer_id |	order_count |	product_name |
|-----------|---------------|---------------|
|A 	|3 	|ramen|
|B 	|2 	|sushi|
|B 	|2 	|curry|
|B 	|2 	|ramen|
|C 	|3 	|ramen|

Using Window function ``most_popular`` - counting ``product_id``s and name it as ``order_count`` and using ``dense_rank`` to rank ``order_count`` by descending order for each customer.<br>
Filtering results with ``rank=1`` which is the most popular item for each customer.

<hr>

### Question #6
Which item was purchased first by the customer after they became a member?

### Solution:
```sql
with first_order_as_member as(
select members.customer_id, join_date, product_name, order_date,
dense_rank() over(partition by sales.customer_id order by sales.order_date) as first_item
from dannys_diner.menu as menu
inner join (dannys_diner.members as members
      inner join dannys_diner.sales as sales
      on members.customer_id = sales.customer_id)
on menu.product_id = sales.product_id
where order_date >= join_date
)

select customer_id, product_name, order_date
from first_order_as_member
where first_item=1
```

|customer_id 	|product_name 	|order_date|
|-|-|-|
|A 	|curry 	|2021-01-07T00:00:00.000Z|
|B 	|sushi 	|2021-01-11T00:00:00.000Z|

- Using Window function ``first_order_as_member`` - 
	- partitioning ``customer_id`` by ascending ``order_date``.<br>
	- join members and sales tables on ``customer_id`` and then join the results with menu table on ``product_id``<br>
	- filtering ``order_date`` to be on or after ``join_date``.
- Filtering results with ``first_item=1`` to return the first item purchased as a member.

<hr>

### Question #7
Which item was purchased just before the customer became a member?

### Solution:
```sql
with before_becoming_member as(
select members.customer_id, join_date, product_name, order_date,
dense_rank() over(partition by sales.customer_id order by sales.order_date desc) as last_item
from dannys_diner.menu as menu
inner join (dannys_diner.members as members
      inner join dannys_diner.sales as sales
      on members.customer_id = sales.customer_id)
on menu.product_id = sales.product_id
where order_date < join_date
)

select customer_id, product_name, order_date
from before_becoming_member
where last_item=1
```
|customer_id 	|product_name 	|order_date|
|-|-|-|
|A	|sushi	|2021-01-01T00:00:00.000Z|
|A 	|curry 	|2021-01-01T00:00:00.000Z|
|B 	|sushi 	|2021-01-04T00:00:00.000Z|

- Using Window function ``before_becoming_member`` - 
	- partitioning ``customer_id`` by descending ``order_date``.<br>
	- join members and sales tables on ``customer_id`` and then join the results with menu table on ``product_id``<br>
	- filtering ``order_date`` to be before ``join_date``.
- Filtering results with ``flast_item=1`` to return the last item purchased before becoming a member.

<hr>

### Question #8
What is the total items and amount spent for each member before they became a member?

### Solution:
```sql
select sales.customer_id, count(sales.product_id) as product_count, sum(menu.price) as amount_spent
from dannys_diner.sales as sales
join dannys_diner.menu as menu
on sales.product_id = menu.product_id
where sales.order_date < (select join_date 
                          from dannys_diner.members as members 
                          where sales.customer_id = members.customer_id)
group by sales.customer_id
```

|customer_id 	|product_count 	|amount_spent|
|-|-|-|
|A 	|2 	|25|
|B 	|3 	|40|

Count ``product_id``s from sales and sum ``price``s from menu based on matching ``product_id``s from the join process. <br>
Filtering ``order_date`` to be before ``join_date``.


<hr>

### Question #9
If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?

### Solution:
```sql
select customer_id, sum(case
                        when menu.product_id = 1 then menu.price*20
                        else menu.price*10
                        end) as points_earned
from dannys_diner.menu as menu
join dannys_diner.sales as sales
on menu.product_id = sales.product_id
group by customer_id
order by customer_id
```

|customer_id 	|points_earned|
|-|-|
|A 	|860|
|B 	|940|
|C 	|360|

1$ = 10 points. <br>
Which means, When customer ordering anything from the menu - on each 1$ they spent, they get 10 points.<br>
BUT! if the customer ordered Sushi (product_id =1) they gets 2 times the points, means 20 points on each 1$ they spent.<br>
So for this we use **CASE** in the sum function to sum the points based on the requirements.
