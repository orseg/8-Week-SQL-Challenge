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

### Steps:
I've used subquery in the ```sum``` function. <br/>
I've selected the price column from the menu table where the ```product_id``` from the menu table is equal to the ```product_id``` from the sales table and set "total_amount_spent" as alias.<br/>
grouped by ```customer_id``` from sales tables.

