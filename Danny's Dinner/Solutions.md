# Case Study 1: Danny's Diner

## Solution
### 1. What is the total amount each customer spent at the restaurant?
````sql
select customer_id,sum(price) from sales inner join menu on sales.product_id = menu.product_id
group by customer_id
````
#### Answer:
| Customer_id | Total_sales |
| ----------- | ----------- |
| A           | 76          |
| B           | 74          |
| C           | 36          |

- Customer A, B and C spent $76, $74 and $36 respectivly.
***

### 2. How many days has each customer visited the restaurant?

````sql
select customer_id , count(distinct(order_date)) as days_visited from sales
group by customer_id
````

#### Answer:
| Customer_id | Times_visited |
| ----------- | ----------- |
| A           | 4          |
| B           | 6          |
| C           | 2          |

- Customer A, B and C visited 4, 6 and 2 times respectivly.

***
### 3. What was the first item from the menu purchased by each customer?

````sql
select customer_id,product_name from (Select S.customer_id, 
       M.product_name, 
       S.order_date,
       DENSE_RANK() OVER (PARTITION BY S.Customer_ID Order by S.order_date) as rank
From Menu m
join Sales s
On m.product_id = s.product_id
group by S.customer_id, M.product_name,S.order_date)as new_table
where rank = 1

````

#### Answer:
| Customer_id | product_name | 
| ----------- | ----------- |
| A           | curry        | 
| A           | sushi        | 
| B           | curry        | 
| C           | ramen        |

- Customer A's first order is curry and sushi.
- Customer B's first order is curry.
- Customer C's first order is ramen.

***
### 4. What is the most purchased item on the menu and how many times was it purchased by all customers?

````sql
SELECT M.product_id, M.product_name,count(M.product_name) as count from Menu M inner join sales S on M.product_id = S.product_id
group by M.product_name
order by count desc limit 1
````

#### Answer:
| Product_name  | Times_Purchased | 
| ----------- | ----------- |
| ramen       | 8|


- Most purchased item on the menu is ramen which is 8 times.

***
### 5. Which item was the most popular for each customer?

````sql
select customer_id,product_name from ( SELECT customer_id,product_name,p,
RANK() over(partition by customer_id order by p desc) as rank from(Select customer_id, product_name,count(product_name) as p
From Menu m
join Sales s
On m.product_id = s.product_id
group by customer_id,product_name) as new_table)
where rank = 1

````

#### Answer:
| Customer_id | Product_name | Count |
| ----------- | ---------- |------------  |
| A           | ramen        |  3   |
| B           | sushi        |  2   |
| B           | curry        |  2   |
| B           | ramen        |  2   |
| C           | ramen        |  3   |

- Customer A and C's favourite item is ramen while customer B savours all items on the menu. 

***
### 6. Which item was purchased first by the customer after they became a member?

````sql
With Rank as
(
Select  S.customer_id,
        M.product_name,
	Dense_rank() OVER (Partition by S.Customer_id Order by S.Order_date) as Rank
From Sales S
Join Menu M
ON m.product_id = s.product_id
JOIN Members Mem
ON Mem.Customer_id = S.customer_id
Where S.order_date >= Mem.join_date  
)
Select *
From Rank
where rank = 1

````
#### Answer:
| customer_id |  product_name |order_date
| ----------- | ----------  |----------  |
| A           |  curry        |2021-01-07 |
| B           |  sushi        |2021-01-11 |

After becoming a member 
- Customer A's first order was curry.
- Customer B's first order was sushi.

***

### 7. Which item was purchased just before the customer became a member?

````sql
.With Rank as
(
Select  S.customer_id,
        M.product_name,
	Dense_rank() OVER (Partition by S.Customer_id Order by S.Order_date) as Rank
From Sales S
Join Menu M
ON m.product_id = s.product_id
JOIN Members Mem
ON Mem.Customer_id = S.customer_id
Where S.order_date < Mem.join_date  
)
Select *
From Rank
where Rank = 1
````

#### Answer:
| customer_id |product_name |order_date  |
| ----------- | ----------  |---------- |
| A           |  sushi      |2021-01-01 | 
| A           |  curry      |2021-01-01 | 
| B           |   sushi     |2021-01-04 |

Before becoming a member 
- Customer A’s last order was sushi and curry.
- Customer B’s last order wassushi.

***
### 8. What is the total items and amount spent for each member before they became a member?

````sql
.With Rank as
(
Select  S.customer_id,
        M.product_name,
        M.price,
	Dense_rank() OVER (Partition by S.Customer_id Order by S.Order_date) as Rank
From Sales S
Join Menu M
ON m.product_id = s.product_id
JOIN Members Mem
ON Mem.Customer_id = S.customer_id
Where S.order_date < Mem.join_date  
)
Select customer_id,count(customer_id) as total_amount, sum(price) as amount_spent
From Rank
group by customer_id
````
#### Answer:
| customer_id |Items | total_sales |
| ----------- | ---------- |----------  |
| A           | 2 |  25       |
| B           | 3 |  40       |

Before becoming a member
- Customer A spent $25 on 2 items.
- Customer B spent $40 on 3 items.

***
### 9. If each $1 spent equates to 10 points and sushi has a 2x points multiplier — how many points would each customer have?

````sql
With Points as
(
Select *, Case When product_id = 1 THEN price*20
               Else price*10
			   End as Points
From Menu
)
Select S.customer_id, Sum(P.points) as Points
From Sales S
Join Points p
On p.product_id = S.product_id
Group by S.customer_id;
````


#### Answer:
| customer_id | Points | 
| ----------- | -------|
| A           | 860 |
| B           | 940 |
| C           | 360 |

- Total points for customer A, B and C are 860, 940 and 360 respectivly.

***

