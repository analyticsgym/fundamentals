#### Window functions basics

-   used to apply functions OVER a result set of data (e.g. window of data)
-   all window functions have an OVER clause
-   number of window function output rows = number of input rows (vs aggregate functions return single summarized output)
-   the OVER clause has 3 main parts: partition, order, and frame
-   depending on the function, argument input used to specify the column or expression to operate on

```         
-- skeleton window function structure
WINDOW_FUNCTION(argument(s)_if_needed) OVER(
    PARTITION BY X
    ORDER BY Y
    FRAME TYPE
) AS window_metric_x
```

#### Partition

-   the partition describes the result set of data to apply the function
-   we can think of a partition as a cut or segment of the data

#### Order

-   orders the rows within each partition / result set (e.g. how to order rows for a ranking window function)
-   inclusion or exclusion of the order by clause can change the behavior of the function (i.e. without an order by clause certain aggregate functions will be applied to the entire result set vs with the order by clause the aggregate function returns a running aggregate)

#### Frame

-   window frame clause is used to describe which records to process in relation to the current row
-   when not specified the default window frame is used for each window function

#### Types of window frames

-   ROWS frame: define by fixed number of rows relative to the current row (below frame boundary keywords can be used to specify number of rows)
-   RANGE frame: include records within boundary in relation to current row (below keywords can be used like ROWS)
-   GROUPS frame: defined by a fixed number of peer groups relative to the current row's peer group (not included as part of all DB vendors using SQL)

#### Window frame boundaries

-   UNBOUNDED PRECEDING: includes all rows before the current row
-   UNBOUNDED FOLLOWING: includes all rows after the current row
-   CURRENT ROW: includes only the current row
-   N PRECEEDING: include N number of rows before current row
-   N FOLLOWING: include N number of rows after current row
-   Typical window frame boundary **with** ORDER BY clause: "UNBOUNDED PRECEDING" to "CURRENT ROW"
-   Typical window frame boundary **without** ORDER BY clause used: entire window partition (e.g. "UNBOUNDED PRECEDING" to "UNBOUNDED FOLLOWING")

#### Frame exclusions

-   PostgreSQL also supports the frame exclusion keywords which can be used to exclude rows from the window frame operation
-   not available across all DB vendors
-   examples include: EXCLUDE CURRENT ROW, EXCLUDE GROUP, EXCLUDE TIES

#### Window function outputs

-   important note: the same window function (i.e. COUNT) can have different output values based on various argument, partition, order by, and frame differences (see below)

#### COUNT()

-   used to count the number of rows in a result set
-   note different results for different specifications of COUNT() function

``` sql
WITH customer_orders(order_id, customer_id, order_date) AS (
  VALUES 
  (1, 101, '2024-01-01'),
  (2, 102, '2024-01-03'),
  (3, 101, '2024-01-04'),
  (4, 103, '2024-01-04'),
  (5, 102, '2024-01-05'),
  (6, 104, '2024-01-05'),
  (7, 101, '2024-01-07'),
  -- add NULL values for COUNT example
  (NULL, 101, NULL)
)

SELECT
  *,
  ----------------------------------------
  -- numnber of result set rows using *
  ----------------------------------------
  COUNT(*) OVER() AS number_of_rows,
  ----------------------------------------
  -- column name agrument used to count non-NULL values
  ----------------------------------------
  COUNT(order_id) OVER() AS number_of_non_null_orders,
  COUNT(order_date) OVER() AS number_of_non_null_order_dates,
  ----------------------------------------
  -- total customer orders using partition by clause
  ----------------------------------------
  COUNT(order_id) OVER(
    PARTITION BY customer_id
  ) total_orders_per_customer,
  ----------------------------------------
  -- total orders per customer with order date less than or equal to row date
  ----------------------------------------
  COUNT(order_id) OVER(
    PARTITION BY customer_id
    ORDER BY order_date ASC
  ) rolling_total_orders_per_customer
FROM customer_orders
```

#### SUM()

-   calculates the total of a numeric column in a result set
-   can be used for metrics beyond simple totals (e.g. running total, moving window totals, conditional sum, etc)

