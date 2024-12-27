# 15 SQL Query Best Practice Reminders

1.  **Verify column value logic**

    -   Avoid assumptions on what column values represent (validate against ETL code, read documentation, consult peers, etc).

2.  **Sanity check query outputs**

    -   Avoid assuming data output is correct because a query ran successfully.

    -   Cross-check with manual calculations, other data sources, known baseline/stable metrics, business knowledge, etc.

3.  **Test logic on sample datasets**

    -   Validate logic on subset of data to catch issues early and increase iteration speed.

4.  **Handle NULL values strategically**

    -   Consider NULL value behavior in calculations and filters.

    -   Check columns for NULL values: `COUNT(*) - COUNT(column_1) AS column_1_null_count`.

5.  **Routinely monitor for unexpected values and dirty data**

    -   Monitor for data quality erosion (e.g. a query could initially pass QA and later be affected by source data changes or bugs).
    -   Consider writing a set of QA tests for critical outputs.

6.  ***Avoid 'SELECT \*' outputs on large data***

    -   Specify columns to reduce output size.

    -   i.e. if downloading output to CSV, select specific columns of interest when possible vs `SELECT *` (especially important on wide and long tables when many columns will not be used for analysis).

7.  **Attempt to join on similar grain for large tables**

    -   1 to 1 relationship joins between large tables tend to be more efficient vs 1 to many relationships.

    -   i.e. User_id 123 is one row in Table A and one row in Table B.

8.  **Include 'LIMIT X' when exploring data**

    -   Reduces load on database and allows for faster query execution.

    -   `SELECT order_id, amount FROM large_orders_table LIMIT 10`

9.  **Help your future self + colleagues with code comments/styling**

    -   Include inline code comments for complex logic.
    -   Follow an agreed upon style guide to assist with code readability/logic understanding speed.

10. **Include data filters as early as possible**

    -   Reduce downstream processing load/run time by filtering to required data early in query logic (i.e. in first CTE where relevant vs last CTE).

11. **Check of unexpected fanout in joins**

    -   i.e. an analyst anticipates a 1 to 1 relationship for key X between Table A and Table B; however, duplicates exist for key X in Table B resulting in multiplicative output (fanout).
    -   Validate expected uniqueness prior to joins.

12. **Integer division defaults**

    -   `SELECT 3 / 4` in PostgreSQL defaults to integer division which returns 0 instead of 0.75 as one might expect (integer division).
    -   `SELECT ROUND(1.0 * 3/4,2)` returns 0.75 (floating point division).
    -   Check your SQL dialect documentation to determine if the dialect defaults to integer division vs floating point division.

13. **When possible, simplify 'where' and 'join' logic on large datasets**

    -   Simpler 'where' and 'join' logic often results in more efficient/faster queries.
    -   i.e. alongside other join/where logic, filtering on a URL string tends to be resource intensive and bogs down query execution time (analysts might explore pre-processing step to reduce load in join/where logic).

14. **Timezone handling can be tricky**

    -   UTC tends to be the most common timezone default. However, different data sources, tables, and tools may handle time zone conversion differently.
    -   Often recommended to validate assumptions with data engineers.

15. **Break complex logic into temp tables or CTEs**

    -   Improves readability, makes QA easier, and can enhance query performance.
