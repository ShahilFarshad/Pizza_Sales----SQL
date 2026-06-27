# 🍕 Pizza Sales Analysis Using SQL

A comprehensive SQL-based data analysis project that transforms raw pizza restaurant transactional data into meaningful business insights — covering sales trends, customer ordering patterns, revenue performance, and product popularity.

---

## 📁 Database Schema

The project uses a relational database with 4 tables:

| Table | Key Columns |
|---|---|
| `orders` | `order_id`, `date`, `time` |
| `order_details` | `order_details_id`, `order_id`, `pizza_id`, `quantity` |
| `pizzas` | `pizza_id`, `pizza_type_id`, `size`, `price` |
| `pizza_types` | `pizza_type_id`, `name`, `category`, `ingredients` |

**Relationships:** `orders` → `order_details` → `pizzas` → `pizza_types`

---

## 📊 Analysis & SQL Queries

### 🟢 Basic

---

**1. Retrieve the total number of orders placed**

```sql
SELECT
    COUNT(order_id) AS `Total Orders`
FROM
    orders;
```

**Result:** `21,350` total orders

---

**2. Calculate the total revenue generated from pizza sales**

```sql
SELECT
    ROUND(SUM(order_details.quantity * pizzas.price), 2) AS `Total Revenue`
FROM
    order_details
JOIN
    pizzas ON order_details.pizza_id = pizzas.pizza_id;
```

**Result:** `$817,860.05`

---

**3. Identify the highest-priced pizza**

```sql
SELECT
    pizza_types.name, pizzas.price
FROM
    pizza_types
JOIN
    pizzas ON pizza_types.pizza_type_id = pizzas.pizza_type_id
ORDER BY pizzas.price DESC
LIMIT 1;
```

**Result:** The Greek Pizza — `$35.95`

---

**4. Identify the most common pizza size ordered**

```sql
SELECT
    pizzas.size,
    COUNT(order_details.order_details_id) AS order_count
FROM
    pizzas
JOIN
    order_details ON pizzas.pizza_id = order_details.pizza_id
GROUP BY pizzas.size
ORDER BY order_count DESC
LIMIT 1;
```

**Result:** Size `L` (Large) — `18,526` orders

---

**5. List the top 5 most ordered pizza types along with their quantities**

```sql
SELECT
    pizza_types.name, SUM(order_details.quantity) AS Quantity
FROM
    pizza_types
JOIN
    pizzas ON pizza_types.pizza_type_id = pizzas.pizza_type_id
JOIN
    order_details ON order_details.pizza_id = pizzas.pizza_id
GROUP BY pizza_types.name
ORDER BY Quantity DESC
LIMIT 5;
```

**Result:**

| Pizza Name | Quantity |
|---|---|
| The Classic Deluxe Pizza | 2,453 |
| The Barbecue Chicken Pizza | 2,432 |
| The Hawaiian Pizza | 2,422 |
| The Pepperoni Pizza | 2,418 |
| The Thai Chicken Pizza | 2,371 |

---

### 🟡 Intermediate

---

**6. Find the total quantity of each pizza category ordered**

```sql
SELECT
    pizza_types.category,
    SUM(order_details.quantity) AS `Total quantity`
FROM
    pizza_types
JOIN
    pizzas ON pizza_types.pizza_type_id = pizzas.pizza_type_id
JOIN
    order_details ON order_details.pizza_id = pizzas.pizza_id
GROUP BY pizza_types.category
ORDER BY `Total quantity` DESC;
```

**Result:**

| Category | Total Quantity |
|---|---|
| Classic | 14,888 |
| Supreme | 11,987 |
| Veggie | 11,649 |
| Chicken | 11,050 |

---

**7. Determine the distribution of orders by hour of the day**

```sql
SELECT
    HOUR(order_time) AS hours, COUNT(order_id) AS order_count
FROM
    orders
GROUP BY HOUR(order_time);
```

**Result:** Peak hours are **12 PM (2,520 orders)** and **1 PM (2,455 orders)**, with a secondary dinner spike at **6 PM (2,399 orders)**.

---

**8. Find the category-wise distribution of pizzas**

```sql
SELECT
    category, COUNT(name) AS count
FROM
    pizza_types
GROUP BY category;
```

**Result:**

| Category | Count |
|---|---|
| Chicken | 6 |
| Classic | 8 |
| Supreme | 9 |
| Veggie | 9 |

---

**9. Calculate the average number of pizzas ordered per day**

```sql
SELECT
    ROUND(AVG(quantity), 0) AS Avg_Pizzas_ordered_per_day
FROM
    (SELECT
        orders.order_date, SUM(order_details.quantity) AS quantity
    FROM
        orders
    JOIN order_details ON orders.order_id = order_details.order_id
    GROUP BY orders.order_date) AS order_quantity;
```

**Result:** `138` pizzas ordered per day on average

---

**10. Determine the top 3 most ordered pizza types based on revenue**

