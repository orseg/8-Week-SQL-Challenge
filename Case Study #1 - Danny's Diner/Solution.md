# 🍜 Case Study #1: Danny's Diner

### Question #1 
What is the total amount each customer spent at the restaurant?

### Solution:
```sql
select customer_id, 
sum(price) as total_amount_spent
from sales
left join menu
on sales.product_id = menu.product_id
group by customer_id
```
| customer_id | total_amount_spent |
| ----------- | ------------------ |
| A | 76 |
| B | 74 |
| C | 36 |

<hr>

### Question #2
How many days has each customer visited the restaurant?

### Solution:
```sql
select customer_id, count(distinct order_date) as visit_count
from sales
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
  from sales
)

select customer_id,
(select product_name 
 from menu 
 where product_id = first_item_purchased.product_id) as product_name
from first_item_purchased
where row_number=1
group by customer_id, product_id
```

### Solution 2 (Using join):
```sql
with first_item_purchased as (
  select *, row_number() over(partition by customer_id order by order_date) as row_number
  from sales
)

select customer_id, product_name
from first_item_purchased as fip
join menu
	on fip.product_id = menu.product_id
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
select top(1) product_name, count(sales.product_id) as total_purchases
from sales
join menu
	on sales.product_id = menu.product_id
group by sales.product_id, product_name
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
  from sales
  group by customer_id, product_id
)

select customer_id, order_count, product_name
from most_popular as mp
join menu
	on mp.product_id = menu.product_id
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
from menu
inner join (members
      inner join sales
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
|A 	|curry 	|2021-01-07|
|B 	|sushi 	|2021-01-11|

- Using Window function ``first_order_as_member`` - 
	- partitioning ``customer_id`` by ascending ``order_date``.<br>
	- join meenu and sales tables on ``product_id`` and then join the results with members table on ``customer_id``<br>
	- filtering ``order_date`` to be on or after ``join_date``.
- Filtering results with ``first_item=1`` to return the first item purchased as a member.

<hr>

### Question #7
Which item was purchased just before the customer became a member?

### Solution:
```sql
with before_becoming_member as (
select members.customer_id, join_date, product_name, order_date,
dense_rank() over(partition by sales.customer_id order by sales.order_date desc) as last_item
from menu
inner join (members
      inner join sales
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
|A	|sushi	|2021-01-01|
|A 	|curry 	|2021-01-01|
|B 	|sushi 	|2021-01-04|

- Using Window function ``before_becoming_member`` - 
	- partitioning ``customer_id`` by descending ``order_date``.<br>
	- join members and sales tables on ``customer_id`` and then join the results with menu table on ``product_id``<br>
	- filtering ``order_date`` to be before ``join_date``.
- Filtering results with ``last_item=1`` to return the last item purchased before becoming a member.

<hr>

### Question #8
What is the total items and amount spent for each member before they became a member?

### Solution:
```sql
select sales.customer_id, count(sales.product_id) as product_count, sum(menu.price) as amount_spent
from sales
join menu
on sales.product_id = menu.product_id
where sales.order_date < (select join_date 
                          from members
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
from menu
join sales
on menu.product_id = sales.product_id
group by customer_id
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

<hr>

### Question #10
In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?

### Solution 1 (If we take in consideration that this is only applied from the day they became members):
```sql
with january_cte as (
  select *,
  dateadd(day, 6, join_date) as first_week --first week of the program
  from members
)

select jan.customer_id, 
sum(case
    when sales.order_date between jan.join_date and jan.first_week then menu.price*10*2
    when sales.order_date not between jan.join_date and jan.first_week and menu.product_id = 1 then menu.price*10*2
    else menu.price*10
    end) as points_earned
from january_cte as jan
join (sales
      join menu
      on sales.product_id = menu.product_id)
on sales.customer_id = jan.customer_id
where sales.order_date <= '2021-01-31' and sales.order_date >= jan.join_date
group by jan.customer_id
```
|customer_id 	|points_earned|
|-|-|
|A 	|1020|
|B 	|320|

### Solution 2 (If not):
```sql
with january_cte as (
  select *,
  dateadd(day, 6, join_date) as first_week --first week of the program
  from members
)

select jan.customer_id, 
sum(case
    when sales.order_date between jan.join_date and jan.first_week then menu.price*10*2
    when sales.order_date not between jan.join_date and jan.first_week and menu.product_id = 1 then menu.price*10*2
    else menu.price*10
    end) as points_earned
from january_cte as jan
join (sales
      join menu
      on sales.product_id = menu.product_id)
on sales.customer_id = jan.customer_id
where sales.order_date <= '2021-01-31'
group by jan.customer_id
```
|customer_id 	|points_earned|
|-|-|
|A 	|1370|
|B 	|820|

<hr>

### Bonus Question #1
Recreate the following table - customer_id, order_date, product_name, price, member (Y/N)

```sql
select sales.customer_id, sales.order_date,  menu.product_name, menu.price, 
case when sales.order_date >= members.join_date then 'Y' else 'N' end as member
from sales
left join menu
on sales.product_id = menu.product_id
left join members
on sales.customer_id = members.customer_id
order by customer_id, order_date
```

|customer_id 	|order_date 	|product_name 	|price 	|member|
|-|-|-|-|-|
|A 	|2021-01-01T00:00:00.000Z 	|sushi 	|10 	|N|
|A 	|2021-01-01T00:00:00.000Z 	|curry 	|15 	|N|
|A 	|2021-01-07T00:00:00.000Z 	|curry 	|15 	|Y|
|A 	|2021-01-10T00:00:00.000Z 	|ramen 	|12 	|Y|
|A 	|2021-01-11T00:00:00.000Z 	|ramen 	|12 	|Y|
|A 	|2021-01-11T00:00:00.000Z 	|ramen 	|12 	|Y|
|B 	|2021-01-01T00:00:00.000Z 	|curry 	|15 	|N|
|B 	|2021-01-02T00:00:00.000Z 	|curry 	|15 	|N|
|B 	|2021-01-04T00:00:00.000Z 	|sushi 	|10 	|N|
|B 	|2021-01-11T00:00:00.000Z 	|sushi 	|10 	|Y|
|B 	|2021-01-16T00:00:00.000Z 	|ramen 	|12 	|Y|
|B 	|2021-02-01T00:00:00.000Z 	|ramen 	|12 	|Y|
|C 	|2021-01-01T00:00:00.000Z 	|ramen 	|12 	|N|
|C 	|2021-01-01T00:00:00.000Z 	|ramen 	|12 	|N|
|C 	|2021-01-07T00:00:00.000Z 	|ramen 	|12 	|N|

### Bonus Question #2
Danny also requires further information about the ranking of customer products, but he purposely does not need the ranking for non-member purchases so he expects null ranking values for the records when customers are not yet part of the loyalty program.

```sql
with members_cte as(
select sales.customer_id, sales.order_date,  menu.product_name, menu.price, 
case when sales.order_date >= members.join_date then 'Y' else 'N' end as member
from sales
left join menu
on sales.product_id = menu.product_id
left join members
on sales.customer_id = members.customer_id
)

select *,
case when member='N' then NULL
else rank() over(partition by customer_id, member order by order_date) end as ranking
from members_cte
```

|customer_id 	|order_date 	|product_name 	|price 	|member| ranking|
|-|-|-|-|-|-|
|A 	|2021-01-01T00:00:00.000Z 	|sushi 	|10 	|N 	|null|
|A 	|2021-01-01T00:00:00.000Z 	|curry 	|15 	|N 	|null|
|A 	|2021-01-07T00:00:00.000Z 	|curry 	|15 	|Y 	|1|
|A 	|2021-01-10T00:00:00.000Z 	|ramen 	|12 	|Y 	|2|
|A 	|2021-01-11T00:00:00.000Z 	|ramen 	|12 	|Y 	|3|
|A 	|2021-01-11T00:00:00.000Z 	|ramen 	|12 	|Y 	|3|
|B 	|2021-01-01T00:00:00.000Z 	|curry 	|15 	|N 	|null|
|B 	|2021-01-02T00:00:00.000Z 	|curry 	|15 	|N 	|null|
|B 	|2021-01-04T00:00:00.000Z 	|sushi 	|10 	|N 	|null|
|B 	|2021-01-11T00:00:00.000Z 	|sushi 	|10 	|Y 	|1|
|B 	|2021-01-16T00:00:00.000Z 	|ramen 	|12 	|Y 	|2|
|B 	|2021-02-01T00:00:00.000Z 	|ramen 	|12 	|Y 	|3|
|C 	|2021-01-01T00:00:00.000Z 	|ramen 	|12 	|N 	|null|
|C 	|2021-01-01T00:00:00.000Z 	|ramen 	|12 	|N 	|null|
|C 	|2021-01-07T00:00:00.000Z 	|ramen 	|12 	|N 	|null|
