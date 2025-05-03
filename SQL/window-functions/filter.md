# Filter
Filter is used in conjunction with aggregate or window functions allowing the filtering of values.

In this example, we print average runner time and then filter on the runners weighing less than 90kg to produce the light_runners_time column
```sql
select country,
avg(time) as avg_time,
avg(time) filter (where weight < 90) as light_runners_time
from runners group by country
```
