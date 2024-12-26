### Reminders

-   below is a recap of key timestamp/date functions in PostgreSQL (database vendors expected to have slightly different syntax)
-   future todo: how to better format code; long horizontal lines are not ideal and wrapping making readability tricky

#### EXTRACT

-   extract parts of a date/time value

```         
WITH ts_example AS (
    SELECT 
        TO_TIMESTAMP('2021-03-13 09:30:03', 'YYYY-MM-DD HH:MI:SS') AS timestamp_1
)

SELECT
    EXTRACT(YEAR FROM timestamp_1) AS year,
    EXTRACT(QUARTER FROM timestamp_1) AS quarter,
    EXTRACT(MONTH FROM timestamp_1) AS month,
    EXTRACT(DOY FROM timestamp_1) AS day_of_year,
    EXTRACT(DAY FROM timestamp_1) AS day_of_month,
    EXTRACT(DOW FROM timestamp_1) AS day_of_week,
    EXTRACT(HOUR FROM timestamp_1) AS ts_hour,
    EXTRACT(MINUTE FROM timestamp_1) AS ts_minute,
    EXTRACT(SECOND FROM timestamp_1) AS ts_second
FROM ts_example
```

#### EXTRACT vs DATE_PART

-   slight differences in DATE_PART vs EXTRACT; often used interchangeably
-   some nuance: DATE_PART can grab fractional values with decimal points vs EXTRACT rounds down to nearest integer

```         
WITH ts_example_2 AS (
    SELECT 
        TO_TIMESTAMP('2022-07-13 11:20:18:06', 'YYYY-MM-DD HH:MI:SS::MS') AS timestamp_2
)

SELECT
    DATE_PART('year', timestamp_2) AS dp_year,
    DATE_PART('quarter', timestamp_2) AS dp_quarter,
    DATE_PART('month', timestamp_2) AS dp_month,
    DATE_PART('day', timestamp_2) AS dp_day,
    DATE_PART('hour', timestamp_2) AS dp_hour,
    DATE_PART('minute', timestamp_2) AS dp_minute,
    -- fractional seconds example 
    DATE_PART('seconds', timestamp_2) AS dp_second
FROM ts_example_2
```

#### DATE_TRUNC

-   similar to FLOOR function but for a date/time and specific unit of time (year, quarter, month, day, hour, minute, etc)
-   commonly used to aggregate by date dimension (i.e. sales by month/quarter)

```         
WITH ts_example_3 AS (
    SELECT 
        TO_TIMESTAMP('2022-05-22 07:30:03', 'YYYY-MM-DD HH:MI:SS') AS timestamp_3
)

SELECT
    *,
    DATE_TRUNC('year', timestamp_3)::DATE AS year,
    DATE_TRUNC('quarter', timestamp_3)::DATE AS quarter,
    DATE_TRUNC('month', timestamp_3)::DATE AS month,
    -- simpler syntax for day
    timestamp_3::DATE AS day,
    DATE_TRUNC('hour', timestamp_3) AS hour
FROM ts_example_3
```

#### Date/Timestamp subtraction

Two main methods (subtracting two dates/timestamps or using AGE function)

1.  Subtracting two dates/timestamps
    -   timestamps: returns an interval data type down to the smallest unit (starting at days when relevant)
    -   dates: returns number of calendar days
2.  AGE function
    -   calculates differences between two given dates/timestamps or one given date/timestamp and current date/timestamp
    -   works kind of like DATEDIFF in other databases
    -   returns interval data type (starting at years when relevant)
    -   helpful to see difference in human readable form starting in years
    -   for complex calcs, similar workarounds needed vs simple date/timestamp subtraction