``` sql
WITH sales_data(sale_id, employee_id, sale_amount, sale_date, discount_amount) AS (
  VALUES 
  (0, 101, 500, '2024-01-01'::DATE, 30),
  (1, 101, 200, '2024-01-01'::DATE, 20),
  (2, 102, 150, '2024-01-03'::DATE, NULL),
  (3, 101, 300, '2024-01-04'::DATE, 30),
  (4, 103, 250, '2024-01-04'::DATE, 25),
  (5, 102, 200, '2024-01-05'::DATE, NULL),
  (6, 104, 350, '2024-01-06'::DATE, 35),
  (7, 101, 400, '2024-01-07'::DATE, NULL),
  (8, 105, 150, '2024-01-08'::DATE, 15),
  (9, 104, 300, '2024-01-09'::DATE, 30),
  (10, 104, 100, '2024-01-10'::DATE, NULL)
)

SELECT
  *,
  ----------------------------------------
  -- overall totals of a column
  ----------------------------------------
  SUM(sale_amount) OVER() AS total_sales,
  SUM(discount_amount) OVER() AS total_discounts,
  ----------------------------------------
  -- rolling totals of a column
  ----------------------------------------
  SUM(sale_amount) OVER (
    ORDER BY sale_id, sale_date
  ) AS running_total_sales,
  ----------------------------------------
  -- total sales per day
  ----------------------------------------
  SUM(sale_amount) OVER(
    PARTITION BY sale_date
  ) total_sales_per_day,
  ----------------------------------------
  -- total sales per employee
  ----------------------------------------
  SUM(sale_amount) OVER(
    PARTITION BY employee_id
  ) total_sales_per_employee,
  ----------------------------------------
  -- running total sales per employee
  ----------------------------------------
  SUM(sale_amount) OVER(
    PARTITION BY employee_id
    ORDER BY sale_id, sale_date
  ) running_total_sales_per_employee,
  ----------------------------------------
  -- trailing 3 days sales total (including current row day)
  ----------------------------------------
  SUM(sale_amount) OVER (
    ORDER BY sale_date
    RANGE BETWEEN INTERVAL '2' DAY PRECEDING AND CURRENT ROW
  ) AS trailing_3_days_sales_total,
  ----------------------------------------
  -- additional examples using RANGE days window frame vs ROWS frame
  ----------------------------------------
  SUM(sale_amount) OVER (
    PARTITION BY employee_id
    ORDER BY sale_date
    RANGE BETWEEN INTERVAL '1' DAY PRECEDING AND INTERVAL '1' DAY PRECEDING
  ) AS employee_previous_day_sales,
  SUM(sale_amount) OVER (
    PARTITION BY employee_id
    ORDER BY sale_date
    RANGE BETWEEN INTERVAL '2' DAY PRECEDING AND INTERVAL '1' DAY PRECEDING
  ) AS employee_previous_2_day_sales,
  ----------------------------------------
  -- conditional logic using CASE statement within SUM() window function
  ----------------------------------------
  SUM(
      CASE 
        WHEN sale_date = '2024-01-01' 
        THEN sale_amount 
        ELSE 0 
      END
  ) OVER (
  ) AS jan_1_sales_total
FROM sales_data
```

### AVG()

-   derive the mean of a numeric column in a result set
-   useful for keeping granular data in a result set to compare vs a group level average
-   also useful for assessing trends via running/trailing averages

``` sql
WITH performance_data(review_id, employee_id, performance_score, review_date) AS (
  VALUES 
  (1, 101, 3.5, '2024-01-01'),
  (2, 102, 4.0, '2024-01-03'),
  (3, 101, 4.5, '2024-01-04'),
  (4, 103, 3.8, '2024-01-04'),
  (5, 102, 4.2, '2024-01-05'),
  (6, 104, 3.6, '2024-01-06'),
  (7, 101, 4.0, '2024-01-07'),
  (8, 105, 3.9, '2024-01-08'),
  (9, 104, 4.1, '2024-01-09'),
  (10, 103, 3.7, '2024-01-10')
)

SELECT
  *,
  ----------------------------------------
  -- avg performance score across all reviews
  ----------------------------------------
  AVG(performance_score) OVER() AS avg_score_all_reviews,
  ----------------------------------------
  -- avg performance score per employee
  ----------------------------------------
  AVG(performance_score) OVER(
    PARTITION BY employee_id
  ) AS avg_score_per_employee,
  ----------------------------------------
  -- rolling avg performance score per employee
  ----------------------------------------
  AVG(performance_score) OVER(
    PARTITION BY employee_id
    ORDER BY review_date
  ) AS rolling_avg_score_per_employee,
  ----------------------------------------
  -- avg performance score on a specific date
  ----------------------------------------
  AVG(performance_score) OVER(
    PARTITION BY review_date
  ) AS avg_score_on_date,
  ----------------------------------------
  -- cumulative avg up to current date
  ----------------------------------------
  AVG(performance_score) OVER(
    ORDER BY review_date
    ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
  ) AS overall_avg_score_to_date,
  ----------------------------------------
  -- avg performance score for the last three reviews
  ----------------------------------------
  AVG(performance_score) OVER(
    ORDER BY review_date
    ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
  ) AS avg_last_3_reviews,
  ----------------------------------------
  -- avg performance score for the next three reviews
  ----------------------------------------
  AVG(performance_score) OVER(
    ORDER BY review_date
    ROWS BETWEEN CURRENT ROW AND 2 FOLLOWING
  ) AS avg_next_3_reviews
FROM performance_data
```

#### ROW_NUMBER()

-   assigns a unique sequential integer to each row in the result set
-   often used for ranking and/or number ordering of rows in a result set
-   can be used to identify duplicate rows in a result set

