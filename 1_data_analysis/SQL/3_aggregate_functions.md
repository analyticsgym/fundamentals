# Aggregate Functions

#### Aggregate functions overview

-   return a single result output from set of input values
-   common calculation based aggregate functions: COUNT, SUM, AVG, MAX, MIN , etc
-   other types of aggregate functions: STRING_AGG, ARRAY_AGG, etc

#### Aggregates with NULL values

-   by default, most aggregate calculation functions ignore NULL values
-   COALESCE can be used set NULLs to a default value to be included in calcs
-   important to consider if NULLs should or should not be included in SQL aggregate calcs

``` sql
CREATE TEMP TABLE temp_pet_goats AS
SELECT *
FROM (VALUES
    ('becky', 4),
    ('jill', 2),
    ('becky', NULL),
    ('tom', 1),
    (NULL, 9),
    ('jill', 3)
) AS t(name, pet_goats);

-- as expected, average shifts based on how NULLs are handled
SELECT
    -- ignores NULL values by default
    AVG(pet_goats::FLOAT) AS avg_pet_goats_per_pet_person,
    -- NULL values set to 0
    AVG(COALESCE(pet_goats, 0)::FLOAT) AS avg_pet_goats_per_person_including_of_non_owners
FROM temp_pet_goats
```

#### COUNT

-   Subtle differences of how COUNT functions can be applied based on use case
-   COUNT(\*) counts number of rows
-   COUNT(<column_name>) count of non-NULL column values
-   COUNT(DISTINCT <column_name>) count unique of non-NULL column values

``` sql
SELECT
    COUNT(*) AS row_count, -- inclusive of NULL values
    COUNT(name) AS number_of_pet_goat_owner_names, -- excludes NULL values by default
    COUNT(DISTINCT name) AS number_of_unique_pet_goat_owner_names, -- excludes NULL values by default
    COUNT(*) - COUNT(name) AS number_of_null_pet_goat_owner_names
-- temp table generate above
FROM temp_pet_goats
```

#### Median and Percentiles

-   PERCENTILE_DIST: returns an existing data point at or exceeding the percentile threshold (does not use interpolation)
-   PERCENTILE_CONT: returns a percentile value with interpolation if needed
-   percentiles represent the proportion of values in a distribution that are less than the percentile value
-   sidebar on interpolation vs extrapolation: interpolation estimates a value between two known values vs extrapolation estimates a value beyond known values
-   continuous vs discrete percentile: continuous percentile is the proportion of values less than the percentile value vs discrete percentile is the proportion of values less than or equal to the percentile value
-   median = 50th percentile = 0.5 continuous percentile

``` sql
WITH student_scores(student_id, test_score) AS (
            VALUES
            (1, 85),
            (2, 78),
            (3, 92),
            (4, 88),
            (5, 74),
            (6, 81),
            (7, 67),
            (8, 95),
            (9, 89),
            (10, 72),
            (11, 90),
            (12, 77),
            (13, 83),
            (14, 65),
            (15, 80)
)

-- PERCENTILE_CONT and PERCENTILE_DISC are a type of ordered-set aggregate function
SELECT 
    DISTINCT
    -- continuous percentile
    PERCENTILE_CONT(0.05) WITHIN GROUP (ORDER BY test_score) AS p05_score,
    PERCENTILE_CONT(0.25) WITHIN GROUP (ORDER BY test_score) AS p25_score,
    PERCENTILE_CONT(0.50) WITHIN GROUP (ORDER BY test_score) AS median_score,
    PERCENTILE_CONT(0.75) WITHIN GROUP (ORDER BY test_score) AS p75_score,
    PERCENTILE_CONT(0.95) WITHIN GROUP (ORDER BY test_score) AS p95_score,
    -- discrete percentile
    PERCENTILE_DISC(0.05) WITHIN GROUP (ORDER BY test_score) AS discrete_05_score,
    PERCENTILE_DISC(0.25) WITHIN GROUP (ORDER BY test_score) AS discrete_25_score,
    PERCENTILE_DISC(0.50) WITHIN GROUP (ORDER BY test_score) AS discrete_median_score,
    PERCENTILE_DISC(0.75) WITHIN GROUP (ORDER BY test_score) AS discrete_75_score,
    PERCENTILE_DISC(0.95) WITHIN GROUP (ORDER BY test_score) AS discrete_95_score
FROM student_scores
```