```         
-- depends on use case for when subtracting dates/timestamps vs AGE function would be more streamlined logic
WITH practice_timestamps AS (
    SELECT 
        TO_TIMESTAMP('2022-04-30 07:30:03', 'YYYY-MM-DD HH:MI:SS') AS home_page_first_visit,
        TO_TIMESTAMP('2022-05-23 10:01:12', 'YYYY-MM-DD HH:MI:SS') AS purchased_at,
        TO_TIMESTAMP('2022-05-23 10:01:12', 'YYYY-MM-DD HH:MI:SS') + INTERVAL '1 YEAR' AS year_1_renewed_at
)

-- why use this CTE? 
-- PostgreSQL has limitations on referencing columns previously specified in a query
-- less verbose than including logic above
, include_intervals AS (
    SELECT
      *,
      -- date/timestamp subtraction
      purchased_at::DATE - home_page_first_visit::DATE AS days_from_home_first_visit_to_pur,
      purchased_at - home_page_first_visit AS ts_sub_interval_pur_vs_home_first_vis,
      year_1_renewed_at - purchased_at AS ts_sub_interval_renew_vs_pur,
      -- AGE function subtraction
      AGE(purchased_at, home_page_first_visit) AS age_interval_pur_vs_home_first_vis,
      AGE(year_1_renewed_at, purchased_at) AS age_interval_renew_vs_pur
    FROM practice_timestamps
)

-- example use cases using subtraction internal and age interval
-- note logic to properly handle interval components for addition
SELECT
  *,
  -- date subtraction days and timestamp subtraction interval
  (days_from_home_first_visit_to_pur * 24) + 
    EXTRACT('hour' FROM ts_sub_interval_pur_vs_home_first_vis) AS hours_from_home_first_vis_to_pur,
  -- using AGE interval
  (EXTRACT('year' FROM age_interval_renew_vs_pur) * 12) + 
    EXTRACT('month' FROM age_interval_renew_vs_pur) AS months_from_pur_to_renew
FROM include_intervals
```

#### EPOCH

-   used to convert date, timestamp, or interval value to seconds elapsed
-   i.e. uniform units used when differences can span days vs hours
-   concept of "epoch" comes from Unix; representing time elapsed since the epoch

```         
-- similar to previous example but using EPOCH certain use cases
WITH practice_timestamps AS (
    SELECT 
        TO_TIMESTAMP('2022-04-30 07:30:03', 'YYYY-MM-DD HH:MI:SS') AS home_page_first_visit,
        TO_TIMESTAMP('2022-05-23 10:01:12', 'YYYY-MM-DD HH:MI:SS') AS purchased_at,
        TO_TIMESTAMP('2022-05-27 11:07:15', 'YYYY-MM-DD HH:MI:SS') AS core_action_first_completed_at,
        -- TO_TIMESTAMP('2022-06-03 05:07:15', 'YYYY-MM-DD HH:MI:SS') AS core_action_first_completed_at,
        TO_TIMESTAMP('2022-05-23 10:01:12', 'YYYY-MM-DD HH:MI:SS') + INTERVAL '1 YEAR' AS year_1_renewed_at
)

, include_intervals AS (
    SELECT
      *,
      purchased_at::DATE - home_page_first_visit::DATE AS cal_days_from_home_first_visit_to_pur,
      purchased_at - home_page_first_visit AS ts_sub_interval_pur_vs_home_first_vis,
      core_action_first_completed_at::DATE - purchased_at::DATE AS cal_days_from_pur_to_core_action,  
      core_action_first_completed_at - purchased_at AS ts_sub_interval_core_action_vs_pur,  
      year_1_renewed_at - purchased_at AS ts_sub_interval_renew_vs_pur
    FROM practice_timestamps
)

-- 24 hour interval = a subscription day
-- 30 day interval = a subscription month
SELECT
  *,
  EXTRACT(EPOCH FROM ts_sub_interval_pur_vs_home_first_vis)::FLOAT / (60) AS minutes_from_home_first_vis_to_pur,
  EXTRACT(EPOCH FROM ts_sub_interval_pur_vs_home_first_vis)::FLOAT / (60*60) AS hours_from_home_first_vis_to_pur,
  -- CEIL used to start at 1 
  CEIL(EXTRACT(EPOCH FROM ts_sub_interval_core_action_vs_pur)::FLOAT / (60*60*24)) AS sub_day_core_action_first_completed_at,
  CEIL(EXTRACT(EPOCH FROM ts_sub_interval_core_action_vs_pur)::FLOAT / (60*60*24*30)) AS sub_month_core_action_first_completed_at
FROM include_intervals
```

#### Timezones