``` sql
-- assume older posts always have smaller post_ids
WITH social_media_posts(post_id, user_id, likes, post_date) AS (
  VALUES 
  (1, 101, 120, '2024-01-01'),
  -- add duplicate row for row number example
  (1, 101, 120, '2024-01-01'),
  (2, 102, 150, '2024-01-02'),
  (3, 101, 200, '2024-01-02'),
  (4, 103, 180, '2024-01-03'),
  (5, 102, 140, '2024-01-04'),
  (6, 104, 160, '2024-01-04'),
  (7, 101, 190, '2024-01-04'),
  (8, 105, 110, '2024-01-05'),
  (9, 104, 130, '2024-01-05'),
  (10, 103, 175, '2024-01-06'),
  (11, 101, 165, '2024-01-06'),
  (12, 102, 155, '2024-01-07'),
  (13, 103, 145, '2024-01-07'),
  (14, 104, 135, '2024-01-08'),
  (15, 105, 125, '2024-01-08')
)

-- use row numbner to get drop duplicate rows
, clean_social_media_posts AS (
  SELECT
    post_id, 
    user_id, 
    likes, 
    post_date
  -- sub query to assign a unique sequential integer to each row
  FROM (
    SELECT
      *,
      ROW_NUMBER() OVER(
        PARTITION BY post_id, user_id, likes, post_date
        ORDER BY post_id
      ) AS row_num
    FROM social_media_posts
  ) AS sub_q
  -- grab first row for each unique combo of partition columns
  WHERE row_num = 1
)

SELECT
  *,
  ----------------------------------------
  -- user post order
  ----------------------------------------
  ROW_NUMBER() OVER(
    PARTITION BY user_id
    ORDER BY post_date ASC, 
             post_id ASC
  ) AS user_post_order,
  ----------------------------------------
  -- post ranking by likes (ties go to oldest post)
  ----------------------------------------
  ROW_NUMBER() OVER(
    ORDER BY likes DESC, 
             post_date ASC, 
             post_id ASC
  ) AS overall_post_rank_by_likes,
  ----------------------------------------
  -- daily post order
  ----------------------------------------
  ROW_NUMBER() OVER(
    PARTITION BY post_date
    ORDER BY post_id ASC
  ) AS post_order_by_day
FROM clean_social_media_posts
```

#### RANK()

-   assigns a rank to each row within a result set, with ties receiving the same rank
-   note that subsequent ranks get skipped based on number of tied entries
-   i.e. when three sales reps tie for rank 1 then the next rank assigned is 4

``` sql
DROP TABLE IF EXISTS temp_sales_data;
CREATE TEMP TABLE temp_sales_data (
    id SERIAL PRIMARY KEY,
    sales_rep VARCHAR(255),
    sales_amount NUMERIC(10, 2)
);

INSERT INTO temp_sales_data (sales_rep, sales_amount)
VALUES ('Alice', 25000),
       ('Bob', 30000),
       ('Carol', 45000),
       ('David', 60000),
       ('Eve', 55000),
       ('Frank', 70000),
       ('Grace', 100000),
       ('Hill', 100000),
       ('Tammy', 100000),
       ('Iris', 21000),
       ('Jan', 90000);

SELECT
    sales_rep,
    sales_amount,
    -- allow for same rank if ties
    RANK() OVER(ORDER BY sales_amount DESC) AS sales_rank
FROM temp_sales_data
ORDER BY sales_amount DESC, sales_rep
```

#### DENSE_RANK()

-   assigns a rank to each row within a result set
-   main difference between RANK and DENSE_RANK: DENSE_RANK ties receive the same rank without gaps between ranks
-   i.e. when three sales reps tie for rank 1 the next rank assigned is 2

``` sql
SELECT
    sales_rep,
    sales_amount,
    -- allow for same rank if ties
    RANK() OVER(ORDER BY sales_amount DESC) AS sales_rank,
    DENSE_RANK() OVER(ORDER BY sales_amount DESC) AS sales_dense_rank
FROM temp_sales_data
ORDER BY sales_amount DESC, sales_rep;
```

#### PERCENT_RANK() and CUME_DIST()

-   How PERCENT_RANK() vs CUME_DIST() differ
    -   Functions use differ ordering methods and differ in how duplicate values are handled
    -   PERCENT_RANK uses relative row position vs the actual row/column value
    -   CUME_DIST uses the row/column value to determine percentage
    -   Query output below highlights differences
    -   Depending on business use case one function may be more relevant than the other
-   PERCENT_RANK()
    -   returns a percentage value between 0 and 1 for the row position vs other rows in the result set
    -   (row rank−1)/(total rows in the partition−1)
    -   first row in the result set will always have a PERCENT_RANK of 0
    -   last row in the result set will always have a PERCENT_RANK of 1
-   CUME_DIST()
    -   returns proportion of rows with values less than or equal to the current row value (e.g. the cumulative distribution of a value in a set of values)

``` sql
WITH salary AS (
    SELECT unnest(ARRAY[50000, 50000, 55000, 60000, 65000, 
                        70000, 75000, 80000, 85000, 90000]) AS salary_dollars
)

SELECT
    salary_dollars,
    CUME_DIST() OVER(ORDER BY salary_dollars) AS cumulative_distribution_pct,
    PERCENT_RANK() OVER(ORDER BY salary_dollars) AS percent_rank_pct
FROM salary
```

