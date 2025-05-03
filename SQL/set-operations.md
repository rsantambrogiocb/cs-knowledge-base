# Set Operations
To execute these statements the result sets must have the same number of columns, in the same order and with the same types.

### UNION
UNION operator **combines and returns the result-set retrieved by two or more SELECT statements**; **both results from the two queries must have the same number of columns and compatible data types**; *UNION removes duplicate rows, while UNION ALL does not;*

### MINUS
MINUS **removes duplicates in the result set from the first select statement**, comparing the result set from the second select query

### INTERSECT
INTERSECT combines the result-set fetched by the two SELECT statements where records from one match the other and then **returns this intersection of result-sets**.