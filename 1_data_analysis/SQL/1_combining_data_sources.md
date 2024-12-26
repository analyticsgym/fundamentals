# Combining Data Sources

#### SQL JOINs Objective

-   generate an output dataset by merging data from two or more tables
-   use join conditions to guide the merging process (e.g. defining how table rows relate)

| Left Table | Right Table |
|------------|-------------|
| Table A    | Table B     |

```sql
-- basic join syntax
SELECT
    a.x,
    b.y
FROM table_a AS a
<<<join approach>>> table_b AS b
  ON <<<join condition>>>
```

#### INNER JOIN
-   return records where Table A and Table B have an exact match on the join condition

```sql         
WITH employees(employee_id, employee_name, department_id) AS (
  VALUES (1, 'Ann', NULL), -- Ann is the CEO
	       (2, 'John', 100),
         (3, 'Jane', 200),
         (4, 'Mark', 100),
         (5, 'Sara', 300),
         (6, 'Tom', 200)
)

, departments(department_id, department_name) AS (
  VALUES (100, 'HR'),
         (200, 'Finance'),
         (300, 'IT')
)

-- employee dropped from output if department_id not found in 'departments' table
SELECT 
    e.*,
    d.department_name
FROM employees AS e
INNER JOIN departments AS d 
    ON e.department_id = d.department_id
ORDER BY e.employee_id
```

#### LEFT JOIN
-   return all records for Table A and pull in Table B column values when the join condition is met else return NULLs

```sql         
WITH users(user_id, email) AS (
  VALUES (1, 'xyz@fakegmail.com'),
         (2, 'abc@fakegmail.com'),
         (3, '123@fakegmail.com'),
         (4, 'cat@fakegmail.com'),
         (5, 'dog@fakegmail.com')
)

, all_time_spending(user_id, total_spending) AS (
  VALUES (1, 5473),
         (2, 9876),
         (3, 24)
)

SELECT 
    u.*,
    -- NULLs expected when user spend not available (i.e. user 4 and 5)
    a.total_spending AS total_spending
FROM users AS u
-- left join to return all rows from 'users' tabel 
-- and pull in spending when available
LEFT JOIN all_time_spending AS a 
    ON a.user_id = u.user_id
```

#### RIGHT JOIN
-   symmetric to left join
-   table A column values returned when join condition met and all records returned for Table B
-   all right joins can be changed to left joins by changing the ordering of the join tables
-   not commonly used in practice (left joins preferred for consistency)

#### FULL JOIN
-   return all records from Table A and Table B with NULL fallback values when join condition not met
-   e.g. records that don't match on the join condition return just Table A values or Table B values and NULLs are filled in where there are no matches

```sql         
WITH products(product_id, product_name, price) AS (
  VALUES (1, 'Laptop', 1200),
         (2, 'Smartphone', 800),
         (3, 'Tablet', 400),
         (4, 'Smartwatch', 200),
         (5, 'Headphones', 150)
)

, sales(sale_id, product_id, sale_date) AS (
  VALUES (1, 1, '2023-01-01'),
         (2, 1, '2023-01-03'),
         (3, 2, '2023-01-05'),
         (4, 4, '2023-01-06'),
         (5, 6, '2023-01-07'),
         (6, 7, '2023-01-08')
)

-- example where sales show up for product ids not listed in the products table (i.e. 6 and 7)
SELECT 
    COALESCE(p.product_id, s.product_id) AS product_id,
    p.product_name,
    p.price,
    s.sale_id,
    s.sale_date
FROM products AS p
-- full join to return all rows from both 'products' and 'sales' when available
FULL JOIN sales AS s 
    ON s.product_id = p.product_id
ORDER BY COALESCE(p.product_id, s.product_id) ASC
```

#### CROSS JOIN
-   return all row combinations between Table A and Table B (i.e. Cartesian product)
-   use with caution as CROSS JOINs can generate large datasets and impact DB performance
-   in practice, occasionally used for analysis setup work when output isn't expected to explode in size

