# Grouping

### Lag and Lead
These two functions allow to examine the next (`LEAD`) or previous row (`LAG`). You can also compare the value under examination with the current row.

For example, consider a race scenario and we want to see the time of the person in front of us and the amount of time we beat the person behind us
```sql
select name, time,
lag(time, 1) over (order by time) as time_of_person_infront_of_me,
lead(time, 1) over (order by time) - time as how_much_i_was_in_front
from runners order by time
```

### nth_value and ntile
- `nth_value()` returns the nth value, but if we do not specify a range it will return NULL if the current value is less then the nth value; if we need always to display a result, we need to specify a range;
- `ntile`() divides the group into n equal partitions and denotes which partition each row is in; often used to divide the data into percentile or quartile;

Continuing with the race example, we want to determine how much faster a runner needs to go to finish on the podium (1st, 2nd or 3rd); then print print the time of the second runner; finally we want to determine if they are in the top half of runners
```sql
select name, time,
nth_value(time, 3) over (order by time) - time as to_go_faster_to_make_podium,
nth_value(time, 2) over (order by time RANGE BETWEEN UNBOUNDED PRECEDING 
	AND UNBOUNDED FOLLOWING) as time_of_second_runner,
ntile(2) over (order by time) as which_half
FROM runners order by time;

```