# Day 5: Santa's Production Dashboard 🎁

Skills Used: LAG(), ROUND(), window functions

### Example Schema
```sql
DROP TABLE IF EXISTS toy_production CASCADE;
CREATE TABLE toy_production (
  production_date DATE PRIMARY KEY,
  toys_produced INTEGER
);
```

### Example Data
```sql
INSERT INTO toy_production (production_date, toys_produced) VALUES
  ('2024-12-18', 500),
  ('2024-12-19', 550),
  ('2024-12-20', 525),
  ('2024-12-21', 600),
  ('2024-12-22', 580),
  ('2024-12-23', 620),
  ('2024-12-24', 610);
```

## Solution

### Query

```sql
with i as (
  select *,
    lag(toys_produced) over (order by production_date) as previous_day_production
  from toy_production
  order by production_date asc
),
ii as (
  select *,
    toys_produced - previous_day_production as production_change
  from i
),
iii as (
  select *,
    production_change::numeric / previous_day_production::numeric * 100 as production_change_percentage
  from ii
  where previous_day_production is not null
)
select production_date, toys_produced, previous_day_production, production_change, round(production_change_percentage, 2)
from iii;
```

### Output

| production_date | toys_produced | previous_day_production | production_change | production_change_percentage |
|-----------------|---------------|-------------------------|-------------------|------------------------------|
 2011-03-26      |          4180 |                    9974 |             -5794 |   -58.09
 2011-03-27      |          9777 |                    4180 |              5597 |   133.90
 2011-03-28      |          6013 |                    9777 |             -3764 |   -38.50
 2011-03-29      |          3470 |                    6013 |             -2543 |   -42.29
 2011-03-30      |          6464 |                    3470 |              2994 |    86.28
 2011-03-31      |          4472 |                    6464 |             -1992 |   -30.82

 ...

### Final Query

 ```sql
with i as (
  select *,
    lag(toys_produced) over (order by production_date) as previous_day_production
  from toy_production
  order by production_date asc
),
ii as (
  select *,
    toys_produced - previous_day_production as production_change
  from i
),
iii as (
  select *,
    production_change::float / previous_day_production::float * 100 as production_change_percentage
  from ii
  where previous_day_production is not null
)
select production_date from iii
order by production_change_percentage desc
limit 1;
```

### Output

| production_date |
|-----------------|
 2017-03-20
 
(1 row)