#### NTILE(n)

-   divides the result set into n groups of roughly equal size and assigns a group number to each row
-   note how ordering is done within the partition/result set (i.e. depending on use case, should largest values be in first or last NTILE?)

``` sql
-- made up data
WITH fake_book_sales(book_id, book_title, copies_sold, publish_date) AS (
    VALUES
    (101, 'The Catcher in the Rye', 800, '1951-07-16'),
    (102, 'To Kill a Mockingbird', 1200, '1960-07-11'),
    (103, 'The Great Gatsby', 600, '1925-04-10'),
    (104, '1984', 900, '1949-06-08'),
    (105, 'Pride and Prejudice', 500, '1813-01-28'),
    (106, 'Animal Farm', 1000, '1945-08-17'),
    (107, 'Brave New World', 750, '1932-10-01'),
    (108, 'Moby-Dick', 450, '1851-10-18'),
    (109, 'Crime and Punishment', 550, '1866-01-01'),
    (110, 'Lord of the Flies', 700, '1954-09-17'),
    (111, 'A Raisin in the Sun', 1700, '1959-03-11'),
    (112, 'The Grapes of Wrath', 1000, '1939-04-14')
)

, perf AS (
SELECT
  *,
  -- smallest NTILE value for books with highest sales
  -- break ties with most recent books getting the lower ntile
  NTILE(4) OVER(
    ORDER BY copies_sold DESC, publish_date DESC
  ) AS performance_quartile
FROM fake_book_sales
)

SELECT
    book_id, 
    book_title, 
    copies_sold,
    -- tiers based on ntile ranking quartiles
    'tier_' || performance_quartile AS performance_tier
FROM perf
ORDER BY copies_sold DESC
```

#### LAG() \| LEAD()

-   LAG: returns column value of the row offset by N row(s) BEFORE the current row
-   LEAD: returns column value of the row offset by N row(s) AFTER the current row

``` sql
DROP TABLE IF EXISTS temp_example_watch_history;
CREATE TEMP TABLE temp_example_watch_history (
    user_id INT,
    user_name VARCHAR,
    show_id INT,
    show_title VARCHAR,
    first_watched_date DATE
);

INSERT INTO temp_example_watch_history(user_id, user_name, show_id, show_title, first_watched_date) VALUES
    -- user 1 example 
    (1, 'Alice', 101, 'Stranger Things', '2023-04-01'),
    (1, 'Alice', 102, 'The Crown', '2023-04-02'),
    (1, 'Alice', 103, 'The Witcher', '2023-04-03'),
    (1, 'Alice', 104, 'Money Heist', '2023-04-04'),
    -- user 2 example
    (2, 'Bob', 105, 'Breaking Bad', '2023-04-01'),
    (2, 'Bob', 106, 'The Queen''s Gambit', '2023-04-02'),
    (2, 'Bob', 107, 'Narcos', '2023-04-03'),
    (2, 'Bob', 108, 'Dark', '2023-04-04'),
    -- user 3 example
    (3, 'Charlie', 109, 'Ozark', '2023-04-01'),
    (3, 'Charlie', 110, 'The Mandalorian', '2023-04-02'),
    (3, 'Charlie', 111, 'The Boys', '2023-04-03'),
    (3, 'Charlie', 112, 'Fleabag', '2023-04-04'),
    -- user 4 example
    (4, 'Dave', 113, 'Killing Eve', '2023-04-01'),
    (4, 'Dave', 114, 'The Umbrella Academy', '2023-04-02'),
    (4, 'Dave', 115, 'Mindhunter', '2023-04-03'),
    (4, 'Dave', 116, 'The Haunting of Bly Manor', '2023-04-04'),
    -- user 5 example
    (5, 'Eve', 117, 'The Witcher', '2023-04-01'),
    (5, 'Eve', 118, 'Bridgerton', '2023-04-02'),
    (5, 'Eve', 119, 'The Expanse', '2023-04-03'),
    (5, 'Eve', 120, 'Schitt''s Creek', '2023-04-04');

-- concat show title and id to handle duplicate titles
-- not in the example data but possible in practice
SELECT
    *,
    COALESCE(
        LAG(show_title || '_' || show_id) OVER (
            PARTITION BY user_id 
            ORDER BY first_watched_date
        ),
        'First Show Watched_000'
    ) AS prev_show_watched,
    COALESCE(
        LEAD(show_title || '_' || show_id) OVER (
            PARTITION BY user_id 
            ORDER BY first_watched_date
        ),
        'Last Show Watched_000'
    ) AS next_show_watched
FROM temp_example_watch_history
ORDER BY user_id, first_watched_date
```

#### FIRST_VALUE() \| LAST_VALUE(expr) \| NTH_VALUE()

