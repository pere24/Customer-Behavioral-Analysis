# Customer-Behavioral-Analysis
Case Study #1 - Danny's Diner

![image](https://github.com/user-attachments/assets/d9dd0416-470c-4fe0-b13e-0b885a740ec3)
# Introduction 
Danny seriously loves Japanese food so at the beginning of 2021, he decides to embark upon a risky venture and opens up a cute little restaurant that sells his 3 favorite foods: sushi, curry, and ramen.

Danny’s Diner requires my assistance to help the restaurant stay afloat - it has captured some fundamental data from its few months of operation. Still, it has no idea how to use their data to help them run the business.
# Problem Statement
Danny wants to use the data to answer a few simple questions about his customers, especially about their visiting patterns, how much money they’ve spent, and also which menu items are their favorite. Having this deeper connection with his customers will help him deliver a better and more personalized experience for his loyal customers.

He plans on using these insights to help him decide whether he should expand the existing customer loyalty program - additionally, he needs help to generate some basic datasets so his team can easily inspect the data without needing to use SQL.

Danny has provided you with a sample of his overall customer data due to privacy issues - but he hopes that these examples are enough for you to write fully functioning SQL queries to help him answer his questions!

Danny has shared with you 3 key datasets for this case study:

sales
menu
members

You can inspect the entity relationship diagram and example data below.

# Entity Relationship Diagram
![image](https://github.com/user-attachments/assets/788bdd53-07cc-4170-a894-cd2abc22140c)

# Table 1: sales
The sales table captures all customer_id level purchases with a corresponding order_date and product_id information for when and what menu items were ordered.
customer_id	order_date	product_id
![image](https://github.com/user-attachments/assets/69afb5e5-4862-43ef-86ff-9653035ad405)
# TABLE 2: MENU

The menu table maps the product_id to the actual product_name and price of each menu item
![image](https://github.com/user-attachments/assets/846022e1-1ae8-4500-931d-f6d1efd2a98b)
# TABLE 3: MEMBERS

The final member's table captures the join_date when a customer_id joined the beta version of the Danny’s Diner loyalty program.


# CASE STUDY:
Each of the following case study questions can be answered using a single SQL statement:
What is the total amount each customer spent at the restaurant?
How many days has each customer visited the restaurant?
What was the first item from the menu purchased by each customer?
What is the most purchased item on the menu and how many times was it purchased by all customers?
Which item was the most popular for each customer?
Which item was purchased first by the customer after they became a member?
Which item was purchased just before the customer became a member?
What is the total items and amount spent for each member before they became a member?
If each $1 spent equates to 10 points and sushi has a 2x points multiplier — how many points would each customer have?
In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi — how many points do customers A and B have at the end of January?

# SOLUTION
# What is the total amount each customer spent at the restaurant?

SELECT customer_id, sum(price) as total price
FROM sales
JOIN menu
ON sales.product_id = menu.product_id
GROUP BY customer_id;

![image](https://github.com/user-attachments/assets/b99c6ff5-b4e1-480e-8806-25848dd1fc69)
Customer A spent 76 being the highest, Customer B spent 74, while Customer C spent 36 being the lowest

# 2. How many days has each customer visited the restaurant

SELECT customer_id, count(distinct(order_date)) as number_of_days
FROM sales
GROUP BY customer_id;

Customer B has the highest number of visiting days (5 days), while Customer C has the least number of visiting days (2 days)

# 3. What was the first item from the menu purchased by each customer?

WITH first_purchase AS
(SELECT product_id, product_name
FROM menu)
SELECT customer_id, product_name, min(order_date) AS first_order_date
FROM sales s
JOIN first_purchase f
ON s.product_id =f.product_id
GROUP BY customer_id,product_name
ORDER BY customer_id asc, min(order_date) asc;

Customer A purchased Curry and Sushi, Customer B purchased Ramen and Curry, Customer C purchased only Ramen

# 4. What is the most purchased item on the menu, and how many times was it purchased by all customers?

SSELECT product_name, SUM(number_of_item) AS Total_no_sold
        FROM(
   SELECT count(product_name) as number_of_item, product_name, customer_id
   FROM sales s
   JOIN menu m
   ON s.product_id = m.product_id
   GROUP BY customer_id, m.product_id, product_name) items
GROUP BY product_name
ORDER BY total_no_sold DESC;

Ramen is the most purchased product, and it has been purchased 8 times

# 5. What item was the most popular for each customer?

SELECT customer_id,product_name,count(product_name) as most_popular_item
FROM menu m
ON m.product_id =s.product_id
GROUP BY customer_id,product_name
ORDER BY customer_id desc, count(product_name) desc;

Customer A and C's most purchased item was Ramen, while Customer purchased all three items the same number of times

# 6. What item was purchased first by the customer after they became a member?

SELECT ms.customer_id,product_name,order_date,join_date
FROM menu m
JOIN sales s
ON m.product_id =s.product_id
JOIN members ms
ON ms.customer_id =s.customer_id
WHERE order_date >join_date;
Only Customers A and B have loyalty cards. After becoming a member, Customer A purchased Ramen, while Customer B purchased Sushi

# 7. Which items were purchased just before the customer became a member?

SELECT ms.customer_id,product_name,order_date,join_date
FROM menu m
JOIN sales s
ON m.product_id =s.product_id
JOIN members ms
ON ms.customer_id = s.customer_id
WHERE order_date <join_date
![image](https://github.com/user-attachments/assets/fc63b99d-0668-4490-9567-07357c2d81de)
Customer A became a member in 2021–01–07, while Customer B became a member in 2021–01–09. Customer A purchased Sushi and Ramen, while Customer B purchased Curry, Sushi, and Ramen

# 8. What is the total item and amount spent by each customer before they became a member?

SELECT s.customer_id,count(product_name) as total_item, sum(price) as total_amount, order_date,join_date
FROM menu m
JOIN sales s
ON m.product_id =s.product_id
JOIN members ms
ON ms.customer_id =s.customer_id
WHERE order_date <join_date
GROUP BY s.customer_id,order_date,join_date

Customer A purchased a total of 2 items and spent an amount of 25 while Customer B purchased a total of 4 items, and spent a total amount of 52.

# 9. If $1 spent equates to 10 points, and sushi has a 2x multiplier, how many points will each customer have?

SELECT customer_id,
        sum(CASE WHEN product_name ='sushi' THEN price *20 ELSE price *10 END) AS total_points
FROM sales s
JOIN menu m
ON s.product_id =m.product_id
GROUP BY customer_id;

In total, Customer A will have a point of 860, Customer B will have the highest point of 940, and Customer C will have a point of 360

# 10. In the first week after a customer joins the program(including their join date), they earn 2x on all their items, not just sushi. How many points do customers A and B have by the end of January?
WITH first_week_order AS
    (SELECT ms.customer_id, order_date, join_date, SUM(price*20) AS total_ppoints
    FROM menu m
    JOIN sales s
    ON m.product_id =s.product_id
    JOIN members ms
    ON ms.customer_id =s.customer_id 
    WHERE order_date BETWEEN '2021-01-07' AND '2021-01-14' OR order_date BETWEEN '2021-01-09' AND '2021-01-16'
    GROUP BY ms.customer_id,order_date,join_date)
SELECT fw.customer_id, fw.order_date,total_points
FROM first_week_order fw
LEFT JOIN (SELECT s.customer_id,s.order_date, SUM(price*10) AS Final_price
          FROM menu m
          JOIN sales s
    ON m.product_id = s.product_id
          JOIN members ms
    ON ms.customer_id =s.customer_id
    WHERE s.order_date BETWEEN '2021-01-14' AND '2021-01-31' AND s.order_date  BETWEEN '2021-01-16' AND '2021-01-31'
          GROUP BY s.customer_id,s.order_date) AS sub
ON fw.customer_id =sub.customer_id

![image](https://github.com/user-attachments/assets/9be548ab-ef88-4c89-8c40-e94eed785572)
At the end of January, Customer A has a total price of 1020, while Customer B has a total price of 440

# FINDINGS/INSIGHTS.

Ramen is the most purchased item while Sushi is the least purchased item.
Customer C does not have a membership card, unlike Customer A and Customer B. Customer C also has a few visiting days at the restaurant which might be a result of the Customer having only one preference for a product. Ramen is the only product Customer C purchased.
# RECOMMENDATIONS.

Ramen seems to be Customer C’s favorite item, however, customer survey data should be collected and data-driven questions should be made to understand why Customer C patronized just once. Other factors like Customer services could be related.
Before becoming a member, Ramen was purchased by the customers, after obtaining the membership card, each customer purchased a different product. The customer survey data should be further analyzed to help provide more insights into the customer experience.
To increase the purchase of the least popular item like Sushi, strategic marketing campaigns. Discounts, or adjusting the menu can be used to attract customers to try these products which will help boost their popularity and purchase.
# CONCLUSION.

The case study helped in uncovering insights about the customer's behavior and these insights would assist in further analysis regarding customer satisfaction with the products and customer service to improve Danny’s business growth.

At the end of January, Customer A has a total price of 1020, while Customer B has a total price of 440
