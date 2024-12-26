## Why Care About Query Performance

-   inefficient queries slow analyst iteration speed and tend to limit investigation depth
-   overly complex queries on large datasets can bog down compute resources/become costly

## Performance Optimization Considerations

#### Inner joins vs outer joins

-   inner joins tend to be more resource efficient/faster than outer joins (left, right, outer)
-   if an inner join works for the use case then default to using inner joins
-   limit the number of joins to only the required tables

#### Data filtering location

-   when possible, apply filtering conditions as early in the query structure as possible to minimize amount of data to process
-   for example, when feasible, include filtering conditions as part of inner join `ON` clause vs `WHERE` clause

#### LEFT JOINs that return a large NULL set

-   left joins on large datasets can be slow due to full scan required which returns NULLs for non-matches vs inner join grabs only matching records
-   when the large NULL set isn't needed for the query requirements explore using inner join logic

#### Unions

-   UNION vs UNION ALL: UNION tends to be more resource intensive on large datasets due to additional processing to remove duplicates
-   if possible, use UNION ALL instead of UNION
-   single query joins or sub queries often preferred over UNION

#### Pre-aggregate

-   for large granular datasets it can be helpful to pre-aggregate prior to joins (storing results as CTE or temp table)
-   this can help improve performance by reducing the amount of data processed during the join operation
-   i.e. imagine 3 tables with user info/event data (demographics, community posts, video starts)
-   pre-aggregation in this example could be aggregating community posts and video starts at the user level then joining to demographics at the same grain as user

#### Subqueries

-   often best to avoid subqueries within joins that return large result sets
-   subqueries within joins create an inefficient loop over the data where the subquery is executed repeatedly
-   use CTEs or temp tables instead prior to join logic
-   also best to avoid window functions in sub queries (use intermediate CTE or temp table to store results)

#### Window functions

-   filter data only to required rows to avoid unnecessary compute
-   grouping data into smaller partitions might help query efficiency/speed
-   nuances of the data can impact window function speed (i.e. number of ntiles, large offsets for lead/lag, large window frames, many rows with duplicate values, etc)
-   general themes on window function speed: tend to be fastest (ROW_NUMBER, RANK, DENSE_RANK), moderate speed (LAG, LEAD, FIRST_VALUE, LAST_VALUE), can be slower (NTILE, SUM, AVG, MIN, MAX, COUNT)

#### Downloading data from query UI

-   for wide datasets, selecting specific columns to output to file vs select \* can be faster and more readable for future code users
-   also throws an error if the table columns ever change after specification

#### Data exploration

-   include LIMITs during initial exploration of tables (e.g. prevents hanging queries)
-   create a small subset of data to test queries and logic on
-   for prototyping of techniques or testing functions consider generating a fake MVP dataset (e.g. create a TEMP TABLE and insert values, UNION ALL values together, etc)

#### Statistical sorting functions on large datasets (e.g. median, percentile)

-   can be slow on large datasets due to sorting and scanning large amounts of data
-   skewed distributions can also take longer to calculate
-   ideas to speed up query: data sampling, partitioning data, creating indexes on the calculation column

#### Misc

-   table indexing can help with query performance (works like a file cabinet of folders with tab labels)
-   SQL tools tend to have a query planner or explain functionality that can be used to unpack query run process (however, haven't see this used in practice; analysts tend to iterate with a test and learn approach to find optimal performance)
-   `WHERE` vs `HAVING` data filtering: use `WHERE` to filter data not dependent on aggregated results; use `HAVING` to filter aggregated data
-   test and compare alternative solutions to assess query speed/efficiency
