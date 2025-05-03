# Window Functions

## Over
Over is used to **limit or tweaking data return from aggregate functions**. *Think it as of a running total.*

In the above example, the AVG(weight) is recomputed at each step: for each row of the result set the average as a different value because it is recomputed at each step; a step is the average computed using the values seen since to that point of ORDER BY. 

The important thing to notice is that the result set is not equals to have a result set out of a grouping by
```sql
select
name, weight,
avg(weight) over (order by name)
from runners order by name
limit 3
```

Remember the ORDER BY in a **OVER statement is not limited to one column**. You can specify how many columns you want to order the windows.

I see it as that part which allow me to create the window where to apply the window function. If not specify how to create the groupings, the window is the entire dataset and the functions are applied to each step.
## Partition By
Partition by **allows to further subdivide the preceding OVER command**. The aggregation function used with OVER is recomputed at each step, but the difference is that the **aggregation is reset when the partition field changes** and OVER step restart.

## Preceding and Following
Preceding and following **allow to perform aggregate functions on the rows just before and after the current row.**

The result set from the example below, will list the minimum weight of each runner weight, its neighbours just before and after them. Important to notice that the min is computed between three weights, the weight field of the current row, the weight filed of the row before and the one of the row after the current. The computation of the minimum for the first element of the result set is done just the current row weight field and the weight field of the row after (there is not a row before). The same for the last row of the result set.

``` sql
select
name,
min(weight) over (order by name ROWS between 1 preceding and 1 following)
from runners order by name
```

The result from the example below will give back the name with the running sum, but *when two rows have the same weight value the windows is stopped*: with **UNBOUNDED PRECEDING** you're saying that **there are not constraints on rows for the lower bound of the window**; but with AND **CURRENT ROW** **limiting the lower bound the window is stopped when the value of current row is encountered**.
``` sql
select name, 
	sum(weight) over (
		order by weight desc 
		rows between unbounded preceding and current row
	) as running_total_weight
from cats order by running_total_weight
```