#### Working with NULL values

-   NULL in SQL means no value or unknown value
-   NULL is different from zero or an empty string
-   NULLs can arise due to missing data, data errors, incorrect joins, etc

#### Check columns for NULL values

``` sql
WITH employee_ids(id, department, salary) AS (
    VALUES
    (1, NULL, 50000),
    (2, 'HR', 60000),
    (3, 'Sales', 70000),
    (4, 'IT', 65000),
    (5, 'Marketing', 75000),
    (6, 'Marketing', NULL),
    (7, 'Admin', NULL)
)

SELECT
    COUNT(*) - COUNT(id) AS id_nulls,
    COUNT(*) - COUNT(department) AS department_nulls,
    COUNT(*) - COUNT(salary) AS salary_nulls
FROM employee_ids
```

#### NULLs in arithmetic operations or comparisons

-   arithmetic operations with NULL will result in NULL
-   comparison operation with NULL will result in NULL
-   why? NULL is considered unknown in SQL so we can't definitively do arithmetic or comparison.
-   NULL is not equal to anything, including itself (e.g. NULL is not equal to NULL)

``` sql
SELECT
    1 + NULL AS test1, -- returns NULL 
    1 + NULL AS test2, -- returns NULL,
    NULL = NULL AS test3, -- returns NULL
    NULL IS NULL  AS test4, -- returns TRUE,
    1 IS NULL AS test5 -- returns FALSE
```

#### Filtering for NULLs in WHERE clause

-   use `IS NULL` vs `= NULL`

``` sql
WITH employees(name, department, salary) AS (
    VALUES
    ('John', 'HR', 50000),
    ('Sarah', 'HR', 60000),
    ('Steve', 'Sales', 70000),
    ('Jake', 'Sales', 80000),
    ('Laura', 'IT', 55000),
    ('Mike', 'IT', 65000),
    ('Emily', 'Marketing', 75000),
    ('Paul', 'Marketing', 85000),
    ('Tim', 'Admin', NULL)
)

-- zero rows returned because = comparison with NULL returns NULL
-- does not work as intended; use IS NULL instead
SELECT 
    *
FROM employees
WHERE salary = NULL

-- returns 1 row for Tim
SELECT 
    *
FROM employees
WHERE salary IS NULL
```

#### NULLs in aggregations

-   when a column is specified in an aggregate function NULLs are ***typically*** ignored
-   COALESCE can be used to set a default value for NULLs
-   `COUNT(*)`: return a row count of NULL and non NULL values
-   `COUNT(column_name)`: return a row count of non NULL values

``` sql
WITH year_1_minutes_watched(user_id, minutes_watched) AS (
    VALUES
    (1, 100),
    (2, 200),
    (3, 300),
    (4, 400),
    (5, 500),
    (6, NULL),
    (7, 150),
    (8, 250),
    (9, NULL),
    (10, 350)
)

SELECT
    COUNT(user_id) AS total_users,
    COUNT(minutes_watched) AS users_with_minutes_watched,
    AVG(minutes_watched::FLOAT) AS avg_minutes_watched_consumers_only,
    AVG(COALESCE(minutes_watched, 0)::FLOAT) AS avg_minutes_watched_including_non_consumers
FROM year_1_minutes_watched
```

#### NULL value in join results

-   NULLs can show up in outer joins when values are not found in the "lookup" table
-   NULLs cannot be used as join keys because a NULL is not equal to anything even itself

``` sql
WITH table1(id, value) AS (
    VALUES
    (1, 'A'),
    (2, 'B'),
    (NULL, 'C')
)
, table2(id, value) AS (
    VALUES
    (1, 'X'),
    (3, 'Y'),
    (NULL, 'Z')
)

-- returns 1 row for id 1 that has non null match on id in both tables
SELECT *
FROM table1
INNER JOIN table2 
    ON table1.id = table2.id

-- returns the 3 rows from table 1. however, only id 1 includes values from table 2
-- NULL values do not qualify in the join logic due to NULLs representing unknown values
SELECT *
FROM table1
LEFT JOIN table2 
    ON table1.id = table2.id
```

#### NULLs in Order By

-   By default, nulls are considered the highest value so when ordering ASC nulls appear at the end of the order