#### Mode
- ordered-set aggregate function similar to above logic
- not implemented in all SQL dialects
- observation/segmentation counts tend to be used more frequently than the mode in practice

```sql
WITH show_ratings(show_id, user_id, rating) AS (
  VALUES 
  (1, 100, 9),
  (1, 101, 8),
  (1, 102, 9),
  (1, 103, 7),
  (2, 101, 6),
  (2, 102, 5),
  (2, 103, 7),
  (3, 101, 9),
  (3, 102, 8),
  (3, 103, 9),
  (4, 101, 4),
  (4, 102, 5),
  (4, 103, 6)
)

SELECT
  show_id,
  -- if ties for mode, returns largest rating value
  MODE() WITHIN GROUP (ORDER BY rating DESC) AS mode_rating,
  COUNT(*) AS ratings_count,
  COUNT(DISTINCT rating) AS unique_ratings_count
FROM show_ratings
GROUP BY 1
```

#### Variance and Standard Deviation

-   VAR_POP and STDDEV_POP used for entire population datasets
-   VAR_SAMP and STDDEV_SAMP used for samples (adjustments made for sample size vs population metrics)
-   for large sample sizes/data sets, population vs sample variance/standard deviation expected to be similar
-   standard deviation is a helpful metric for assessing the variability/dispersion of a dataset alongside measures of centrality (i.e. averages, etc)

``` sql
WITH batting_sample AS (
    SELECT 
        UNNEST(ARRAY[1,2,3,4,5,6,7,8,9,10,11,12]) AS player,
        UNNEST(ARRAY[0.287, 0.312, 0.254, 0.326, 0.268, 0.291, 
                     0.299, 0.276, 0.305, 0.282, 0.320, 0.294]) AS batting_average
)

SELECT
    AVG(batting_average::FLOAT) AS average_batting_average,
    VAR_SAMP(batting_average::FLOAT) AS sample_variance,
    STDDEV_SAMP(batting_average::FLOAT) AS sample_standard_deviation
FROM batting_sample 
```

#### Binary Flags with MAX

-   Common approach for creating product usage flags

``` sql
-- update with cleaner code
WITH installs(user_id, install_date, platform) AS (
      VALUES
        (1, '2023-01-01', 'iOS'),
        (1, '2023-01-02', 'Android'),
        (1, '2023-01-02', 'Apple TV'),
        (2, '2023-01-03', 'iOS'),
        (3, '2023-01-04', 'Android'),
        (4, '2023-01-05', 'iOS'),
        (4, '2023-01-05', 'Fire TV'),
        (4, '2023-01-05', 'Android TV')
  )

SELECT
  user_id,
  MAX(CASE WHEN platform = 'iOS' THEN 1 ELSE 0 END) AS flag_ios_installed,
  MAX(CASE WHEN platform = 'Android' THEN 1 ELSE 0 END) AS flag_android_installed,
  MAX(CASE WHEN platform = 'Apple TV' THEN 1 ELSE 0 END) AS flag_apple_tv_installed,
  MAX(CASE WHEN platform = 'Android TV' THEN 1 ELSE 0 END) AS flag_apple_tv_installed,
  MAX(CASE WHEN platform = 'Fire TV' THEN 1 ELSE 0 END) AS flag_fire_tv_installed
FROM installs
GROUP BY user_id
```

#### Conditional aggregate functions

-   spreadsheet functions such as COUNTIF, SUMIF, AVGIF, etc
-   most common approach uses CASE WHEN statements within aggregate functions
-   postgreSQL also has FILTER clause for aggregate functions to filter data before aggregation

Conditional aggregation with CASE WHEN

