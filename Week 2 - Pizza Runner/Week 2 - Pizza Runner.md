
# SQL Challenge Week 2: Pizza Runner
This case study can be found [here](https://8weeksqlchallenge.com/case-study-2/)
  

![Database Schema](image.png)

  

## Case Study Questions

### A. Pizza Metrics

  

A.1: How many pizzas were ordered?

A.2: How many unique customer orders were made?

A.3: How many successful orders were delivered by each runner?

A.4: How many of each type of pizza was delivered?

A.5: How many Vegetarian and Meatlovers were ordered by each customer?

A.6: What was the maximum number of pizzas delivered in a single order?

A.7: For each customer, how many delivered pizzas had at least 1 change and how many had no changes?

A.8: How many pizzas were delivered that had both exclusions and extras?

A.9: What was the total volume of pizzas ordered for each hour of the day?

A.10: What was the volume of orders for each day of the week?

  

### B. Runner and Customer Experience

  

B.1: How many runners signed up for each 1 week period? (i.e. week starts 2021-01-01)

B.2: What was the average time in minutes it took for each runner to arrive at the Pizza Runner HQ to pickup the order?

B.3: Is there any relationship between the number of pizzas and how long the order takes to prepare?

B.4: What was the average distance travelled for each customer?

B.5: What was the difference between the longest and shortest delivery times for all orders?

B.6: What was the average speed for each runner for each delivery and do you notice any trend for these values?

B.7: What is the successful delivery percentage for each runner?

  

### C. Ingredient Optimisation

  

C.1: What are the standard ingredients for each pizza?

C.2: What was the most commonly added extra?

C.3: What was the most common exclusion?

C.4: Generate an order item for each record in the customers_orders table in the format of one of the following:

- Meat Lovers

- Meat Lovers - Exclude Beef

- Meat Lovers - Extra Bacon

- Meat Lovers - Exclude Cheese, Bacon - Extra Mushroom, Peppers

C.5: Generate an alphabetically ordered comma separated ingredient list for each pizza order from the customer_orders table and add a 2x in front of any relevant ingredients

- For example: "Meat Lovers: 2xBacon, Beef, ... , Salami"

C.6: What is the total quantity of each ingredient used in all delivered pizzas sorted by most frequent first?

  

### D. Pricing and Ratings

  

D.1: If a Meat Lovers pizza costs $12 and Vegetarian costs $10 and there were no charges for changes - how much money has Pizza Runner made so far if there are no delivery fees?

D.2: What if there was an additional $1 charge for any pizza extras?

D.3: Add cheese is $1 extra

D.4: The Pizza Runner team now wants to add an additional ratings system that allows customers to rate their runner, how would you design an additional table for this new dataset - generate a schema for this new table and insert your own data for ratings for each successful customer order between 1 to 5.

D.5: Using your newly generated table - can you join all of the information together to form a table which has the following information for successful deliveries?

    customer_id
    
    order_id
    
    runner_id
    
    rating
    
    order_time
    
    pickup_time
    
    Time between order and pickup
    
    Delivery duration
    
    Average speed
    
    Total number of pizzas

D.6: If a Meat Lovers pizza was $12 and Vegetarian $10 fixed prices with no cost for extras and each runner is paid $0.30 per kilometre traveled - how much money does Pizza Runner have left over after these deliveries?