``` sql
WITH year_1_minutes_watched(user_id, year, minutes_watched) AS (
    VALUES
    (1, 1, 100),
    (2, 1, 200),
    (3, 1, 300),
    (4, 1, 400),
    (5, 1, 500),
    (6, 1, NULL),
    (7, 1, 150),
    (8, 1, 250),
    (9, 1, NULL),
    (10, 1, 350)
)

-- will order the 'minutes_watched' column in ascending order, with NULLs first
SELECT 
    user_id, 
    minutes_watched 
FROM year_1_minutes_watched
ORDER BY minutes_watched ASC NULLS FIRST

-- will order the 'minutes_watched' column in descending order, with NULLs last
SELECT 
    user_id, 
    minutes_watched 
FROM year_1_minutes_watched
ORDER BY minutes_watched DESC NULLS LAST
```

#### NULLs in String Concatenation

-   `||` operator for string concatenation : inclusion of NULL value results in NULL output.
-   `CONCAT` function: treats NULL values as empty strings.

``` sql
SELECT 
  'abc' || NULL AS test1, -- returns NULL
  'abc' || COALESCE(NULL, 'efg') AS test2, -- returns abcefg
  'abc' || COALESCE(NULL, '') AS test3, -- returns abc; note COALESCE with empty string
  'abc' || NULL || 'efg' AS test4, -- returns NULL
  CONCAT('abc', NULL) AS test5 -- returns abc
```

#### COALESCE

-   return first non NULL input

``` sql
SELECT COALESCE(NULL, NULL, NULL, 1) AS returns_first_non_null_input
```

#### NULLIF

-   use `NULLIF` to set NULL value when condition is true

``` sql
SELECT
  ------------------------------
  -- avoid division by zero errors; return NULL instead of error
  130 / NULLIF(0, 0) AS test0,
  ------------------------------
  -- empty strings to NULL
  NULLIF('', '') AS test1,
  ------------------------------
  -- set default date to NULL
  NULLIF('2000-01-01'::DATE, '2000-01-01'::DATE) AS test2,
  ------------------------------
  -- replace 0 values with default value (could default value via subquery)
  COALESCE(NULLIF(0, 0), 100) AS test3,
  ------------------------------
  -- set outlier/error values to NULL
  NULLIF(-999, -999) AS test4,
  ------------------------------
  -- covert NA string to NULL
  NULLIF('N/A', 'N/A') AS test5
```

#### Window functions with NULLs

-   NULL ordering in window functions (by default, NULLs appear as highest values)
-   `RESPECT NULLS` and `IGNORE NULLS` included in the SQL standard; not available in Postgres
-   ignoring NULLs with ordered PostgresSQL window functions often requires workaround solution (example below)

``` sql
WITH sales_data(salesperson, sales_amount) AS (
    VALUES
    ('Alice', 100),
    ('Bob', NULL),
    ('Charlie', 200),
    ('Diana', NULL),
    ('Eve', 150),
    ('Frank', 300),
    ('Grace', NULL),
    ('Helen', 200),
    ('Ian', 350),
    ('Jane', NULL),
    ('Barry', 350)
)

-- case when logic used to return sales rank only for folks with sales amount
SELECT 
    salesperson,
    sales_amount,
    CASE
        WHEN sales_amount IS NULL THEN NULL
        -- dense rank so folks with ties get same rank (i.e. Barry and Ian both 1) 
        ELSE DENSE_RANK() OVER (ORDER BY sales_amount DESC NULLS LAST)
    END AS sales_rank
FROM sales_data
```

#### Table schemas and row inserts with NULL

-   columns with NOT NULL constraint:
    -   prevents NULL from being added to the table column (returns error)
-   columns with DEFAULT values:
    -   if no value provided then DEFAULT value returned
    -   if explicit NULL provided then NULL returned
    -   if DEFAULT keyword provided then DEFAULT value returned
-   columns without NOT NULL constraint or DEFAULT values:
    -   if no value provided for a column then NULL returned
    -   if NULL explicitly given then NULL returned

``` sql
-- DROP TABLE IF EXISTS temp_example_table;
CREATE TEMP TABLE temp_example_table (
    id INT PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    age INT DEFAULT 25, -- Default value for age is 25
    join_date DATE
);

----------------------------
-- Inserting rows into the temporary table examples
----------------------------
INSERT INTO temp_example_table (id, name, age, join_date) VALUES (1, 'Alice', NULL, NULL);
INSERT INTO temp_example_table (id, name, age) VALUES (2, 'Bob', 30);
INSERT INTO temp_example_table (id, name) VALUES (3, 'Charlie');
INSERT INTO temp_example_table (id, name, join_date) VALUES (4, 'Charlie', '2023-01-01');
INSERT INTO temp_example_table (id, name, age) VALUES (5, 'Eve', DEFAULT);

-- Error expected given null constraint on name column
-- INSERT INTO temp_example_table (id) VALUES (6);

-- Optional: Query to view the inserted data
SELECT * FROM temp_example_table;
```
