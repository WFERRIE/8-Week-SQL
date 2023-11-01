# SQL Challenge Week 1: Danny's Diner
This case study can be found [here](https://8weeksqlchallenge.com/case-study-1/).

Below is the provided entity relationship diagram:

![Database Schema](image.png)

## Case Study Questions
1. What is the total amount each customer spent at the restaurant?
2. How many days has each customer visited the restaurant?
3. What was the first item from the menu purchased by each customer?
4. What is the most purchased item on the menu and how many times was it purchased by all customers?
5. Which item was the most popular for each customer?
6. Which item was purchased first by the customer after they became a member?
7. Which item was purchased just before the customer became a member?
8. What is the total items and amount spent for each member before they became a member?
9.  If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?
10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?


### 1. What is the total amount each customer spent at the restaurant?
For this we need to access the price of the menu items as well as the sales history, so we will need to use a join on the product_id columns in the sales and menu tables. Then we simply group by the customer_id field and sum the prices of their orders to get our final answer:

|customer_id|total_sales  |
|--|--|
| A | 76 |
| B | 74 |
| C | 36 |

The query used is:

    SELECT
    customer_id, SUM(price) as "total_sales"
    FROM dannys_diner.sales
    JOIN dannys_diner.menu
    ON sales.product_id = menu.product_id
    GROUP BY customer_id
    ORDER BY customer_id;


### 2. How many days has each customer visited the restaurant?
Here we only need the sales table, and we can just select the customer_id field, and use the COUNT() function on the unique order_date values. Again, we then group by the customer_id to get the following:

|customer_id|times_visited|
|--|--|
| A | 4 |
| B | 6 |
| C | 2 |

The query used is:

    SELECT
    customer_id, COUNT(DISTINCT order_date) as "times_visited"
    FROM dannys_diner.sales
    GROUP BY customer_id
    ORDER BY customer_id;

### 3. What was the first item from the menu purchased by each customer?
Here we need to assign an order to the orders using the RANK() function so we can get the first order. SQL doesn't allow you to constrain via the WHERE clause for aliases created in the SELECT statement, so instead we have to use a common table expression. Alternatively we could use a subquery (I have included both approaches). Either way you get the following answer:

|customer_id|product_name|
|--|--|
| A | curry |
| A | sushi |
| B | curry |
| C | ramen |

Using common table expression:

    WITH cte AS (
      SELECT customer_id, order_date, product_name, RANK() OVER(ORDER BY order_date) as ranks
      FROM dannys_diner.sales
      JOIN dannys_diner.menu
      ON sales.product_id = menu.product_id
      ORDER BY order_date
      )
      
    SELECT DISTINCT customer_id, product_name
    FROM cte
    WHERE ranks = 1
    ORDER BY customer_id, product_name;

Using a subquery:

    SELECT customer_id, product_name
    FROM (
      SELECT DISTINCT customer_id, order_date, product_name, RANK() OVER(ORDER BY order_date) as ranks
      FROM dannys_diner.sales
      JOIN dannys_diner.menu ON sales.product_id = menu.product_id
    ) AS RankedSales
    WHERE ranks = 1
    ORDER BY customer_id, product_name;


### 4. What is the most purchased item on the menu and how many times was it purchased by all customers?
For this, we just need to use the COUNT() function to count the number of times in the sales table that each product was ordered. We then GROUP BY the product, and ORDER BY the order count, ensure it is sorted in DESC order, and set LIMIT to 1. Then we get the following answer:

|product_name|product_order_count|
|--|--|
| ramen | 8 |

To get this I used the following query:

    SELECT menu.product_name, COUNT(sales.product_id) as "product_order_count"
    FROM dannys_diner.sales
    JOIN dannys_diner.menu
    ON sales.product_id = menu.product_id
    GROUP BY menu.product_name
    ORDER BY product_order_count DESC LIMIT 1;

### 5. Which item was the most popular for each customer?
For this, we are going to use another common table expression. In the cte, we are going to select the customer_id, product_name, the COUNT() of the product_id, and then the DENSE_RANK() function partitioned by customer_id, and ordered by the count of product_id from the sales table. Finally we GROUP BY the customer_id and the product names. The idea here is that we use COUNT(sales.product_id) in order to get the number of times each product was purchased by a given customer, and then we use DENSE_RANK() to rank these counts on a customer-by-customer basis in order of highest to lowest. In the outer query, we simply select the relevant information from the cte and filter for only the first rank to get the most frequently ordered product.

|customer_id|product_name|order_count|
|--|--|--|
| A | ramen |3|
| B| ramen |2|
| B | curry |2|
| B | sushi |2|
| C | ramen |3|



    WITH cte AS (
      SELECT 
      	sales.customer_id,
      	menu.product_name,
      	COUNT(sales.product_id) AS order_count,
      	DENSE_RANK() OVER (
          	PARTITION BY sales.customer_id
          	ORDER BY COUNT(sales.product_id) DESC
          ) AS ranks
      
      FROM dannys_diner.sales
      JOIN dannys_diner.menu
      ON sales.product_id = menu.product_id
      GROUP BY sales.customer_id, menu.product_name
      )
      
    SELECT customer_id, product_name, order_count
    FROM cte
    WHERE ranks = 1;

### 6. Which item was purchased first by the customer after they became a member?
Again, we need to use a common table expression. For the first time, we will now use the menu table, and join it with the sales table on the customer_id fields. This way we only consider customers who are members (exist in the members table). We can then include a WHERE statement to filter for orders that occurred after the member's join_date. In our SELECT statement, we get the customer_id, and product_id, and then use the DENSE_RANK() window function partitioned by customer_id and ordered by order_date. The idea here is that for each customer, we want to rank their orders from oldest to newest (because of the aforementioned WHERE statement, we will only be getting orders that were placed after they became members). Then in our outer query, we do another JOIN on the cte's product_id field and the menu table's product_id field so that we can access the product_name in our SELECT statement. We then use a WHERE statement to filter for the first rank to get the first item each customer ordered after they became a member.


|customer_id|product_name|
|--|--|
| A | ramen|
| B| sushi|



    WITH cte AS (
      SELECT 
      	members.customer_id, 
      	sales.product_id, 
      	DENSE_RANK() OVER(PARTITION BY members.customer_id ORDER BY sales.order_date) AS ranks
      FROM dannys_diner.members
      JOIN dannys_diner.sales
      ON members.customer_id = sales.customer_id
      WHERE sales.order_date > members.join_date
    )
    
    
    SELECT cte.customer_id,	cte.product_id, menu.product_name, cte.ranks
    FROM cte
    JOIN dannys_diner.menu
    ON cte.product_id = menu.product_id
    WHERE ranks = 1
    ORDER BY cte.customer_id;


### 7. Which item was purchased just before the customer became a member?
This is very similar to question 6, but instead of filtering for sales after the join date, we filter for sales before the join date. We also need to ensure in the DENSE_RANK() window function we are ordering in a DESC manner. We end up getting the following result, where customer A ordered sushi and curry, and customer B ordered just sushi:

|customer_id|product_name|
|--|--|
| A | sushi|
| A| curry|
| B| sushi|

    WITH cte AS (
      SELECT 
        members.customer_id, 
        sales.product_id, 
        sales.order_date,
        DENSE_RANK() OVER(PARTITION BY members.customer_id ORDER BY sales.order_date DESC) AS ranks
      FROM dannys_diner.members
      JOIN dannys_diner.sales
      ON members.customer_id = sales.customer_id
      WHERE sales.order_date < members.join_date
    )
    
    
    SELECT cte.customer_id, menu.product_name
    FROM cte
    JOIN dannys_diner.menu
    ON cte.product_id = menu.product_id
    WHERE ranks = 1
    ORDER BY cte.customer_id;

### 8. What is the total items and amount spent for each member before they became a member?

Similarly to question 7, we ensure that in our common table expression we are using a WHERE statement to filter for only sales made before a member's join date. Then in our cte we don't need to worry about rank anymore so we can remove the DENSE_RANK() window function, and remove the `WHERE ranks = 1` statement in the outer query. Then, in the outer query we simply group by customer_id, and SELECT the COUNT of product_id and the SUM of prices.

|customer_id|total_items|total_spent|
|--|--|--|
| A |2|25|
| B | 3|40|

    WITH cte AS (
      SELECT 
        members.customer_id, 
        sales.product_id, 
        sales.order_date
      FROM dannys_diner.members
      JOIN dannys_diner.sales
      ON members.customer_id = sales.customer_id
      WHERE sales.order_date < members.join_date
    )
    
    
    SELECT cte.customer_id, COUNT(cte.product_id) as total_items, SUM(price) as total_spent
    FROM cte
    JOIN dannys_diner.menu
    ON cte.product_id = menu.product_id
    GROUP BY customer_id
    ORDER BY cte.customer_id;

### 9. If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?
Again, we use a common table expression. In our cte's SELECT statement, we use a CASE expression. This way, we create a new column called "points", and assign it 20x the value of the item price if the item is sushi, or 10x the value of the item price otherwise. Then, in our outer query, we can simply group by the customer_id and sum up all of their points earned.

|customer_id|points_earned|
|--|--|
|A|860|
|B|940|
|C|360|

    WITH cte AS (
      SELECT 
        sales.customer_id,
      	sales.product_id,
      	menu.product_name,
      	menu.price,
      	CASE
      		WHEN menu.product_name = 'sushi' THEN menu.price * 20
      		ELSE menu.price * 10
      	END AS points
      FROM dannys_diner.sales
      JOIN dannys_diner.menu
      ON sales.product_id = menu.product_id
    )
   
    
    SELECT customer_id, SUM(points) AS points_earned
    FROM cte
    GROUP BY customer_id
    ORDER BY customer_id;

### 10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customers A and B have at the end of January?
For this query we do pretty much the same thing as in question 9, the big difference however is that we have to add a bit more logic to our CASE statement. We first check if the order date falls into the 1-week 'bonus' period once they join as a member. If it does, we assign 20 points per dollar. If it doesn't then it moves on to the next WHEN statement, which checks if it is sushi, in which case it assigns 20 points per dollar. If the order is not sushi, then it just assigns 10 points per dollar. We also ensure using a WHERE statement that we are only considering orders in the month of January.

|customer_id|january_points|
|--|--|
|A|1370|
|B|820|

    WITH cte AS (
      SELECT 
        members.customer_id,
      	sales.product_id,
      	menu.product_name,
      	sales.order_date,
      	members.join_date,
        menu.price,
      	CASE
      		WHEN sales.order_date <= members.join_date + 6 AND sales.order_date >= members.join_date THEN menu.price * 20
      		WHEN menu.product_name = 'sushi' THEN menu.price * 20
      		ELSE menu.price * 10
      	END AS points
      FROM dannys_diner.members
      JOIN dannys_diner.sales
      ON members.customer_id = sales.customer_id
      JOIN dannys_diner.menu
      ON sales.product_id = menu.product_id
      WHERE sales.order_date < DATE('2021-02-01')
    )
    
    
    SELECT customer_id, SUM(points) AS january_points
    FROM cte
    GROUP BY customer_id;