-   FIRST_VALUE: returns the first value in an ordered set of values
-   LAST_VALUE: returns the last value in an ordered set of values
-   NTH_VALUE: returns the Nth value in an ordered set of values

``` sql
SELECT
    *,
    FIRST_VALUE(show_title || '_' || show_id) OVER(
        PARTITION BY user_id 
        ORDER BY first_watched_date 
        ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
    ) AS first_show_watched,
    NTH_VALUE(show_title || '_' || show_id, 2) OVER(
        PARTITION BY user_id 
        ORDER BY first_watched_date
        ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
    ) AS second_show_watched,
    NTH_VALUE(show_title || '_' || show_id, 3) OVER(
        PARTITION BY user_id 
        ORDER BY first_watched_date
        ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
    ) AS third_show_watched,
    LAST_VALUE(show_title || '_' || show_id) OVER(
        PARTITION BY user_id 
        ORDER BY first_watched_date
        ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
    ) AS last_show_watched
-- temp table generate above
FROM temp_example_watch_history
```

#### MIN and MAX

-   MIN: find minimum value in a partition / result set
-   MAX: find maximum value in a partition / result set
-   example below using MIN and MAX window functions for min-max normalization

``` sql
WITH fake_stocks(stock_name, date, closing_price) AS (
    VALUES
    ('NetGrid', '2023-01-01',  98.50),
    ('NetGrid', '2023-01-02', 100.75),
    ('NetGrid', '2023-01-03',  99.20),
    ('NetGrid', '2023-01-04',  97.80),
    ('NetGrid', '2023-01-05', 101.00),
    ('ByteTech', '2023-01-01', 210.40),
    ('ByteTech', '2023-01-02', 215.55),
    ('ByteTech', '2023-01-03', 212.30),
    ('ByteTech', '2023-01-04', 214.00),
    ('ByteTech', '2023-01-05', 216.75),
    ('QuantumCore', '2023-01-01', 550.00),
    ('QuantumCore', '2023-01-02', 560.25),
    ('QuantumCore', '2023-01-03', 555.50),
    ('QuantumCore', '2023-01-04', 558.75),
    ('QuantumCore', '2023-01-05', 563.00),
    ('SkyData', '2023-01-01', 1020.50),
    ('SkyData', '2023-01-02', 1035.00),
    ('SkyData', '2023-01-03', 1025.75),
    ('SkyData', '2023-01-04', 1040.30),
    ('SkyData', '2023-01-05', 1038.80),
    ('GlobalLink', '2023-01-01', 3050.00),
    ('GlobalLink', '2023-01-02', 3080.25),
    ('GlobalLink', '2023-01-03', 3065.75),
    ('GlobalLink', '2023-01-04', 3075.30),
    ('GlobalLink', '2023-01-05', 3090.80)
)

, setup_stock_price_ranges AS (
    SELECT
        *,
        MIN(closing_price) OVER(PARTITION BY stock_name) as min_price,
        MAX(closing_price) OVER(PARTITION BY stock_name) as max_price
    FROM fake_stocks
)

SELECT
    *,
    (closing_price - min_price)::FLOAT / (max_price - min_price) AS normalized_price
FROM setup_stock_price_ranges
```

## Practical Use Cases

#### Percent of total

-   approach with PostgreSQL
-   database vendors have slightly different approaches (i.e. Redshift has RATIO_TO_REPORT() which can be used for pct total calcs)

``` sql
WITH example_data(item, amount) AS (
  VALUES
    ('Item A', 50),
    ('Item B', 100),
    ('Item C', 150),
    ('Item D', 200),
    ('Item E', 13)
)

SELECT 
    item,
    amount, 
    ROUND((100.0 * amount / SUM(amount) OVER()),2) AS percent_total 
FROM example_data
ORDER BY percent_total DESC
```

#### Cumulative percent total using window functions

-   rolling sum window function compared to total aggregation

``` sql
WITH salary AS (
    SELECT unnest(ARRAY[50000, 55000, 60000, 65000, 70000, 
                        75000, 80000, 85000, 90000, 950000]) AS salary_dollars
)

SELECT
    salary_dollars,
    SUM(salary_dollars) OVER (
        ORDER BY salary_dollars
    )::FLOAT / SUM(salary_dollars) OVER () * 100 AS cumulative_percent_total
FROM salary
ORDER BY salary_dollars
```

#### Quartiles with lower and upper bound values

-   approach to provide additional context when generating quartiles