``` sql
WITH state_parks(park_id, park_name, state, visitors, revenue, has_camping) AS (
  VALUES 
  (1, 'Park A', 'FL', 5000, 150000, true),
  (2, 'Park B', 'CA', 8000, 200000, false),
  (3, 'Park C', 'CA', 12000, 300000, true),
  (4, 'Park D', 'CA', 4000, 80000, false),
  (5, 'Park E', 'MA', 7000, 250000, true),
  (6, 'Park F', 'MA', 3000, 50000, false),
  (7, 'Park G', 'TN', 9500, 280000, true),
  (8, 'Park H', 'TN', 6000, 120000, false)
)

SELECT 
  state,
  SUM(CASE WHEN has_camping = true THEN visitors ELSE 0 END) AS total_visitors_park_with_camping,
  SUM(CASE WHEN has_camping = true THEN revenue ELSE 0 END) AS total_revenue_park_with_camping,
  SUM(CASE WHEN has_camping != true THEN visitors ELSE 0 END) AS total_visitors_park_without_camping,
  SUM(CASE WHEN has_camping != true THEN revenue ELSE 0 END) AS total_revenue_park_without_camping
FROM state_parks
GROUP BY state
ORDER BY total_visitors_park_with_camping DESC
```

Conditional aggregation with FILTER clause

``` sql
WITH transactions(transaction_id, user_id, amount, category, user_type) AS (
  VALUES 
  (1, 101, 100.00, 'Groceries', 'Individual'),
  (2, 102, 150.00, 'Electronics', 'Business'),
  (3, 103, 200.00, 'Groceries', 'Individual'),
  (4, 101, 50.00, 'Utilities', 'Individual'),
  (5, 102, 300.00, 'Electronics', 'Business'),
  (6, 103, 400.00, 'Travel', 'Individual'),
  (7, 101, 1000.00, 'Travel', 'Business'),
  (8, 102, 80.00, 'Groceries', 'Individual'),
  (9, 103, 90.00, 'Utilities', 'Business')
)

SELECT 
  category,
  COUNT(*) AS transaction_count,
  COUNT(DISTINCT user_id) AS user_count,
  COUNT(*) FILTER (WHERE amount > 100) AS count_transactions_over_100,
  AVG(amount) FILTER (WHERE amount < 1000) AS avg_amount_under_1k,
  AVG(amount) FILTER (WHERE user_type = 'Business') AS avg_business_user_type_amount
FROM transactions
GROUP BY category
```

#### Flagging outliers

-   outliers tend to be defined as unusual observations different from the rest of the data (specific to the data distribution and domain context)
-   several methods can be used to identify outliers (e.g. z-scores, IQR, etc)
-   method tradeoffs based on data distribution, domain context, and business requirements

Example outlier logic

1.  data point greater than 2 standard deviations from the mean (useful for normally distributed data)
2.  data point below p25 - 1.5 \* IQR or above p75 + 1.5 \* IQR (useful for skewed data)

``` sql
WITH youtube_video_views(video_id, view_count) AS (
  VALUES 
  (1, 1200),
  (2, 1100),
  (3, 1150),
  (4, 1300),
  (5, 1250),
  (6, 1350),
  (7, 1400),
  (8, 1450),
  (9, 1500),
  (10, 1550),
  (11, 5000),
  (12, 5100),
  (13, 100),
  (14, 1600),
  (15, 1650),
  (16, 1700),
  (17, 1750),
  (18, 1800),
  (19, 1850),
  (20, 6000)
)

, stats AS (
  SELECT 
    AVG(view_count::FLOAT) AS avg_views,
    STDDEV(view_count::FLOAT) AS sd_views,
    percentile_cont(0.25) WITHIN GROUP (ORDER BY view_count) AS q1,
    percentile_cont(0.75) WITHIN GROUP (ORDER BY view_count) AS q3
  FROM youtube_video_views
)

SELECT 
    v.*,
    CASE 
      WHEN 
        (v.view_count > s.avg_views + (2*sd_views) OR 
         v.view_count < s.avg_views - (2*sd_views))
      THEN 'Yes'
      ELSE 'No'
    END AS outlier_flag_method_1,
    -- method 2 likely a better fit for this dataset
    CASE 
      WHEN 
        (v.view_count > (q3 + (q3 - q1)*1.5) OR 
         v.view_count < (q1 - (q3 - q1)*1.5))
      THEN 'Yes'
      ELSE 'No'
    END AS outlier_flag_method_2
FROM youtube_video_views AS v
INNER JOIN stats AS s
    ON 1=1
ORDER BY v.view_count DESC
```

