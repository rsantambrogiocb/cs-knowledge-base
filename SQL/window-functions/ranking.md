# Ranking
With ranking functions an ordered number is added to each of the output rows.

### RANK, DENSE_RANK, ROW_NUMBER
`row_number()` assign a unique number to each row even if two or more rows are equals.

Instead, `rank()` and `dense_rank()` assign the same order number to rows that have equal values, but in different way to the next row. Let's understand it with an example: considered the case in which two runners finish equals in the same 3rd position, with `dense_rank()`the next runner will be assigned to the 4th place, while `rank()` the next runner place would be 5th (skipping the 4th position).

### CUME_DIST and PERCENT_RANK
These two functions calculate the rank for a group of rows:
- `percent_rank()` returns a number from 1 to 0 comprehended, the highest being 1 and the lowest 0; it allows to generate distributions or percentiles;
- `cume_dist()` returns a number from 1 towards 0 (never returns 0)

If there are 4 different values, do you count from 1 in steps of 0.25 (percent_rank) or in steps of 0.2 ensuring that we never hit 0 (cume_dist)

### Examples
``` sql
select name, time,
percent_rank() over (order by time),
cume_dist() over (order by time)
FROM runners order by time
```

| name   | time | percent_rank | cume_dist |
| ------ | ---- | ------------ | --------- |
| andy   | 101  | 0            | 0.2       |
| bob    | 103  | 0.25         | 0.4       |
| cedric | 104  | 0.5          | 0.8       |
| dave   | 104  | 0.5          | 0.8       |
| eric   | 108  | 1            | 1         |

