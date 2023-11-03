-- 1. What is the total amount each customer spent at the restaurant?

SELECT M.product_id,sum(M.price) as Total_amount,S.customer_id
FROM dannys_diner.menu as M
inner join dannys_diner.sales as S
on M.product_id=S.product_id
group by s.customer_id
order by sum(M.price)

select customer_id,sum(price) amount from sales s
join menu m on s.product_id = m.product_id
group by customer_id
order by customer_id

-- 2. How many days has each customer visited the restaurant?

select customer_id,distinct(count(order_date)) as "Total days customer visited" from dannys_diner.sales
group by customer_id 

---3. What was the first item from the menu purchased by each customer?

select S.customer_id,MIN(S.order_date) AS "First visited date",M.product_name AS "FIRST PURCHASED ITEAM" from dannys_diner.sales as S 
inner join 
dannys_diner.menu as M
on S.product_id=M.product_id
GROUP BY S.customer_id
ORDER BY S.customer_id

---4.What is the most purchased item on the menu and how many times was it purchased by all customers?

;with cte as
(select S.customer_id,S.product_id,count(*) as "count_of_iteam",M.product_name 
 from dannys_diner.sales as S
 inner join dannys_diner.menu as M
 on  S.product_id=M.product_id 
group by S.product_id)
 select M.product_name,count_of_iteam from cte
 order by count_of_iteam desc
 
 -- 5. Which item was the most popular for each customer?
 
  SELECT M.product_name as "favorite_dish",s.customer_id,count(*) as Purchase_count
  from dannys_diner.sales as S
  inner join 
  dannys_diner.menu as M on
  S.product_id=M.product_id
  group by S.product_id,s.customer_id
  having count(*) =select max(maxcount) from 
(select s.customer_id,count(*) as Purchase_count from dannys_diner.sales
group by customer_id,product_id ) as maxcount from dannys_diner.sales as S
where S.product_id=M.product_id) order by Purchase_count desc
 
 ---6 Which item was the most popular for each customer? 
;WITH CTE AS
(select S.customer_id,count(S.product_id) as Scount,dense_rank() over(partition by S.customer_id order by count(S.product_id) desc) as rnk from dannys_diner.sales S
join dannys_diner.Menu M
on S.product_id=M.product_id
GROUP BY S.product_id,S.customer_id,M.product_name )
SELECT Scount,S.customer_id,M.product_name FROM CTE
WHERE rnk=1

----7 Which item was purchased first by the customer after they became a member?
;WITH CTE AS 
(
select S.customer_id,S.order_date,S.product_id,ROW_NUMBER() OVER(PARTITION BY S.customer_id ORDER BY S.customer_id) AS CSID
from dannys_diner.sales as S
inner join 
dannys_diner.members as M
on S.customer_id=M.customer_id
where S.order_date>=M.join_date
ORDER BY order_date)
SELECT S.customer_id,S.order_date,S.product_id FROM CTE 
WHERE CSID=1

----8 Which item was purchased just before the customer became a member?
; WITH CTE AS 
(select S.customer_id,S.order_date,S.product_id,ROW_NUMBER() OVER(PARTITION BY S.customer_id ORDER BY order_date DESC) AS rcn
from dannys_diner.sales as S
inner join dannys_diner.members as M
on S.customer_id=M.customer_id
where S.order_date<M.JOIN_DATE
ORDER BY order_date DESC)
SELECT S.customer_id,S.order_date,S.product_id FROM CTE
WHERE rcn=1

---9 What is the total items and amount spent for each member before they became a member?
SELECT S.customer_id,count(S.Product_id) as quantity,sum(M.price) as Total_Spent
from dannys_diner.sales as S
inner join
dannys_diner.menu as M
on S.Product_id=M.Product_id
INNER JOIN
dannys_diner.MEMBERS AS K
on S.customer_id=K.customer_id
WHERE S.order_date <K.JOIN_DATE
GROUP BY S.customer_id

---10  If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?
select S.customer_id,
sum((CASE 
 WHEN M.PRODUCT_NAME='sushi' then m.PRICE*2
 else m.PRICE*10
 end)) as Total_points
from dannys_diner.sales as S
inner join
dannys_diner.menu AS M
on
S.product_id=M.product_id
group by customer_id

---11 In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?

Select
        s.customer_id
	,Sum(CASE
                 When (DATEDIFF(DAY, me.join_date, s.order_date) between 0 and 7) or (m.product_ID = 1) Then m.price * 20
                 Else m.price * 10
              END) As Points
From members as me
    Inner Join sales as s on s.customer_id = me.customer_id
    Inner Join menu as m on m.product_id = s.product_id
where s.order_date >= me.join_date and s.order_date <= CAST('2021-01-31' AS DATE)
Group by s.customer_id
