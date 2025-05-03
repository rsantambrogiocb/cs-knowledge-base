# Window
Window allows us to name a SQL window so that it can be reused

```sql
select 
name, weight,
ntile(2) over ntile_window as by_half,
ntile(3) over ntile_window as thirds,
ntile(4) over ntile_window as quart
from cats 
window ntile_window as (order by weight)
```