```sql         
WITH veg_tbl AS (
    SELECT UNNEST(ARRAY['salad', 'carrot', 'onion']) AS veg
)

, fruit_tbl AS (
    SELECT UNNEST(ARRAY['apple', 'orange', 'kiwi']) AS fruit
)

-- return veg and fruit combinations
SELECT
    veg_tbl.veg,
    fruit_tbl.fruit
FROM veg_tbl
CROSS JOIN fruit_tbl
```

#### SELF JOIN
-   joining table to itself
-   in some cases, window functions could be used in replace of a self join
-   efficiency/speed trade offs between self joins vs window functions depends on use case
-   for simple use cases, window functions tend to be more efficient/readable

```sql         
WITH sales AS (
  SELECT 
      UNNEST(ARRAY[23456, 43567, 65812, 79234, 567394]) AS annual_sales,
      UNNEST(ARRAY[1999, 2000, 2001, 2002, 2003]) AS year
)

SELECT
    current_year.year,
    current_year.annual_sales,
    prior_year.annual_sales AS prior_year_sales,
    ((current_year.annual_sales - prior_year.annual_sales)::FLOAT / prior_year.annual_sales)*100 AS yoy_sales_growth
FROM sales AS current_year
-- also joins to sales table; join condition logic is key here
LEFT JOIN sales AS prior_year
    -- this also work ==> prior_year.year + 1 = current_year.year
	ON prior_year.year = current_year.year - 1
```

#### UNION
-   combine rows of two or more queries with matching column names and matching data types
-   union drops duplicate rows

```sql         
-- returns only 2 records given james duplicate permutation
SELECT 
  1900 AS birth_year,
  'rose' AS first_name

UNION 

SELECT
  1900 AS birth_year,
  'james' AS first_name

UNION 

SELECT
  1900 AS birth_year,
  'james' AS first_name
```

#### UNION ALL
-   similar logic to UNION except retains all row permutations (does not drop duplicates)
-   UNION tends to be more resource intensive on large datasets due to additional processing to remove duplicates
-   UNION ALL is recommended when you know there are no duplicates or you want to retain duplicates

```sql         
-- returns all 3 records; includes james duplicate permutation
SELECT 
  1900 AS birth_year,
  'rose' AS first_name

UNION ALL

SELECT
  1900 AS birth_year,
  'james' AS first_name

UNION ALL

SELECT
  1900 AS birth_year,
  'james' As first_name
```

# Practical Examples

#### UNION ALL to output multiple levels of aggregation
-   stack 3 levels of aggregation results
-   UNION ALL used below as we don't expect duplicates
-   GROUPING SETS could also be used to achieve similar results with less verbose code

```sql         
WITH sales_example(date, category, amount) AS (
  VALUES
  ('2023-01-01'::date, 'Electronics', 150.00),
  ('2023-01-01'::date, 'Electronics', 300.00),
  ('2023-01-01'::date, 'Groceries', 45.00),
  ('2023-01-02'::date, 'Electronics', 200.00),
  ('2023-01-02'::date, 'Groceries', 60.00),
  ('2023-01-02'::date, 'Groceries', 40.00)
)

-- sales by day by category
SELECT
    date,
    category AS agg_label,
    SUM(amount) AS total_amount
FROM sales_example
GROUP BY date, category

UNION ALL

-- sales by day
SELECT
    date,
    'Total for the day' AS agg_label,
    SUM(amount) AS total_amount
FROM sales_example
GROUP BY date

UNION ALL

-- overall sales
SELECT
    NULL AS date,
    'Total for all dates and categories' AS category,
    SUM(amount) AS total_amount
FROM sales_example
```

#### Self join to compare monthly sales vs trailing 12 month average