#### Multi-level aggregation
-   grouping sets can be used to output multiple levels of aggregation in a single query
-   more streamlined code pattern than using UNION ALL to output multiple levels of aggregation
-   not available across all SQL/DB platforms

```sql
WITH zip_sales(state, city, total_sales, fake_zipcode) AS (
    VALUES
        ('NY', 'New York', 50000.00, 12345),
        ('NY', 'Buffalo', 20000.00, 45678),
        ('CA', 'Los Angeles', 75000.00, 54321),
        ('CA', 'San Francisco', 60000.00, 56789),
        ('TX', 'Houston', 45000.00, 11222),
        ('TX', 'Houston', 45000.00, 11223),
        ('TX', 'Dallas', 40000.00, 33444)
)

, grouping_set_setup AS (
SELECT
    fake_zipcode,
    city,
    state,
    SUM(total_sales) AS sales_agg
FROM zip_sales
GROUP BY GROUPING SETS(
        (fake_zipcode),
        (city, state),
        (state)
    )
)

SELECT
	CASE
		WHEN city IS NULL AND state IS NULL 
		THEN 'fake_zipcode'
		WHEN city IS NOT NULL AND state IS NOT NULL
		THEN 'city and state'
		WHEN city IS NULL AND state IS NOT NULL
		THEN 'state'
		ELSE 'error'
	END	AS aggregation_level,
	*
FROM grouping_set_setup
```

UNION ALL approach when GROUPING SETS function not available
- labeling can be done inline via UNION approach

```sql
SELECT
    'fake_zipcode' AS aggregation_level,
    fake_zipcode::VARCHAR,
    NULL AS city,
    NULL AS state,
    total_sales AS sales_agg
-- group by aggregation not needed here given data is at the zip level
FROM zip_sales

UNION ALL

SELECT
    'city and state' AS aggregation_level,
    NULL AS fake_zipcode,
    city,
    state,
    SUM(total_sales) AS sales_agg
FROM zip_sales
GROUP BY city, state

UNION ALL

SELECT
    'state' AS aggregation_level,
    NULL AS fake_zipcode,
    NULL AS city,
    state,
    SUM(total_sales) AS sales_agg
FROM zip_sales
GROUP BY state
```

#### Performance vs baseline year
-   one of several approaches to achieve desired end result

```sql         
WITH sales AS (
    SELECT 
        UNNEST(ARRAY[23456, 43567, 65812, 79234, 567394]) AS annual_sales,
        UNNEST(ARRAY[1999, 2000, 2001, 2002, 2003]) AS year
)

, baseline_year AS (
SELECT
    annual_sales,
    year
FROM sales
WHERE year = (SELECT MIN(year) FROM sales)
)

SELECT 
    s.*,
    b.annual_sales AS baseline_sales,
    CASE
        WHEN b.annual_sales IS NULL THEN 0
        ELSE ((s.annual_sales - b.annual_sales)::FLOAT /(b.annual_sales)) * 100
    END AS percent_change_vs_baseline
FROM sales AS s
INNER JOIN baseline_year AS b
    ON 1=1
```

#### Derive z-score
-   z-scores used to asses how far away data points are from the mean in terms of standard deviations from the mean
-   i.e. z-score of 0: data point is equal to the mean, z-score of 1: data point 1 standard deviation above the mean, z-score of -1: data point 1 standard deviation below the mean

```sql
WITH batting_sample AS (
    SELECT 
        UNNEST(ARRAY[1,2,3,4,5,6,7,8,9,10,11,12]) AS player,
        UNNEST(ARRAY[0.287, 0.312, 0.254, 0.326, 0.268, 0.291, 
                     0.299, 0.276, 0.305, 0.282, 0.320, 0.294]) AS batting_average
)

, sample_stats AS (
SELECT
    AVG(batting_average::FLOAT) AS average_batting_average,
    STDDEV_SAMP(batting_average::FLOAT) AS sample_standard_deviation
FROM batting_sample 
)

SELECT
    bs.player,
    bs.batting_average,
    s.average_batting_average AS sample_average_batting_average,
	s.sample_standard_deviation,
    (bs.batting_average - s.average_batting_average) / 
        s.sample_standard_deviation::FLOAT AS batting_average_zscore
FROM batting_sample AS bs
INNER JOIN sample_stats AS s
    ON 1=1
ORDER BY bs.batting_average DESC
```