``` sql
-- source: https://www.statmuse.com/nfl/ask/nfl-qb-total-touchdown-leaders-2022
WITH top_20_quarterbacks(player_name, team, total_td)
VALUES ('Patrick Mahomes', 'KC', 45),
       ('Josh Allen', 'BUF', 42),
       ('Joe Burrow', 'CIN', 40),
       ('Jalen Hurts', 'PHI', 35),
       ('Kirk Cousins', 'MIN', 31),
       ('Geno Smith', 'SEA', 31),
       ('Trevor Lawrence', 'JAX', 30),
       ('Jared Goff', 'DET', 29),
       ('Aaron Rodgers', 'GB', 27),
       ('Tom Brady', 'TB', 26),
       ('Justin Herbert', 'LAC', 25),
       ('Tua Tagovailoa', 'MIA', 25),
       ('Justin Fields', 'CHI', 25),
       ('Derek Carr', 'LV', 24),
       ('Dak Prescott', 'DAL', 24),
       ('Daniel Jones', 'NYG', 22),
       ('Lamar Jackson', 'BAL', 20),
       ('Russell Wilson', 'DEN', 19),
       ('Marcus Mariota', 'ATL', 19),
       ('Davis Mills', 'HOU', 19);

, ntile_setup AS (
SELECT
    *,
    NTILE(4) OVER(ORDER BY total_td DESC) AS quartile
FROM top_20_quarterbacks
)

SELECT
    *,
    MIN(total_td) OVER(
        PARTITION BY quartile
    ) AS quartile_lower_bound,
    MAX(total_td) OVER(
        PARTITION BY quartile
    ) AS quartile_upper_bound
FROM ntile_setup
ORDER BY total_td DESC, player_name
```

#### Moving Average

-   moving batting average across last 3 seasons

``` sql
-- first 20 seasons stats for Barry Bonds
-- https://www.baseball-reference.com/players/b/bondsba01.shtml
WITH barry_bonds(season, batting_average) AS (
VALUES
  (1986, 0.223),
  (1987, 0.261),
  (1988, 0.283),
  (1989, 0.248),
  (1990, 0.301),
  (1991, 0.292),
  (1992, 0.311),
  (1993, 0.336),
  (1994, 0.312),
  (1995, 0.294),
  (1996, 0.308),
  (1997, 0.291),
  (1998, 0.303),
  (1999, 0.262),
  (2000, 0.306),
  (2001, 0.328),
  (2002, 0.370),
  (2003, 0.341),
  (2004, 0.362),
  (2005, 0.286)
)

SELECT
    season,
    ROUND(batting_average,3) AS batting_average,
    -- logic to return NULL when fewer than 3 seasons
    CASE 
        WHEN season_order >= 3 
        THEN ROUND(batting_avg_moving_average_3::NUMERIC,3)
        ELSE NULL
    END AS batting_avg_moving_average_3_seasons 
FROM (
    SELECT
        *,
        ROW_NUMBER() OVER(
            ORDER BY season ASC
        ) AS season_order,
        AVG(batting_average::FLOAT) OVER (
            ORDER BY season ASC
            ROWS BETWEEN 2 PRECEDING and CURRENT ROW
        ) AS batting_avg_moving_average_3
    FROM barry_bonds
) AS sub_q
```

#### New yearly revenue record

-   find the rolling max revenue for prior years and compare to current row year

``` sql
-- to get reproducible results
SELECT SETSEED(0.25);

WITH yearly_revenue(year, revenue) AS (
VALUES
  (1992, RANDOM() * 1000000.00),
  (1993, RANDOM() * 1200000.00),
  (1994, RANDOM() * 1500000.00),
  (1995, RANDOM() * 2000000.00),
  (1996, RANDOM() * 2500000.00),
  (1997, RANDOM() * 3000000.00),
  (1998, RANDOM() * 3500000.00),
  (1999, RANDOM() * 4000000.00),
  (2000, RANDOM() * 4500000.00),
  (2001, RANDOM() * 5000000.00),
  (2002, RANDOM() * 5500000.00),
  (2003, RANDOM() * 6000000.00),
  (2004, RANDOM() * 6500000.00),
  (2005, RANDOM() * 7000000.00),
  (2006, RANDOM() * 7500000.00),
  (2007, RANDOM() * 8000000.00),
  (2008, RANDOM() * 8500000.00),
  (2009, RANDOM() * 9000000.00),
  (2010, RANDOM() * 9500000.00)
)

WITH yearly_revenue_setup AS (
SELECT
    year,
    revenue,
    MAX(revenue) OVER(
        ORDER BY year ASC
        ROWS BETWEEN UNBOUNDED PRECEDING AND 1 PRECEDING
    ) AS previous_years_revenue_all_time_high
FROM temp_revenue
)

SELECT
    *,
    CASE 
        WHEN revenue > previous_years_revenue_all_time_high
        THEN TRUE
        ELSE FALSE
    END AS new_all_time_revenue_high_flag
FROM yearly_revenue_setup
```

#### Last 7 Day Total Sales

-   note use of frame logic using RANGE BETWEEN which is conditional on date vs row position

``` sql
-- to get reproducible results
SELECT SETSEED(0.1);

DROP TABLE IF EXISTS temp_daily_sales;
CREATE TEMP TABLE temp_daily_sales AS
SELECT 
    sales_day,
    ROUND((RANDOM()::NUMERIC * 1000),2) AS sale_amount
FROM GENERATE_SERIES(
        '2022-01-01'::DATE, 
        '2022-04-04'::DATE, 
        '1 day'::interval
    ) AS sales_day;

-- randomly drop rows to create intentional date gaps
DELETE FROM temp_daily_sales
WHERE RANDOM() < 0.3;

SELECT
    *,
    SUM(sale_amount) OVER (
      ORDER BY sales_day ASC 
        -- handles use case when there are date gaps
        RANGE BETWEEN INTERVAL '6 days' PRECEDING AND CURRENT ROW
   ) AS last_7_days_sales   
FROM temp_daily_sales
```

