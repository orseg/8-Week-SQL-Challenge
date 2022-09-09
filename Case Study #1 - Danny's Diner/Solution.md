Question #1 - 
What is the total amount each customer spent at the restaurant?

Solution:
`select customer_id, 
sum((select M.price 
     from dannys_diner.menu M 
     where M.product_id = S.product_id)) as total_amount_spent
from dannys_diner.sales S
group by S.customer_id`
