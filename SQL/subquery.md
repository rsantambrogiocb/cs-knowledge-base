# Subquery
A sub-query is a query within another query, also known as nested query or inner query.
It is used to restrict or enhance the data to be queried by the main query.

There are two types of sub-query:
- **Correlated** which cannot be considered as an independent query, but it refers to columns in a table listed in the FROM of the main query
- **Non-correlated** can be considered as an independent query and the output of the sub-query is substituted with a new table/dataset in the main query.

The **IN** operator is a good demonstrator of the elegance of the relational model.
`select * from cd.facilities where facid in (1,5);`
The arguments of IN is not just a list of values, but it's actually a table with a single column.
Since queries returns tables, if you create a query that returns a single column table then you can feed the result into the IN operator.

**An example**
``` sql
select * 
from cd.facilities
where fac_id in (
	select fac_id from cd.other_table where condition = true
); 
```