#### Derive elements used to draw a boxplot
-   John Tukey invented the boxplot during the 1970s to further visualize and describe the distribution of data
-   lower whisker edge: 25th percentile - 1.5\*IQR
-   lower box edge: 25th percentile
-   median box line: 50th percenitle
-   upper box edge: 75th percentile
-   upper whisker edge: 75th percentile + 1.5\*IQR
-   note: points outside lower and upper whisker edge tend to plotted as points and represent outliers on the boxplot

```sql         
WITH test_scores(score) AS (
  VALUES 
  (41), (70), (81), (32), (75), 
  (51), (62), (83), (84), (79),
  (75), (76), (76), (78), (80),
  (79), (74), (60), (24), (19),
  (59), (67), (83), (98), (99)
)

, box_plot_values AS (
  SELECT 
    -- lower whisker edge = min data value if derived lower whisker edge exceeds min value in data
      GREATEST(
      MIN(score),
      PERCENTILE_CONT(0.25) WITHIN GROUP (ORDER BY score) - 
      1.5 * (PERCENTILE_CONT(0.75) WITHIN GROUP (ORDER BY score) - 
             PERCENTILE_CONT(0.25) WITHIN GROUP (ORDER BY score))
    ) AS lower_whisker_edge,
    
    PERCENTILE_CONT(0.25) WITHIN GROUP (ORDER BY score) AS lower_box_edge,
    
    PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY score) AS median_box_line,

    PERCENTILE_CONT(0.75) WITHIN GROUP (ORDER BY score) AS upper_box_edge,

    -- upper whisker edge = max data value if derived upper whisker edge exceeds max value in data
      LEAST(
      MAX(score),
      PERCENTILE_CONT(0.75) WITHIN GROUP (ORDER BY score) + 
      1.5 * (PERCENTILE_CONT(0.75) WITHIN GROUP (ORDER BY score) - 
             PERCENTILE_CONT(0.25) WITHIN GROUP (ORDER BY score))
    ) AS upper_whisker_edge
  FROM test_scores
)

-- if lower outliers exist, generate text string of outliers
, boxplot_low_outliers AS (
SELECT
    COALESCE(STRING_AGG(score::VARCHAR, ', ' ORDER BY score ASC), 'no lower outliers') AS lower_outliers
FROM test_scores
WHERE score < (SELECT lower_whisker_edge FROM box_plot_values)
)

-- if upper outliers exist, generate text string of outliers
, boxplot_up_outliers AS (
SELECT
    COALESCE(STRING_AGG(score::VARCHAR, ', ' ORDER BY score), 'no upper outliers') AS upper_outliers
FROM test_scores
WHERE score > (SELECT upper_whisker_edge FROM box_plot_values)
)

-- values used to generate a boxplot 
SELECT
    l.lower_outliers,
    b.lower_whisker_edge,
    b.lower_box_edge,
    b.median_box_line,
    b.upper_box_edge,
    b.upper_whisker_edge,
    u.upper_outliers
FROM box_plot_values AS b
INNER JOIN boxplot_up_outliers AS u
    ON 1=1
INNER JOIN boxplot_low_outliers AS l
    ON 1=1
```

#### Pre vs Post Analysis
-   aggregate functions to generate simple pre vs post overall conversion rate summary data

