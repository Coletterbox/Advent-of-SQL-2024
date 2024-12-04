# Day 4: The Great Toy Tag Migration of 2024

## Table Schema
```sql
DROP TABLE IF EXISTS toy_production CASCADE;
CREATE TABLE toy_production (
  toy_id INT,
  toy_name VARCHAR(100),
  previous_tags TEXT[],
  new_tags TEXT[]
);
```

## Example data
```sql
INSERT INTO toy_production VALUES
  (1, 'Robot', ARRAY['fun', 'battery'], ARRAY['smart', 'battery', 'educational', 'scientific']),
  (2, 'Doll', ARRAY['cute', 'classic'], ARRAY['cute', 'collectible', 'classic']),
  (3, 'Puzzle', ARRAY['brain', 'wood'], ARRAY['educational', 'wood', 'strategy']);
```

with o as (
  select toy_id, unnest(previous_tags) as old
  from toy_production
),
n as (
  select toy_id, unnest(new_tags) as new
  from toy_production
),
unchanged as (
  select o.toy_id, old
  from o
  inner join n
  where o.toy_id = n.toy_id and old = new
)
select...
