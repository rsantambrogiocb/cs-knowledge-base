# JOIN Statement
SQL join is used to **combine rows from two tables**. To perform a join, each table must have at least one field that will be used to find the matching records in the other table.
The join type defines which records will go into the dataset.

### INNER JOIN
The result set would **contains only data (rows) where the criteria matches**.
### OUTER JOIN
An OUTER JOIN will always **contains the result set of the INNER JOIN, but may contains also records that have no matching in the other table**.

Outer joins are divided in more subtypes:
- **LEFT JOIN** the result set will *contains all records from the left table; if no matching is found in the right table,* for that row from left table *all fields from right table will have NULL value*
- **RIGHT JOIN** the result set will *contain all records from the right table; if no matching records were found in the left table, then those fields will have NULL values*
- **FULL OUTER JOIN** *all records from the two tables will be included in the result set; if no matching values are found in the left, or right, table then the corresponding columns will have NULL values*; performing a full outer join between a table A and an empty table, will result in a dataset containing all rows from table A.
### SELF JOIN
A table is joined with itself. Typically, to work with this type of join, are used the inner join or the left join.
### CROSS JOIN
Cross join can be defined as a cartesian product of the two tables included in the join. The result set will have all possible combination of all rows.

- The table after join contains the same number of rows as in the cross-product of the number of rows in the two tables;
-  If a WHERE clause is used in cross join then the query will work like an INNER JOIN;
- The cross join has not the ON condition, because it does not check for matching rows;
- If you cross join a table with an empty table, the result-set will have 0 rows because no combinations can be made with an empty table.

### MULTIPLE JOIN STATEMENTS
``` sql
select distinct concat(m.firstname, ' ', m.surname), f.name facility
from cd.members m
join cd.bookings b on m.memid = b.memid
join cd.facilities f on b.facid = f.facid
```
The second JOIN in this query has a right hand side of cd.facilities. The 
left hand side, however, is the table returned by joining cd.members to cd.bookings.