# Order of Execution of a SQL Query
Understanding the order of execution of a SQL query helps us to identify the steps involved in query optimization.

``` sql
SELECT DISTINCT column, AGG_FUNC(column_or_expression), …
FROM mytable
JOIN another_table
  ON mytable.column = another_table.column
WHERE constraint_expression
GROUP BY column
HAVING constraint_expression
ORDER BY column ASC/DESC
LIMIT count OFFSET COUNT;
```

### 1. FROM and JOIN
**FROM clause, and subsequent JOIN, are first executed to determine the total working set of data that is being queried.**
The result is a working dataset that contains all columns from all the referenced tables. **Subqueries in the FROM clause are executed at this point**, and temporary tables may be created to hold intermediate results. This step establishes the complete set of data before any filtering occurs.
### 2. WHERE
Filtering conditions are applied to the working dataset from step 1. **Only rows that satisfy the specified conditions are kept for further processing**. 
The WHERE clause **can only reference tables and columns that were made available in FROM/JOIN step**. *Aliases or expression defined in the SELECT can not be referenced here*, because it is not yet been processed at this point.
### 3. GROUP BY
If present, the database **groups the (filtered) rows based on the columns specified in the GROUP BY** clause. This transform the data from individual rows to groups of rows that share the same values in the grouped columns.
**After this step, there will be only one row for each unique combination of values in the GROUP BY** columns. 
This step is typically used when aggregate functions (SUM, COUNT, AVG, etc.) are present in the query.
### 4. HAVING
The HAVING clause filters the grouped data resulting from the GROUP BY step. The difference with the WHERE clause is that **HAVING filters rows after the groups have been formed**, but like WHERE **it can not reference aliases or expressions defined in the SELECT** clause.
HAVING is typically used with aggregate functions (e.g., HAVING COUNT() > 5).
### 5. SELECT
Now the database identifies which columns to include in the final result set.
Expressions, column aliases and calculations are all evaluated at this setp.
For each original column, or each group if there is a GROUP BY, the database compute the values for each item in the SELECT list, including aggregation functions.
### 6. WINDOW
Window functions (like ROW_NUMBER(), RANK(), LEAD(), LAG()) are processed after the SELECT list is determined. These functions perform calculations across a set of rows related to the current row without collapsing groups like aggregation functions. 
### 7. DISTINCT
If the DISTINCT clause is present, the database eliminates duplicate rows from the result set. A row is considered a duplicate if all selected columns have identical values to another row. This step reduces the result set to only unique combinations of the selected columns.
### 8. ORDER BY
The database sorts the result set according to the columns specified in the ORDER BY clause. The sorting can be ascending (ASC, which is the default) or descending (DESC) for each column.
At this point aliases defined in the SELECT can be referenced since it is already been processed. 
### 9. LIMIT / OFFSET
Finally, the database applies any row count restrictions.
**LIMIT specifies the maximum number of rows to return**, while **OFFSET determines how many rows to skip before starting to return rows**. Different database systems might use slightly different syntax.