#### Store Sales Performance

-   using multiple window functions: ranking, ntiles, running total, etc

``` sql
DROP TABLE IF EXISTS temp_store_sales_2022;
CREATE TEMP TABLE temp_store_sales_2022 (
  location_id INTEGER,
  sale_year INTEGER,
  total_sales NUMERIC(10,2)
);

SELECT SETSEED(0.5);

INSERT INTO temp_store_sales_2022 (location_id, sale_year, total_sales)
SELECT 
  location_id,
  2022::INT AS sales_year,
  ROUND(RANDOM()::NUMERIC * 100000, 2) AS total_sales
FROM generate_series(1, 40) AS location_id;
  
SELECT 
    *,
    total_sales / SUM(total_sales) OVER () AS percent_total_sales,
    -- dense rank to allow for locations to have same rank if sales amount is equal
    DENSE_RANK() OVER(
        ORDER BY total_sales DESC
    ) AS location_sales_rank,
    NTILE(10) OVER(
        ORDER BY total_sales DESC
    ) AS location_sales_decile,
    SUM(total_sales) OVER(
        ORDER BY total_sales DESC
        ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
    ) AS top_location_to_bottom_location_running_total,
    (
        SUM(total_sales) OVER (
            ORDER BY total_sales DESC
            ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
        ) / 
        SUM(total_sales) OVER ()
    ) * 100 AS cumulative_percent_total
FROM temp_store_sales_2022
ORDER BY total_sales DESC
```

#### YoY sales growth using window function

-   note use of lag function to get prior year sales
-   could also be solved using a self join

``` sql
WITH sales AS (
    SELECT 
        UNNEST(ARRAY[23456, 43567, 65812, 79234, 567394]) AS annual_sales,
        UNNEST(ARRAY[1999, 2000, 2001, 2002, 2003]) AS year
)

-- NULLs expected for first year prior year sales and YoY growth
SELECT
  'window_fun_results' AS label,
    year,
  annual_sales,
    LAG(annual_sales) OVER(ORDER BY year ASC) AS prior_year_sales,
    ((annual_sales - LAG(annual_sales) OVER(ORDER BY year ASC)::FLOAT) 
       / LAG(annual_sales) OVER(ORDER BY year ASC)) * 100 AS yoy_sales_growth
FROM sales

UNION ALL 

SELECT 
    'self_join_results' AS label,
    current_year.year,
    current_year.annual_sales,
    prior_year.annual_sales AS prior_year_sales,
    ((current_year.annual_sales - prior_year.annual_sales)::FLOAT
        / prior_year.annual_sales) * 100 AS yoy_sales_growth
FROM sales AS current_year
LEFT JOIN sales AS prior_year
    ON prior_year.year = (current_year.year - 1)
```

#### Sales by month and add YTD sales column

-   note use of partition by year and frame boundary

``` sql
WITH sales_data(sales_month, sales) AS (
    VALUES
    ('2022-10-01'::date, 1000),
    ('2022-11-01'::date, 1200),
    ('2022-12-01'::date, 1400),
    ('2023-01-01'::date, 1100)
)

SELECT
    sales_month,
    sales,
    SUM(sales) OVER (
    -- note partition by year
        PARTITION BY EXTRACT(year FROM sales_month)
        ORDER BY sales_month
    -- frame default with order by; being explicit for readability
        ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
    ) AS ytd_sales
FROM sales_data
ORDER BY sales_month
```

#### Sales by month output with prior year sales column for visual comparison

-   leverage lag function to get prior year sales; note partition spec

``` sql
-- set seed so sales amount is consistent on query reruns
SELECT SETSEED(0.99);

-- generates sequence of 24 months
WITH months AS (
        SELECT
            generate_series(
                date_trunc('month', '2024-01-01'::DATE - INTERVAL '24 months'),
                '2024-01-01'::DATE - INTERVAL '1 month',
                '1 month'
            )::DATE AS sales_month
)

-- assign random sales amount to each month
, example_sales_data AS (
    SELECT
      sales_month,
      ROUND((RANDOM() * 10000)::NUMERIC, 2) AS sales_amount,
      MIN(sales_month) OVER() AS first_sales_month
    FROM months
)

-- sub query to do where clause filter after window function compute
SELECT
    sales_month,
    sales_amount,
    prior_year_sales
FROM (
    SELECT
        *,
        LAG(sales_amount) OVER(
            PARTITION BY EXTRACT(month FROM sales_month)
            ORDER BY sales_month
        ) AS prior_year_sales
    FROM example_sales_data
    ORDER BY sales_month
) AS sub_q
-- returns rows where we have prior year sales
WHERE sales_month >= first_sales_month + INTERVAL '12 months'
```

