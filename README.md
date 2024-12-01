https://wiki.termux.com/wiki/Postgresql

# Advent of SQL 2024

## In which: Santa and his elves are just as helpless as in Advent of Code

And who is the reader supposed to be in these things, anyway? Do we work for Santa? Is it a seasonal job? Would a person with this job list it on their CV?

---

## Solutions:

Example: The Great Christmas Analytics Crisis of 2024 (intermediate)

```sql
  select city, country, avg(naughty_nice_score) as avg_score, count(*) as children_count from children
  group by city, country
  order by avg_score desc;
```

|    city    |   country   |      avg_score      | children_count |
------------|-------------|---------------------|----------------
 Rome       | Italy       | 91.3333333333333333 |              6
 Amsterdam  | Netherlands | 91.2000000000000000 |              5
 Berlin     | Germany     | 91.1666666666666667 |              6
 Lyon       | France      | 90.6000000000000000 |              5
 Manchester | UK          | 90.6000000000000000 |              5
 Madrid     | Spain       | 90.4285714285714286 |              7
 London     | UK          | 89.5000000000000000 |              6
 Paris      | France      | 89.5000000000000000 |              6
 Barcelona  | Spain       | 89.5000000000000000 |              6
 Munich     | Germany     | 89.0000000000000000 |              5
 Milan      | Italy       | 89.0000000000000000 |              5
 Birmingham | UK          | 88.0000000000000000 |              5
 Rotterdam  | Netherlands | 39.0000000000000000 |              5
 
(13 rows)

```sql
  SELECT column_name, data_type FROM information_schema.columns where table_name = 'christmaslist';
```

|  column_name   | data_type |
----------------|-----------
 list_id        | integer
 child_id       | integer
 gift_id        | integer
 year           | integer
 was_delivered  | boolean
 delivery_order | integer
 
(6 rows)

```sql
  with avgs as (
    select city, country, avg(naughty_nice_score) as avg_score, count(*) as children_count from children
    left join christmaslist on children.child_id = christmaslist.child_id
    where was_delivered is true
    group by city, country
  )
  select city, country, avg_score from avgs
  where children_count >= 5
  order by country, avg_score desc;
```

|    city    |   country   |      avg_score      |
------------|-------------|---------------------
 Lyon       | France      | 90.6000000000000000
 Paris      | France      | 89.5000000000000000
 Rotterdam  | Netherlands | 39.0000000000000000
 Manchester | UK          | 90.6000000000000000
 London     | UK          | 89.5000000000000000
 Birmingham | UK          | 88.0000000000000000
 
(6 rows)