```sql         
WITH monthly_sales(year_month, sales_amount) AS (
  VALUES
  (TO_DATE('2021-10', 'YYYY-MM'), 20),
  (TO_DATE('2021-11', 'YYYY-MM'), 30),
  (TO_DATE('2021-12', 'YYYY-MM'), 40),
  (TO_DATE('2022-01', 'YYYY-MM'), 90),
  (TO_DATE('2022-02', 'YYYY-MM'), 120),
  (TO_DATE('2022-03', 'YYYY-MM'), 200),
  (TO_DATE('2022-04', 'YYYY-MM'), 300),
  (TO_DATE('2022-05', 'YYYY-MM'), 400),
  (TO_DATE('2022-06', 'YYYY-MM'), 650),
  (TO_DATE('2022-07', 'YYYY-MM'), 400),
  (TO_DATE('2022-08', 'YYYY-MM'), 450),
  (TO_DATE('2022-09', 'YYYY-MM'), 500),
  (TO_DATE('2022-10', 'YYYY-MM'), 550),
  (TO_DATE('2022-11', 'YYYY-MM'), 600),
  (TO_DATE('2022-12', 'YYYY-MM'), 650),
  (TO_DATE('2023-01', 'YYYY-MM'), 700),
  (TO_DATE('2023-02', 'YYYY-MM'), 750),
  (TO_DATE('2023-03', 'YYYY-MM'), 800),
  (TO_DATE('2023-04', 'YYYY-MM'), 850)
)

, ttm_setup AS (
SELECT
  c.year_month,
  c.sales_amount,
  AVG(tm.sales_amount) AS trailing_12_months_avg_sales,
  COUNT(tm.year_month) AS trailing_12_months_count
FROM monthly_sales AS c
INNER JOIN monthly_sales AS tm
  ON tm.year_month BETWEEN c.year_month - interval '11 months' AND c.year_month
GROUP BY
  c.year_month,
  c.sales_amount
)

SELECT
    year_month,
    sales_amount,
    trailing_12_months_avg_sales,
    CASE
        WHEN trailing_12_months_count = 12
        THEN 100 * (sales_amount - trailing_12_months_avg_sales::FLOAT) / (trailing_12_months_avg_sales)
        ELSE NULL
    END AS pct_change_vs_trailing_12m_baseline
FROM ttm_setup  
ORDER BY year_month ASC
```

#### Self join to compare monthly sales vs prior year same month

```sql 
-- to get reproducible results on query reruns
SELECT SETSEED(0.11);

WITH months AS (
SELECT
    generate_series(
        date_trunc('month', current_date - INTERVAL '4 years'),
        current_date,
        INTERVAL '1 month'
    ) AS month_var
)

, example_sales_data AS (
SELECT
    month_var,
    ROUND((RANDOM() * 10000)::NUMERIC, 2) AS sales_amount
FROM months
)

SELECT
    cy.*,
    -- 1 year prior if available
    py.sales_amount AS prior_year_month_sales,
    ((cy.sales_amount - py.sales_amount) / (py.sales_amount)::FLOAT)*100 AS yoy_pct_change,
    -- 2 years prior if available
    py2.sales_amount AS prior_2_years_month_sales,
    ((cy.sales_amount - py2.sales_amount) / (py2.sales_amount)::FLOAT)*100 AS prior_2_yrs_yoy_pct_change,
    -- 3 years prior if available
    py3.sales_amount AS prior_3_years_month_sales,
    ((cy.sales_amount - py3.sales_amount) / (py3.sales_amount)::FLOAT)*100 AS prior_3_yrs_yoy_pct_change,
    -- average of the last 3 years
    (py.sales_amount + py2.sales_amount + py3.sales_amount)::FLOAT / 3 AS trailing_3_years_average_sales
FROM example_sales_data AS cy
LEFT JOIN example_sales_data AS py
    ON py.month_var::DATE = (cy.month_var::DATE - interval '12 months')
LEFT JOIN example_sales_data AS py2
    ON py2.month_var::DATE = (cy.month_var::DATE - interval '24 months')
LEFT JOIN example_sales_data AS py3
    ON py3.month_var::DATE = (cy.month_var::DATE - interval '36 months')
ORDER BY month_var DESC
```

### Cohort analysis with joins