```sql         
DROP TABLE IF EXISTS temp_conversion_summary_data;
CREATE TEMP TABLE temp_conversion_summary_data AS
WITH conversion_data(date, site_visitors, conversions) AS (
    VALUES
        ('2023-05-01'::DATE, 100::INTEGER, 20::INTEGER),
        ('2023-05-02', 110, 19),
        ('2023-05-03', 120, 22),
        ('2023-05-04', 105, 21),
        ('2023-05-05', 115, 20),
        ('2023-05-06', 125, 24),
        ('2023-05-07', 130, 26),
        ('2023-05-08', 135, 28),
        ('2023-05-09', 140, 30),
        ('2023-05-10', 120, 24),
        ('2023-05-11', 155, 31),
        ('2023-05-12', 130, 26),
        ('2023-05-13', 140, 28),
        ('2023-05-14', 160, 32), 
        ('2023-05-15', 165, 37),
        ('2023-05-16', 180, 42),
        ('2023-05-17', 190, 47),
        ('2023-05-18', 200, 55),
        ('2023-05-19', 210, 60),
        ('2023-05-20', 220, 66),
        ('2023-05-21', 190, 53),
        ('2023-05-22', 195, 58),
        ('2023-05-23', 200, 63),
        ('2023-05-24', 205, 68),
        ('2023-05-25', 210, 73),
        ('2023-05-26', 215, 78),
        ('2023-05-27', 220, 83),
        ('2023-05-28', 225, 88)
)

SELECT
    CASE
        WHEN date BETWEEN '2023-05-01'::DATE AND '2023-05-14'::DATE
        THEN 'pre_period'
        WHEN date BETWEEN '2023-05-15'::DATE AND '2023-05-28'::DATE
        THEN 'post_period'
        ELSE 'error'
    END AS measurement_window,
    MIN(date) AS window_start_date, 
    MAX(date) AS window_end_date, 
    COUNT(DISTINCT date) AS measurement_period_days, 
    SUM(site_visitors) AS total_visitors,
    SUM(conversions) AS total_conversions,
    SUM(conversions)::FLOAT / SUM(site_visitors) AS window_conversion_rate
FROM conversion_data
GROUP BY 1
ORDER BY window_start_date ASC;

-- view example data setup
SELECT * FROM temp_conversion_summary_data
```

#### Two proportions z-test
- Two proportion z test or chi-square test often used to compare two proportions
- For example, we'll assumption 2 prop z-test assumptions are met
- Test steps: 1) generate z test statistic 2) compare z test statistic to z critical value 3) interpret results
- Pvalue not calculations not available in Postgres natively, so use z test statistic vs z critical value comparison for statistical significance check

```sql
-- two tail test setup
-- H0: p1 = p2
-- H1: p1 ≠ p2
-- alpha = 0.05
-- if z test statistic > 1.96 or < -1.96, reject null hypothesis
-- =/-1.96 is the z critical value for a two tail test at alpha = 0.05

WITH test_stats_setup AS (
SELECT
  MAX(CASE WHEN measurement_window = 'pre_period' THEN window_conversion_rate END) AS p1,
  MAX(CASE WHEN measurement_window = 'post_period' THEN window_conversion_rate END) AS p2,
  MAX(CASE WHEN measurement_window = 'pre_period' THEN total_visitors END) AS n1,
  MAX(CASE WHEN measurement_window = 'post_period' THEN total_visitors END) AS n2,
  SUM(total_conversions)::FLOAT / SUM(total_visitors) AS pooled_proportion
FROM temp_conversion_summary_data
)

, z_test_stat AS (
SELECT
    *,
    (p1 - p2) / SQRT(pooled_proportion * (1 - pooled_proportion) * ((1 / n1::FLOAT) + (1 / n2::FLOAT))) AS z_test_statistic
FROM test_stats_setup
)

SELECT
    *,
    -- two tail test z critical value approx 1.96
    CASE
      WHEN ABS(z_test_statistic) > 1.96 THEN 'reject null'
      ELSE 'fail to reject null'
    END AS test_result
FROM z_test_stat
```

Result interpretation: we reject the NULL in favor of the alternative hypothesis (e.g. difference in conversion rates between pre and post periods is statistically significant and unlikely to be due to chance assuming the NULL hypothesis distribution is true)

### Dynamic aggregation functions to create long data format
-   code pattern below using CASE WHEN to dynamically generate aggregation functions based on input parameters

```sql         
WITH user_movie_consumption(user_id, movie_id, minutes_consumed) AS (
  VALUES
  (101, 'A', 45),
  (101, 'B', 105),
  (102, 'C', 60),
  (102, 'D', 30),
  (103, 'E', 80),
  (103, 'F', 120),
  (104, 'G', 25),
  (105, 'H', 200)
)

, minutes_thresholds(threshold_level) AS (
  VALUES
  (30),
  (60),
  (90),
  (120)
)

, consumption_and_thresholds AS (
  SELECT
    u.*,
    m.threshold_level
  FROM user_movie_consumption AS u
  -- use caution with this approach if the cross join set is large
  CROSS JOIN minutes_thresholds AS m
)

SELECT
  user_id,
  threshold_level,
  COUNT(
    CASE 
      WHEN minutes_consumed >= threshold_level
      THEN movie_id
    END
  ) AS movies_consumed_at_or_above_threshold
FROM consumption_and_thresholds
GROUP BY 1, 2
ORDER BY 1, 2
```

