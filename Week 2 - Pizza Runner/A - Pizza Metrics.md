### A. Pizza Metrics
**A.1: How many pizzas were ordered?**
|number_pizzas_ordered| 
|--|
|14|

    SELECT
    	COUNT(order_id) as number_pizzas_ordered
    FROM pizza_runner.customer_orders;

**A.2: How many unique customer orders were made?**
|unique_customer_orders| 
|--|
|10|

    SELECT
    	COUNT(DISTINCT order_id) as unique_customer_orders
    FROM pizza_runner.customer_orders;

**A.3: How many successful orders were delivered by each runner?**

|runner_id| successful_orders_delivered |
|--|--|
| 1 | 4 |
| 2 | 3 |
| 3 | 1 |

    SELECT
    	runner_id, COUNT(order_id) as successful_orders_delivered
    FROM pizza_runner.runner_orders
    WHERE duration != 'null'
    GROUP BY runner_id
    ORDER BY runner_id;

**A.4: How many of each type of pizza was delivered?**
|pizza_id| num_delivered |
|--|--|
| 1 | 9 |
| 2 | 3 |

    SELECT
    	pizza_id, COUNT(pizza_id) as num_delivered
    FROM pizza_runner.runner_orders
    JOIN pizza_runner.customer_orders
    ON runner_orders.order_id = customer_orders.order_id
    WHERE duration != 'null'
    GROUP BY pizza_id
    ORDER BY pizza_id;

**A.5: How many Vegetarian and Meatlovers were ordered by each customer?**
|customer_id|pizza_name|num_ordered|
|--|--|--|
| 101 | Meatlovers | 2 |
| 101 | Vegetarian| 1 |
| 102 | Meatlovers | 2 |
| 102 | Vegetarian| 1 |
| 103 | Meatlovers | 3 |
| 103 | Vegetarian| 1 |
| 104 | Meatlovers | 3 |
| 105 | Vegetarian| 1 |

    SELECT
    	customer_id, pizza_name, COUNT(pizza_name) as num_ordered
    FROM pizza_runner.customer_orders
    JOIN pizza_runner.pizza_names
    ON customer_orders.pizza_id = pizza_names.pizza_id
    WHERE pizza_name = 'Meatlovers' OR pizza_name = 'Vegetarian'
    GROUP BY customer_id, pizza_name
    ORDER BY customer_id, pizza_name;

**A.6: What was the maximum number of pizzas delivered in a single order?**
|max_pizzas|
|--|
| 3 | 

    WITH cte AS 
    (
    SELECT
          runner_orders.order_id, COUNT(customer_orders.pizza_id) as pizzas_per_order
      FROM pizza_runner.runner_orders
      JOIN pizza_runner.customer_orders
      ON runner_orders.order_id = customer_orders.order_id
      WHERE duration != 'null'
      GROUP BY runner_orders.order_id
    )
    
    SELECT 
    	MAX(pizzas_per_order) as max_pizzas
    FROM cte;

**A.7: For each customer, how many delivered pizzas had at least 1 change and how many had no changes?**

**A.8: How many pizzas were delivered that had both exclusions and extras?**

**A.9: What was the total volume of pizzas ordered for each hour of the day?**

**A.10: What was the volume of orders for each day of the week?**