```sql
SELECT
    pizza_types.name,
    SUM(order_details.quantity * pizzas.price) AS Revenue
FROM
    pizza_types
JOIN
    pizzas ON pizza_types.pizza_type_id = pizzas.pizza_type_id
JOIN
    order_details ON order_details.pizza_id = pizzas.pizza_id
GROUP BY pizza_types.name
ORDER BY Revenue DESC
LIMIT 3;
```

**Result:**

| Pizza Name | Revenue |
|---|---|
| The Thai Chicken Pizza | $43,434.25 |
| The Barbecue Chicken Pizza | $42,768.00 |
| The California Chicken Pizza | $41,409.50 |

---

### 🔴 Advanced

---

**11. Calculate the percentage contribution of each pizza category to total revenue**

```sql
SELECT
    pizza_types.category,
    CONCAT(ROUND(SUM(order_details.quantity * pizzas.price) / (SELECT
        ROUND(SUM(order_details.quantity * pizzas.price), 2) AS `Total Revenue`
    FROM
        order_details
    JOIN
        pizzas ON pizzas.pizza_id = order_details.pizza_id) * 100, 2), '%') AS revenue
FROM
    pizza_types
JOIN
    pizzas ON pizza_types.pizza_type_id = pizzas.pizza_type_id
JOIN
    order_details ON order_details.pizza_id = pizzas.pizza_id
GROUP BY pizza_types.category
ORDER BY revenue DESC;
```

**Result:**

| Category | Revenue Contribution |
|---|---|
| Classic | 26.91% |
| Supreme | 25.46% |
| Chicken | 23.96% |
| Veggie | 23.68% |

---

**12. Analyze the cumulative revenue generated over time**

```sql
SELECT
    order_date,
    ROUND(SUM(revenue) OVER (ORDER BY order_date), 2) AS cum_sales
FROM
    (SELECT
        orders.order_date,
        SUM(order_details.quantity * pizzas.price) AS revenue
    FROM
        order_details
    JOIN
        pizzas ON order_details.pizza_id = pizzas.pizza_id
    JOIN
        orders ON orders.order_id = order_details.order_id
    GROUP BY orders.order_date) AS sales;
```

**Result:** Cumulative revenue grows steadily, starting from `$2,713.85` on 2015-01-01, reflecting consistent daily sales throughout the year.

---

**13. Determine the top 3 most ordered pizza types based on revenue for each pizza category**

```sql
SELECT name, revenue
FROM
    (SELECT
        category, name, revenue,
        RANK() OVER (PARTITION BY category ORDER BY revenue DESC) AS rn
    FROM
        (SELECT
            pizza_types.category,
            pizza_types.name,
            SUM(order_details.quantity * pizzas.price) AS revenue
        FROM
            pizza_types
        JOIN
            pizzas ON pizza_types.pizza_type_id = pizzas.pizza_type_id
        JOIN
            order_details ON order_details.pizza_id = pizzas.pizza_id
        GROUP BY pizza_types.category, pizza_types.name) AS sales) AS b
WHERE rn <= 3;
```

**Result:**

| Category | Pizza Name | Revenue |
|---|---|---|
| Chicken | The Thai Chicken Pizza | $43,434.25 |
| Chicken | The Barbecue Chicken Pizza | $42,768.00 |
| Chicken | The California Chicken Pizza | $41,409.50 |
| Classic | The Classic Deluxe Pizza | $38,180.50 |
| Classic | The Hawaiian Pizza | $32,273.25 |
| Classic | The Pepperoni Pizza | $30,161.75 |
| Supreme | The Spicy Italian Pizza | $34,831.25 |
| Supreme | The Italian Supreme Pizza | $33,476.75 |
| Supreme | The Sicilian Pizza | $30,940.50 |
| Veggie | The Four Cheese Pizza | $32,265.70 |
| Veggie | The Mexicana Pizza | $26,780.75 |
| Veggie | The Five Cheese Pizza | $26,066.50 |

---

## 🔍 Key Insights

- **Total Orders:** 21,350 | **Total Revenue:** $817,860.05
- **Most Popular Size:** Large (L) dominates with 18,526 orders
- **Peak Hours:** Lunch (12–1 PM) and Dinner (6–7 PM)
- **Top Revenue Category:** Classic pizzas lead at 26.91% of total revenue
- **Best Selling Pizza by Quantity:** The Classic Deluxe Pizza (2,453 units)
- **Highest Revenue Pizza:** The Thai Chicken Pizza ($43,434.25)
- **Most Expensive Pizza:** The Greek Pizza at $35.95
- **Average Daily Orders:** 138 pizzas per day

---

## 🛠️ Tools & Technologies

- **Database:** MySQL
- **SQL Concepts Used:** JOINs, Aggregate Functions (SUM, COUNT, AVG, ROUND), GROUP BY, ORDER BY, Subqueries, Window Functions (SUM OVER, RANK OVER PARTITION BY), HOUR(), CONCAT()

---

## 👤 Author

**Shahil Farshad**  
Data Analyst | SQL | Power BI | Python  
[GitHub](https://github.com/ShahilFarshad)
[LinkedIn].(www.linkedin.com/in/shahil-farshad-287374370)