-   many databases default to using UTC (doesn't have daylight savings)
-   confirm with data engineers the default database time zone to be safe
-   note timezones with daylight savings will have 2 standard abbreviations (i.e. PST daylight savings not in effect and PDT daylight savings in effect)
-   PostgreSQL does not have a built-in convert_timezone() function like some database vendors

```         
-- PostgreSQL approach to convert from PDT to UTC
WITH example_ts AS (
    SELECT TIMESTAMP '2023-04-04 15:00:00' AT TIME ZONE 'PDT' AS timestamp_pdt
)

SELECT
    timestamp_pdt,
    timestamp_pdt AT TIME ZONE 'UTC' AS timestamp_utc
FROM example_ts
```

## Practical Use Cases

#### Day of Quarter

-   example use case where built-in function doesn't exist
-   leverage combination of existing functions to achieve desired result

```         
WITH ts_example AS (
    SELECT TO_TIMESTAMP('2021-01-01 09:30:00', 'YYYY-MM-DD HH:MI:SS') AS timestamp_example
        UNION ALL
    SELECT TO_TIMESTAMP('2021-01-13 09:30:00', 'YYYY-MM-DD HH:MI:SS') AS timestamp_example
        UNION ALL
    SELECT TO_TIMESTAMP('2021-04-10 09:30:00', 'YYYY-MM-DD HH:MI:SS') AS timestamp_example
        UNION ALL
    SELECT TO_TIMESTAMP('2021-12-28 09:30:00', 'YYYY-MM-DD HH:MI:SS') AS timestamp_example
)

SELECT
    -- add 1 so count starts 1; default is 0
    EXTRACT(DAY FROM timestamp_example - DATE_TRUNC('QUARTER', timestamp_example))+1 AS day_of_quarter
FROM ts_example
```

#### Calendar days vs 24-hour-windows difference

-   use case where we want to look at 24 hour intervals as a "day" vs a calendar day
-   i.e. 24 hour units can be more useful to assess time to first action when users are spread through a day vs calendar day differences (e.g. 11:30 pm to 1am represents 1 calendar day difference but is within the same 24 hour interval)

```         
WITH user_events(user_id, event_type, event_timestamp) AS (
    VALUES
        (1, 'sign_up', '2024-01-01 08:00:00'::timestamp),
        (1, 'first_action', '2024-01-01 15:30:00'::timestamp),  
        (2, 'sign_up', '2024-01-02 09:15:00'::timestamp),
        (2, 'first_action', '2024-01-03 11:45:00'::timestamp),  
        (3, 'sign_up', '2024-01-03 07:20:00'::timestamp),
        (3, 'first_action', '2024-01-06 18:55:00'::timestamp),
        (4, 'sign_up', '2024-01-04 11:00:00'::timestamp)
        -- User 4 has signed up but not completed the first action
)

, user_events_agg AS (
  SELECT
    user_id,
    MIN(CASE WHEN event_type = 'sign_up' THEN event_timestamp END) AS first_sign_up_timestamp,
    MIN(CASE WHEN event_type = 'first_action' THEN event_timestamp END) AS first_action_timestamp
  FROM user_events
  GROUP BY 1
  ORDER BY 1
)

SELECT
    *,
    first_action_timestamp::DATE - first_sign_up_timestamp::DATE AS cal_days_till_first_action,
    first_action_timestamp < first_sign_up_timestamp::DATE + 1 AS first_action_within_first_24_hours, 
    EXTRACT(EPOCH FROM first_action_timestamp - first_sign_up_timestamp)::FLOAT / (60*60) AS hrs_to_first_action,
    -- i.e. 2 represents the 2nd 24 hour window since signup when the first action occurs
    -- CEIL to start the count at 1; FLOOR would start the count at 0
    CEIL(EXTRACT(EPOCH FROM first_action_timestamp - first_sign_up_timestamp)::FLOAT / (60*60*24)) AS user_24hr_windows_at_first_action
FROM user_events_agg
```

#### Leap years

-   output leap years using leap year rule filtering and identify the leap year equal to after the current year

```         
------------------------------------------
-- leap year rule
------------------------------------------
-- if a year is evenly divisible by 4, it is a leap year, except:
-- if the year is evenly divisible by 100, it is NOT a leap year, unless:
-- the year is also evenly divisible by 400, in which case it is a leap year
------------------------------------------

-- leverage PostgreSQL generate_series function to create years sequence
WITH leap_years_1900_to_3000 AS (
  SELECT 
      year
  FROM generate_series(1900, 3000) AS year
  WHERE (year % 4 = 0 AND year % 100 <> 0) OR (year % 400 = 0)
)

-- identify the leap year equal to after the current year
SELECT
  year
FROM leap_years_1900_to_3000
WHERE year >= EXTRACT(YEAR FROM CURRENT_DATE)
ORDER BY year ASC
LIMIT 1
```

#### Round a date timestamp to the nearest calendar day

-   like most coding problems there are multiple solution paths to same end result here
-   below solution path is a concise approach example

```         
WITH sample_data AS (
    SELECT '2022-04-02 00:06:00'::timestamp AS original_timestamp
       UNION ALL
    SELECT '2022-04-02 08:21:00'::timestamp AS original_timestamp
       UNION ALL
    SELECT '2022-04-02 11:59:59'::timestamp AS original_timestamp
         UNION ALL
    SELECT '2022-04-02 12:00:00'::timestamp AS original_timestamp
         UNION ALL
    SELECT '2022-04-02 16:30:00'::timestamp AS original_timestamp 
)

SELECT
    original_timestamp,
      -- if timestamp is before noon then +12 hours keeps the timestamp in same day
      -- if timestamp at noon or later then +12 hours pushes timestamp to the next day
    (original_timestamp + INTERVAL '12 hours')::DATE AS rounded_to_nearest_calendar_day
FROM sample_data
ORDER BY 1
```

#### CTE to assist with date filtering

-   useful when a query filters on a date in multiple locations
-   update the date filter once vs multiple locations

```         
-- CTE with filter data
WITH filter_date AS (
    SELECT '2023-03-12'::DATE as filter_var
),

sales_data AS (
    SELECT 1 AS sale_id, '2023-03-11'::DATE AS sale_date, 100 AS amount
    UNION ALL
    SELECT 2, '2023-03-12'::DATE, 150
    UNION ALL
    SELECT 3, '2023-03-13'::DATE, 200
)

-- filter sales data use CTE date filter
SELECT
    *
FROM sales_data
WHERE sale_date = (SELECT filter_var FROM filter_date)
```

#### Find all date gaps in a series of dates

-   left join to the complete date sequence with NULL where clause to surface date gaps

```         
DROP TABLE IF EXISTS temp_date_series;
CREATE TEMP TABLE temp_date_series AS (
    SELECT
        '2022-10-01'::DATE AS series_start,
        '2022-12-31'::DATE AS series_end
);

-- example reporting data on new purchase volume by day
DROP TABLE IF EXISTS temp_new_purchasers;
CREATE TEMP TABLE temp_new_purchasers AS (
    SELECT 
      purchase_date,
      ROUND((RANDOM()::NUMERIC * 100)) AS first_time_purchasers
    FROM GENERATE_SERIES(
        (SELECT series_start FROM temp_date_series), 
        (SELECT series_end FROM temp_date_series), 
        '1 day'::interval
    ) AS purchase_date
);

-- set seed to replicate results on query rerun
SELECT SETSEED(0.1);

-- randomly drop rows to create intentional date gaps
DELETE FROM temp_new_purchasers
WHERE RANDOM() < 0.05;

-- generate date sequence without gaps
-- in practice, we'd use a calendar table here
DROP TABLE IF EXISTS temp_complete_date_sequence;
CREATE TEMP TABLE temp_complete_date_sequence AS (
  SELECT 
    sequence_date
  FROM GENERATE_SERIES(
        (SELECT series_start FROM temp_date_series), 
        (SELECT series_end FROM temp_date_series), 
        '1 day'::interval
      ) AS sequence_date
);

-- return missing dates
SELECT 
    c.sequence_date
FROM temp_complete_date_sequence AS c
LEFT JOIN temp_new_purchasers AS t 
    ON t.purchase_date = c.sequence_date
WHERE t.purchase_date IS NULL
```

#### Fill in date gaps in a series of dates

-   for the date gaps in the new customers table we get NULL values for `first_time_purchasers` and replace the NULLs with 0 so have a complete date sequence

```         
-- using example data created above
-- date gaps filled and 0s used when gap exists
SELECT 
    c.sequence_date,
    COALESCE(t.first_time_purchasers, 0) AS first_time_purchasers_clean
FROM temp_complete_date_sequence AS c
LEFT JOIN temp_new_purchasers AS t 
    ON t.purchase_date = c.sequence_date
ORDER BY c.sequence_date ASC
```

#### Filtering dates with BETWEEN

-   remember BETWEEN clause is inclusive of beginning and end value

```         
WITH monthly_sales(year_month, sales_amount) AS (
  VALUES
    ('2021-10', 20),
    ('2021-11', 30),
    ('2021-12', 40),
    ('2022-01', 90),
    ('2022-02', 120),
    ('2022-03', 200),
    ('2022-04', 300),
    ('2022-05', 400),
    ('2022-06', 650),
    ('2022-07', 400),
    ('2022-08', 450),
    ('2022-09', 500),
    ('2022-10', 550),
    ('2022-11', 600),
    ('2022-12', 650),
    ('2023-01', 700),
    ('2023-02', 750),
    ('2023-03', 800),
    ('2023-04', 850)
)

SELECT
    *
FROM monthly_sales
-- use BETWEEN to grab 2022 months
-- TO_DATE() converts a string to a date
WHERE TO_DATE(year_month, 'YYYY-MM') BETWEEN '2022-01-01'::DATE AND '2022-12-01'::DATE
```

#### Common timestamp filtering mistake

-   BETWEEN date is being used to filter a timestamp but the end value unexpectedly limits to a day before the end date

```         
-- objective: filter to orders occurred on Jan 1, Jan 2, or Jan 3

WITH orders (order_id, order_timestamp) AS (
  VALUES
    (1, '2024-01-01 08:30:00'::timestamp),
    (2, '2024-01-01 15:45:00'::timestamp),
    (3, '2024-01-02 10:15:00'::timestamp),
    (4, '2024-01-02 20:00:00'::timestamp),
    (5, '2024-01-03 07:00:00'::timestamp),
      (6, '2024-01-03 12:00:00'::timestamp),
    (7, '2024-01-04 11:00:00'::timestamp),
      (9, '2024-01-04 00:00:00'::timestamp),
    (10, '2024-01-05 11:00:00'::timestamp)
)

-- does not deliver on objective; Jan 3 records not returned as desired
SELECT *
FROM orders
WHERE order_timestamp BETWEEN '2024-01-01'::DATE AND '2024-01-03'::DATE;

-- convert timestamp inline to date and filter works as desired
SELECT *
FROM orders
WHERE order_timestamp::DATE BETWEEN '2024-01-01'::DATE AND '2024-01-03'::DATE;

-- sometimes folks use the following logic
-- technically does not achieve desire result
-- jan 4 records could get returned if timestamp occur at exactly midnight
SELECT *
FROM orders
-- end point filter set to 1 day after the desired end date range
WHERE order_timestamp BETWEEN '2024-01-01'::DATE AND '2024-01-04'::DATE;

-- another option without using BETWEEN
SELECT *
FROM orders
WHERE 1=1
    AND order_timestamp >= '2024-01-01'::DATE 
    AND order_timestamp < '2024-01-04'::DATE;
```

### ADDING and SUBTRACTING INTERVALS

-   useful for date/timestamp math and filtering/windowing data

```         
-- adding interval units to a date
SELECT 
    '2024-01-10'::date + INTERVAL '1 year' AS plus_one_year,
    '2024-01-10'::date + INTERVAL '1 month' AS plus_one_month,
    '2024-01-10'::date + INTERVAL '1 week' AS plus_one_week,
    '2024-01-10'::date + INTERVAL '1 day' AS plus_one_day;

--subtracting interval units from a date
SELECT 
    '2024-01-10'::date - INTERVAL '1 year' AS minus_one_year,
    '2024-01-10'::date - INTERVAL '1 month' AS minus_one_month,
    '2024-01-10'::date - INTERVAL '1 week' AS minus_one_week,
    '2024-01-10'::date - INTERVAL '1 day' AS minus_one_day;
    
-- similar approaches work for timestamps
-- could also use other interval units like hour, minute, second
SELECT 
    CURRENT_TIMESTAMP + INTERVAL '1 hour' AS plus_one_hour,
    CURRENT_TIMESTAMP - INTERVAL '1 hour' AS minus_one_hour;
```