```sql         
WITH fake_users(user_id, name, signup_date, country) AS (
    VALUES (1, 'John Doe', '2021-01-01'::DATE, 'US'),
           (2, 'Jane Smith', '2021-01-15'::DATE, 'US'),
           (3, 'Alice Johnson', '2021-02-01'::DATE, 'AU'),
           (4, 'Bob Brown', '2021-02-02'::DATE, 'DE'),
           (5, 'Charlie Green', '2021-03-10'::DATE, 'CA'),
           (6, 'David White', '2021-03-15'::DATE, 'CA'),
           (7, 'Tammy Cho', '2021-03-05'::DATE, 'CA')
)

, fake_user_activity(user_id, activity_date, activity_type) AS (
VALUES (1, '2021-01-02'::DATE, 'post'),
       (1, '2021-01-05'::DATE, 'comment'),
       (1, '2021-02-06'::DATE, 'post'),
       (1, '2021-02-09'::DATE, 'comment'),
       (2, '2021-01-16'::DATE, 'post'),
       (2, '2021-02-21'::DATE, 'post'),
       (3, '2021-02-05'::DATE, 'post'),
       (3, '2021-03-05'::DATE, 'comment'),
       (4, '2021-02-10'::DATE, 'comment'),
       (4, '2021-02-12'::DATE, 'post'),
       (4, '2021-03-18'::DATE, 'post'),
       (5, '2021-03-11'::DATE, 'post'),
       (5, '2021-03-18'::DATE, 'post'),
       (5, '2021-04-23'::DATE, 'comment'),
       (7, '2021-03-06'::DATE, 'post')
)

, cohort_sizes AS (
  SELECT 
      DATE_TRUNC('month', signup_date)::DATE AS cohort_month,
      COUNT(DISTINCT user_id) AS cohort_user_count
  FROM fake_users
  GROUP BY 1
)

-- note: this logic isn't robust enough to handle use case where an 
-- entire cohort is not active for a given activity period
SELECT
    cs.cohort_user_count,
    DATE_TRUNC('month', f2.signup_date)::DATE AS cohort_month, 
    -- each period represents 30 days; starting at day 1
    CEIL(((f1.activity_date - f2.signup_date)::FLOAT+1)/30) AS activity_period,
    COUNT(DISTINCT f1.user_id) AS active_users,
    (COUNT(DISTINCT f1.user_id)::FLOAT / cs.cohort_user_count)*100 AS pct_cohort_active
FROM fake_user_activity AS f1
INNER JOIN fake_users AS f2
    ON f2.user_id = f1.user_id
INNER JOIN cohort_sizes AS cs
    ON cs.cohort_month = DATE_TRUNC('month', f2.signup_date)::DATE
GROUP BY 1,2,3
```

### Sequential stages funnel analysis with joins

-   derive step/stage conversion rate from n-1 to n
-   sequential conversion logic: number of users to complete stage n / number of users who completed stage n-1

```sql         
CREATE TEMP TABLE funnel_base(
    user_id INT,
    date DATE,
    action VARCHAR
);

INSERT INTO funnel_base(user_id, date, action)
VALUES
    (1, '2023-06-01', 'website_visit'),
    (2, '2023-06-01', 'website_visit'),
    (3, '2023-06-02', 'website_visit'),
    (4, '2023-06-03', 'website_visit'),
    (1, '2023-06-02', 'purchase_click'),
    (2, '2023-06-03', 'purchase_click'),
    (3, '2023-06-04', 'purchase_click'),
    (1, '2023-06-02', 'add to cart'),
    (3, '2023-06-04', 'add to cart'),
    (3, '2023-06-04', 'order completed');

SELECT
    COUNT(DISTINCT wv.user_id) AS website_vistors,
    COUNT(DISTINCT pc.user_id) AS users_with_purchase_click,
    COUNT(DISTINCT ac.user_id) AS users_with_add_to_cart,
    COUNT(DISTINCT oc.user_id) AS users_with_order_complete,
    ---------- step 1 cvr ---------- 
    COUNT(DISTINCT pc.user_id) / 
      COUNT(DISTINCT wv.user_id)::FLOAT AS visitor_to_purchase_click_conv_rate,
    ---------- step 2 cvr ---------- 
    COUNT(DISTINCT ac.user_id) / 
      COUNT(DISTINCT pc.user_id)::FLOAT AS purchase_click_to_add_to_cart_conv_rate,
    ---------- step 3 cvr ---------- 
    COUNT(DISTINCT oc.user_id) / 
      COUNT(DISTINCT ac.user_id)::FLOAT AS add_to_cart_to_order_complete_conv_rate
FROM funnel_base AS wv
LEFT JOIN funnel_base AS pc 
    ON wv.user_id = pc.user_id
    AND pc.action = 'purchase_click'
LEFT JOIN funnel_base AS ac 
    ON ac.user_id = pc.user_id
    AND ac.action = 'add to cart'
LEFT JOIN funnel_base AS oc 
    ON oc.user_id = ac.user_id
    AND oc.action = 'order completed'
WHERE wv.action = 'website_visit'
```

