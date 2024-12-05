# Day 4: The Great Toy Tag Migration of 2024

### Table Schema
```sql
DROP TABLE IF EXISTS toy_production CASCADE;
CREATE TABLE toy_production (
  toy_id INT,
  toy_name VARCHAR(100),
  previous_tags TEXT[],
  new_tags TEXT[]
);
```

### Example data
```sql
INSERT INTO toy_production VALUES
  (1, 'Robot', ARRAY['fun', 'battery'], ARRAY['smart', 'battery', 'educational', 'scientific']),
  (2, 'Doll', ARRAY['cute', 'classic'], ARRAY['cute', 'collectible', 'classic']),
  (3, 'Puzzle', ARRAY['brain', 'wood'], ARRAY['educational', 'wood', 'strategy']);
```

## Solution

### Query
```sql
with o as (
  select toy_id, unnest(previous_tags) as old
  from toy_production
),
n as (
  select toy_id, unnest(new_tags) as new
  from toy_production
),
unchanged as (
  select o.toy_id, old as same
  from o
  inner join n
  on o.toy_id = n.toy_id and old = new
),
old_count as (
  select toy_id, count(old) as old_tags from o
  group by toy_id
),
new_count as (
  select toy_id, count(new) as new_tags from n
  group by toy_id
),
unchanged_count as (
  select toy_id, count(same) as unchanged_tags from unchanged
  group by toy_id
),
list as (
  select o.toy_id, old_tags, unchanged_tags, new_tags
  from old_count o
  full join new_count on o.toy_id = new_count.toy_id
  full join unchanged_count on o.toy_id = unchanged_count.toy_id
)
select toy_id,
  new_tags - unchanged_tags as added_tags,
  unchanged_tags,
  old_tags - unchanged_tags as removed_tags
from list
where new_tags is not null
  and unchanged_tags is not null
order by added_tags desc
limit 1;
```

### Output

| toy_id | added_tags | unchanged_tags | removed_tags |
--------|------------|----------------|--------------
   2726 |         98 |              2 |            0
