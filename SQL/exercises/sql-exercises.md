# SQL Exercises

### Exercise 1
Write a SQL query to find the top 5 customers who have spent the most money on orders. The result should include the customer’s name, email, and the total amount they have spent. The results should be ordered by the total amount spent in descending order.

Customers
- customer_id (INT, Primary Key)
- name (VARCHAR)
- email (VARCHAR)
- created_at (DATETIME)
Orders
- order_id (INT, Primary Key)
- customer_id (INT, Foreign Key)
- order_date (DATETIME)
- total_amount (DECIMAL)
OrderItems
- order_item_id (INT, Primary Key)
- order_id (INT, Foreign Key)
- product_id (INT)
- quantity (INT)
- price (DECIMAL)
Products
- product_id (INT, Primary Key)
- name (VARCHAR)
- category (VARCHAR)
- price (DECIMAL)

``` SQL
SELECT c.name, c.email, sum(oi.price * oi.quantity) as total_amount_spent
FROM Customserc c
JOIN Orders o ON c.customer_id = o.customer_id
JOIN OrderItems oi ON o.oerder_id = oi.order_id
GROUP BY c.customer_id, c.name, c.email
ORDER BY total_amount_spent DESC
LIMIT 5;
```

### Exercise 2
You will be working with a database that tracks product inventory and sales. Write a SQL query to find the top 3 products in each category that have generated the highest revenue from sales. The result should include the product name, category, and total revenue. The results should be ordered by category and then by total revenue in descending order.

Products 
- product_id (INT, Primary Key) 
- name (VARCHAR) 
- category (VARCHAR) 
- price (DECIMAL)
Inventory 
- inventory_id (INT, Primary Key) 
- product_id (INT, Foreign Key) 
- quantity (INT)
- last_updated (DATETIME) 
Sales 
- sale_id (INT, Primary Key) 
- product_id (INT, Foreign Key) 
- sale_date (DATETIME) 
- quantity (INT) 
- total_amount (DECIMAL)

``` SQL
WITH ProductRevenue AS (
	SELECT
		p.name,
		p.category,
		SUM(s.total_amount) AS total_revenue,
		ROW_NUMBER() OVER (
			PARTITION BY p.category 
			ORDER BY SUM(s.total_amount) DESC
		) AS revenu
		FROM Products p
	JOIN Sales s ON p.product_id = s.product_id
	GROUP BY p.product_id, p.name, p.category
)
SELECT name, category, total_revenue
FROM ProductRevenue
WHERE revenue_rank <= 3
ORDER BY category, total_revenue DESC;
```

##### Advanced notes
The **ROW_NUMBER()** function **assigns a unique rank to each row within the partition**: *when there are ties still assigns distinct ranks based on the order in which it processes the rows.* To handle ties more fairly and **include all tied products, you can use the RANK()** which *assigns the same rank to rows with the same values*.
##### Performance Considerations
For large datasets, consider creating indexes on product_id in the Sales table join operations

### Exercise 3
You are working for a streaming service company, and we have a database that tracks user subscriptions and their activity on the platform. We want to find out which movies have the highest average watch time across all users. The result should include the movie title, genre, and average watch time. The results should be ordered by average watch time in descending order.

Users 
- user_id (INT, Primary Key)
- name (VARCHAR)
- email (VARCHAR)
- subscription_date (DATETIME)
Movies 
- movie_id (INT, Primary Key)
- title (VARCHAR)
- genre (VARCHAR)
- release_year (INT)
UserActivity
- activity_id (INT, Primary Key)
- user_id (INT, Foreign Key)
- movie_id (INT, Foreign Key)
- watch_date (DATETIME)
- watch_time (INT) – in minutes

``` sql
SELECT 
	m.title, 
	m.genre,
	COALESCE(AVG(ua.watch_time), 0) AS avg_watch_time 
FROM Movies m
LEFT JOIN UserActivity ua ON ua.movie_id = m.movie_id 
GROUP BY m.movie_id, m.title, m.genre 
ORDER BY avg_watch_time DESC;
```
##### Advanced notes
Using a LEFT JOIN instead of an INNER JOIN will allow you to include movies that have not been watched. These movies will have a NULL average watch time, which can be replaced with 0 using COALESCE.

##### Performance considerations
Creating an index on the movie_id field in the UserActivity table can indeed help speed up the join operation.