#### Workaround for COUNT DISTINCT window function use case

-   using ROW_NUMBER() and a conditional SUM()

``` sql
WITH sales_data(transaction_id, customer_id, transaction_date, amount) AS (
    VALUES
    (1, 'C001', '2024-01-10'::DATE, 100),
    (2, 'C001', '2024-02-01'::DATE, 150),
    (3, 'C002', '2024-02-04'::DATE, 200),
    (4, 'C002', '2024-02-18'::DATE, 300),
    (5, 'C003', '2024-03-03'::DATE, 250),
    (6, 'C003', '2024-03-05'::DATE, 350),
    (7, 'C003', '2024-03-15'::DATE, 400),
    (8, 'C004', '2024-04-10'::DATE, 500),
    (9, 'C004', '2024-04-15'::DATE, 550),
    (10, 'C001', '2024-04-20'::DATE, 600)
)

, sales_data_setup AS (
    SELECT
        *,
        ROW_NUMBER() OVER(
            PARTITION BY customer_id, DATE_TRUNC('month', transaction_date)
            ORDER BY transaction_date ASC
        ) AS by_month_customer_purchase_order
    FROM sales_data
)

SELECT
    *,
    AVG(amount) OVER (PARTITION BY customer_id) AS avg_amount_per_customer,
    SUM(amount) OVER (PARTITION BY customer_id) AS total_amount_per_customer,
    COUNT(*) OVER (PARTITION BY customer_id) AS transaction_count_per_customer,
    -- count unique purchase months by customer
    SUM(
        CASE 
            WHEN by_month_customer_purchase_order = 1
            THEN 1
            ELSE 0
        END
    ) OVER(
        PARTITION BY customer_id
    ) AS unqiue_months_with_purchases_per_customer
FROM sales_data_setup
ORDER BY transaction_id
```

#### Handling NULLs in window functions

-   given the nuances, worth building test cases to validate results for the dataset one is working with
-   argument input:
    -   SQL standard includes a RESPECT NULLS or IGNORE NULLS clause for certain window functions
    -   IGNORE NULLS not implemented in PostgreSQL; often included in other DB vendor SQL (i.e. Redshift, etc)
    -   NULL treatment is specific to the function use case (i.e. AVG(column_x) ignores NULLs vs COUNT(\*) includes NULLs)
-   partition:
    -   NULLs included in a partition column are treated as a single group
-   order by:
    -   by default, NULLs are sorted last in ascending order and first in descending order
    -   override default behavior with NULLS FIRST or NULLS LAST
-   window frame:
    -   NULLs tend to be treated as normal rows
    -   however, results can differ for ROWS, RANGE, and GROUPS frame types (worth validating expected behavior)

``` sql
WITH social_media_posts(post_id, user_id, likes, post_date) AS (
  VALUES 
  (1, 101, NULL, '2024-01-01'),
  (2, 102, 150, '2024-01-02'),
  (3, 101, 200, '2024-01-02'),
  (4, 103, NULL, '2024-01-03'),
  (5, 102, 140, '2024-01-04'),
  (6, 104, 160, '2024-01-04'),
  (7, 101, 190, '2024-01-04'),
  (8, 105, 110, '2024-01-05'),
  (9, 104, 130, '2024-01-05'),
  (10, 103, 175, '2024-01-06'),
  (11, 101, 165, '2024-01-06'),
  (12, 102, 155, '2024-01-07'),
  (13, 103, 145, '2024-01-07'),
  (14, 104, 130, '2024-01-08'),
  (15, 105, 125, '2024-01-08')
)

SELECT
    *,
    DENSE_RANK() OVER(
        PARTITION BY user_id
        -- force NULLs to sort at end vs default DESC NULLS first
        ORDER BY likes DESC NULLS LAST
        -- could also use
        -- ORDER BY COALESCE(likes, 0) DESC
    ) AS user_post_likes_rank
FROM social_media_posts
```

#### Filtering window function results

-   both WHERE and HAVING clause filtering occurs before window function logic is applied
-   HAVING clause filtering primarily used to filter results of an aggregate function
-   subquery and CTEs tend to be used to filter window function results
-   some DB vendors like Amazon Redshift support inline window function filtering ([see QUALIFY clause](https://docs.aws.amazon.com/redshift/latest/dg/r_QUALIFY_clause.html))

``` sql
WITH sales_table(region, sales) AS (
  VALUES 
  ('North', 6000),
  ('North', 5000),
  ('South', 7000),
  ('South', 3000),
  ('East', 4000),
  ('East', 4500),
  ('West', 8000),
  ('West', 2000)
)
, sales_avg AS (
  SELECT region, 
         AVG(sales::FLOAT) OVER (
            PARTITION BY region 
            ORDER BY sales DESC
          ) AS top_sale_avg
  FROM sales_table
)

-- typical window function result filtering pattern
SELECT 
    region, top_sale_avg
FROM sales_avg
WHERE top_sale_avg > 5000
```