### Funnel analysis v2 without joins

-   different flavor of funnel analysis (likely more efficient than logic with joins above)
-   example below deriving funnel step conversion based on total visitors and step to step conversion rate

```sql         
-- temp table created in above code logic
WITH action_counts AS (
  SELECT 
    ---------- visitors ---------- 
    COUNT(
      DISTINCT 
        CASE 
          WHEN action = 'website_visit' 
          THEN user_id 
        END
    ) AS website_vistors,
   ---------- purchase clicks ----------
    COUNT(
      DISTINCT 
        CASE 
          WHEN action = 'purchase_click' 
          THEN user_id 
        END
    ) AS users_with_purchase_click,
    ---------- add to cart ----------
    COUNT(
      DISTINCT 
        CASE 
          WHEN action = 'add to cart' 
          THEN user_id 
        END
    ) AS users_with_add_to_cart,
    ---------- order completed ----------
    COUNT(
      DISTINCT 
        CASE 
          WHEN action = 'order completed' 
          THEN user_id 
        END
    ) AS users_with_order_complete
  FROM funnel_base
)

SELECT
  action_counts.*,
  users_with_purchase_click / website_vistors::FLOAT AS pct_visitors_with_purchase_click,
  users_with_add_to_cart / website_vistors::FLOAT AS pct_visitors_with_add_to_cart,
  users_with_order_complete / website_vistors::FLOAT AS pct_visitors_with_order_complete,
  users_with_add_to_cart / users_with_purchase_click::FLOAT AS pct_purchase_clicks_to_add_to_cart,
  users_with_order_complete / users_with_add_to_cart::FLOAT AS pct_add_to_cart_to_order_complete
FROM action_counts
```

### Self join for basic affinity analysis
- find the most common pairs of movies watched together
- for large datasets consider R or Python for more efficient processing/algorithms

```sql         
WITH movies_watched(user_id, movie_title) AS (
  VALUES
    (0, 'The Shawshank Redemption'),
	(1, 'The Shawshank Redemption'),
    (2, 'The Shawshank Redemption'),
    (2, 'The Dark Knight'),
    (3, 'The Godfather'),
    (4, 'The Godfather'),
    (4, '12 Angry Men'),
    (5, 'The Godfather: Part II'),
    (5, 'Pulp Fiction'),
    (5, 'Fight Club'),
    (6, 'The Dark Knight'),
    (6, 'The Lord of the Rings: The Return of the King'),
    (7, '12 Angry Men'),
    (8, 'Schindler''s List'),
    (8, 'The Good, the Bad and the Ugly'),
    (9, 'Pulp Fiction'),
    (10, 'The Lord of the Rings: The Return of the King'),
    (10, 'Fight Club'),
    (11, 'The Good, the Bad and the Ugly'),
    (12, 'Fight Club'),
    (13, 'The Shawshank Redemption'),
    (14, 'The Godfather'),
    (15, 'The Godfather: Part II'),
    (15, 'The Dark Knight'),
    (16, '12 Angry Men'),
    (17, 'Schindler''s List'),
    (18, 'Pulp Fiction'),
    (18, 'The Lord of the Rings: The Return of the King'),
    (19, 'The Good, the Bad and the Ugly'),
    (19, 'Fight Club'),
    (20, 'The Shawshank Redemption'),
    (20, 'The Godfather'),
    (20, 'The Godfather: Part II'),
    (20, 'The Dark Knight'),
	(21, 'The Dark Knight'),
	(21, 'The Shawshank Redemption')
)

SELECT 
  m1.movie_title AS movie_pair_slot_1, 
  m2.movie_title AS movie_pair_slot_2,
  COUNT(m1.user_id) AS user_count
FROM movies_watched AS m1 
INNER JOIN movies_watched AS m2 
  ON m2.user_id = m1.user_id
  -- avoid duplicate pairs with 
  AND m1.movie_title < m2.movie_title
GROUP BY 1,2
ORDER BY user_count DESC
```