#### Capping values
-   method to reduce outlier influence on outlier sensitive metrics (important for AB test analysis in some cases)
-   example capping approach: if a value is below 10th percentile cap the value at the 10th percentile; if a value is above 90th percentile cap the value at the 90th percentile

```sql         
WITH example_data(result) AS (
   VALUES 
    (1), (2), (3), (4), (5), (6), (7), (8), (9), (10), 
    (11), (12), (13), (14), (15), (16), (17), (18), (19), (20)
)

, percentiles AS (
SELECT 
    PERCENTILE_CONT(0.10) WITHIN GROUP (ORDER BY result) AS perc_10,
    PERCENTILE_CONT(0.90) WITHIN GROUP (ORDER BY result) AS perc_90
FROM example_data
)

-- both capping methods below produce the same result
SELECT 
  result, 
  -- capping using LEAST and GREATEST
  LEAST(GREATEST(result, perc_10), perc_90) as capped_result_1,
  -- more verbose/explicit approach using CASE WHEN
  CASE 
    WHEN result < perc_10 
    THEN perc_10
    WHEN result > perc_90 
    THEN perc_90
    ELSE result
  END as capped_result_2
FROM example_data AS ed
INNER JOIN percentiles AS p
    ON 1=1
```

#### Binning values
-  binning method to group continuous data into discrete bins
-  helpful for segmentation analysis and visualization setup
-  scalable approach below vs hard coded case when statements
-  case when logic likely needed for more custom binning logic (variable bin sizes, etc)

```sql
WITH example_data(metric_value) AS (
   VALUES 
    (-1), (0), (1), (10), (20), (30), (40), (50), 
    (60), (70), (80), (90), (100)
)

-- adjustable bin size parameter
, bin_settings AS (
  SELECT 30 AS bin_size
)

, bin_setup AS (
  SELECT 
    metric_value,
    FLOOR(metric_value::FLOAT / bin_size) * bin_size AS bin_start,
    FLOOR(metric_value::FLOAT / bin_size) * bin_size + bin_size - 1 AS bin_end
  FROM example_data
  INNER JOIN bin_settings
    ON 1=1
)

SELECT
  bin_start || ' to ' || bin_end AS bin_label,
  COUNT(*) AS observation_count
FROM bin_setup
GROUP BY bin_label
-- used to order the bins
ORDER BY MIN(metric_value) ASC
```

#### Hypothetical-Set Aggregate Functions
- calculates the result of a hypothetical value as if it were part of actual data

``` sql
WITH social_media_posts(post_id, user_id, likes, post_date) AS (
  VALUES 
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

SELECT 
    -- what would the rank of a post with 500 likes be?    
    rank(500) WITHIN GROUP (ORDER BY likes DESC) AS test1,
    -- what would the rank of a post with 105 likes be?
    rank(105) WITHIN GROUP (ORDER BY likes DESC) AS test2
FROM social_media_posts;
```

#### INT vs FLOAT vs NUMERIC data types
- INT: whole numbers
- FLOAT: approximate numeric with fractional precision
- NUMERIC: represent numbers exactly
- depending on the use case and DB vendor, the data type input can change results of calculations
- some DBs will default AVG and division operations to INT, which can lead to unexpected results
- additionally, the data input type can have performance impact on large datasets
- casting to FLOAT tends to be a good balance between performance and precision
- review DB documentation and build test cases to understand how data types impact calculation results

```sql
-- code pattern to check how different data types impact results
WITH example_data(metric) AS (
  VALUES 
    (1),
    (2),
    (3),
    (4),
    (5),
    (6)
)

SELECT
  AVG(metric::INT) AS avg_int,
  AVG(metric::FLOAT) AS avg_float,
  AVG(metric::NUMERIC) AS avg_numeric
FROM example_data
```