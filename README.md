--Danny-s-Diner -CASE STUDY 1--

--Overview:--

Danny opened Danny’s Diner, a restaurant in 2021, serving his favorite Japanese dishes—sushi, curry, and ramen. He has basic data from the first few months of operation but does not know how to derive actionable insight to drive business growth. 

The goal is to analyze this data to gain insights into customer behavior, popular menu items, and sales trends to help the diner grow and succeed.


--Source: https://8weeksqlchallenge.com/ 

--Tool Used: SSMS (Microsoft SQL Server)


--Schema SQL:--

CREATE SCHEMA dannys_diner;
SET search_path = dannys_diner;
CREATE TABLE sales (
  "customer_id" VARCHAR(1),
  "order_date" DATE,
  "product_id" INTEGER
);
INSERT INTO sales
  ("customer_id", "order_date", "product_id")
VALUES
  ('A', '2021-01-01', '1'),
  ('A', '2021-01-01', '2'),
  ('A', '2021-01-07', '2'),
  ('A', '2021-01-10', '3'),
  ('A', '2021-01-11', '3'),
  ('A', '2021-01-11', '3'),
  ('B', '2021-01-01', '2'),
  ('B', '2021-01-02', '2'),
  ('B', '2021-01-04', '1'),
  ('B', '2021-01-11', '1'),
  ('B', '2021-01-16', '3'),
  ('B', '2021-02-01', '3'),
  ('C', '2021-01-01', '3'),
  ('C', '2021-01-01', '3'),
  ('C', '2021-01-07', '3');

CREATE TABLE menu (
  "product_id" INTEGER,
  "product_name" VARCHAR(5),
  "price" INTEGER
);

INSERT INTO menu
  ("product_id", "product_name", "price")
VALUES
  ('1', 'sushi', '10'),
  ('2', 'curry', '15'),
  ('3', 'ramen', '12');
  

CREATE TABLE members (
  "customer_id" VARCHAR(1),
  "join_date" DATE
);

INSERT INTO members
  ("customer_id", "join_date")
VALUES
  ('A', '2021-01-07'),
  ('B', '2021-01-09');



--Case Study Questions:--

-- 1. What is the total amount each customer spent at the restaurant?

select
customer_id,
sum(price) as total_amount
from sales s
left join menu m
on s.product_id = m.product_id
group by customer_id
order by total_amount desc;


-- 2. How many days has each customer visited the restaurant?

select 
customer_id,
count(distinct order_date) as no_of_visits
from sales
group by customer_id
order by customer_id asc;

-- 3. What was the first item from the menu purchased by each customer?

select 
customer_id,
min(order_date),
product_name
from sales s
left join menu m
on s.product_id=m.product_id
group by customer_id, product_name;


-- 4. What is the most purchased item on the menu and how many times was it purchased by all customers?

select
product_name,
count(product_name) as no_of_purchases
from sales s
left join menu m
on s.product_id=m.product_id
group by product_name
order by no_of_purchases desc;


-- 5. Which item was the most popular for each customer?

select 
m.product_id,
product_name,
count(product_name) as most_popular_item
from menu m
left join sales s
on m.product_id=s.product_id
group by m.product_id, product_name
order by most_popular_item desc;


-- 6. Which item was purchased first by the customer after they became a member?

select
  s.customer_id,
  m.product_name,
  min(s.order_date) as items_purchased_after_membership,
  b.join_date
from sales s
join menu m on s.product_id = m.product_id
join members b on s.customer_id = b.customer_id
where s.order_date >= b.join_date
group by s.customer_id, m.product_name, b.join_date;

--7.'Which item was purchased just before the customer became a member?

select
  s.customer_id,
  m.product_name,
  max(s.order_date) as items_purchased_before_membership,
  b.join_date
from sales s
join menu m on s.product_id = m.product_id
join members b on s.customer_id = b.customer_id
where s.order_date < b.join_date
group by s.customer_id, m.product_name, b.join_date;

-- 8. What is the total items and amount spent for each member before they became a member?

select
  s.customer_id,
  m.product_name,
  count(m.product_name) as total_items,
  sum(price) as amount_spent,
  max(s.order_date) as items_purchased_before_membership,
  b.join_date
from sales s
join menu m on s.product_id = m.product_id
join members b on s.customer_id = b.customer_id
where s.order_date < b.join_date
group by s.customer_id, m.product_name, b.join_date;

-- 9.  If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?

select 
s.customer_id,
m.product_name,
m.price,
sum(price) as total_amount,
sum(
    case
      when m.product_name='sushi' then m.price *20
  else m.price *10
  end
  )
  as total_points
  from sales s
  join menu m on s.product_id=m.product_id
  group by s.customer_id, m.product_name, m.price;

-- 10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?

with points as (
  select
  s.customer_id,s.order_date,m.price,m.product_name,b.join_date,
  case
  when s.order_date between b.join_date
  and (b.join_date + interval '6 days') then m.price * 20
  when m.product_name ='sushi' then m.price *20
  else m.price *10
  end as total_points
  from sales s
  join menu m on s.product_id=m.product_id
  join members b on s.customer_id=b.customer_id
  where s.order_date <= '2025-01-31'
  )
  select customer_id, sum (total_points) as totalpoints
  from points
  where customer_id in ('A','B')
  group by customer_id;

