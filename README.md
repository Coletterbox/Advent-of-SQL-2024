https://wiki.termux.com/wiki/Postgresql

# Advent of SQL 2024

## In which: Santa and his elves are just as helpless as in Advent of Code

And who is the reader supposed to be in these things, anyway? Do we work for Santa? Is it a seasonal job? Would a person with this job list it on their CV?

---

## Solutions:

### Example: The Great Christmas Analytics Crisis of 2024 (intermediate)

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

### Day 1: Santa's Gift List Parser (beginner)

#### Database Structure:

```sql
  CREATE TABLE children (
      child_id INT PRIMARY KEY,
      name VARCHAR(100),
      age INT
  );
  CREATE TABLE wish_lists (
      list_id INT PRIMARY KEY,
      child_id INT,
      wishes JSON,
      submitted_date DATE
  );
  CREATE TABLE toy_catalogue (
      toy_id INT PRIMARY KEY,
      toy_name VARCHAR(100),
      category VARCHAR(50),
      difficulty_to_make INT
  );
```

return top 5 for: name,primary_wish,backup_wish,favorite_color,color_count,gift_complexity,workshop_assignment
 (no spaces)

#### Solution:

```sql
\pset format unaligned
   \pset fieldsep ','

with gift_list as (
  select name,
    wishes ->> 'first_choice' as primary_wish,
    wishes ->> 'second_choice' as backup_wish,
    wishes ->> 'colors' as favourite_colours,
    pg_typeof(wishes ->> 'colors')
  from children c
  left join wish_lists w on c.child_id = w.child_id
), l2 as (
  -- select pg_typeof(favourite_colours) from gift_list
  select name,
    primary_wish,
    backup_wish,
    string_to_array(favourite_colours, '","') as colour_array,
    case when difficulty_to_make = 1 then 'Simple Gift'
      when difficulty_to_make = 2 then 'Moderate Gift'
      else 'Complex Gift' end
      as gift_complexity,
    case when category like 'outdoor' then 'Outside Workshop'
      when category like 'educational' then 'Learning Workshop'
      else 'General Workshop' end
      as workshop_assignment
  from gift_list
  left join toy_catalogue on primary_wish = toy_name
  order by name asc
  limit 5
)
select name,
  primary_wish,
  backup_wish,
  case when colour_array[1] like '%Blue%' then 'Blue'
    when colour_array[1] like '%Purple%' then 'Purple'
    when colour_array[1] like '%Pink%' then 'Pink'
    when colour_array[1] like '%Yellow%' then 'Yellow'
    when colour_array[1] like '%White%' then 'White'
    when colour_array[1] like '%Red%' then 'Red'
    when colour_array[1] like '%Brown%' then 'Brown'
    when colour_array[1] like '%Green%' then 'Green'
    when colour_array[1] like '%Black%' then 'Black'
    when colour_array[1] like '%Orange%' then 'Orange'
    else colour_array[1] end as favorite_color,
  array_length(colour_array, 1) as color_count,
  gift_complexity,
  workshop_assignment
from l2;
```

(early version of table)
|   name    |   primary_wish   |   backup_wish   | gift_complexity | workshop_assignment |
-----------|------------------|-----------------|-----------------|---------------------
 Anastasia | Toy kitchen sets | Barbie dolls    |               4 | General Workshop
 Giuseppe  | LEGO blocks      | Building sets   |               3 | Learning Workshop
 Wendy     | LEGO blocks      | Toy trains      |               3 | Learning Workshop
 Winnifred | Building sets    | Rubiks cubes    |               4 | Learning Workshop
 Kayla     | Toy trucks       | Stuffed animals |               2 | General Workshop
 
(5 rows)

name,primary_wish,backup_wish,gift_complexity,workshop_assignment\
Abagail,Building sets,LEGO blocks,Blue,1,Complex Gift,Learning Workshop
Abbey,Stuffed animals,Teddy bears,White,4,Complex Gift,General Workshop
Abbey,Toy trains,Toy trains,Pink,2,Complex Gift,General Workshop
Abbey,Barbie dolls,Play-Doh,Purple,1,Moderate Gift,General Workshop
Abbey,Yo-yos,Building blocks,Blue,5,Simple Gift,General Workshop

### Day 2: Santa's Jumbled Letters (beginner)

#### Database Structure:

```sql
-- Binky's Table
CREATE TABLE letters_a (
    id SERIAL PRIMARY KEY,
    value INTEGER
);

-- Blinky's Table
CREATE TABLE letters_b (
    id SERIAL PRIMARY KEY,
    value INTEGER
);
```

#### (Part of) Example Data:

```sql
-- Binky's data (letters_a)
INSERT INTO letters_a (id, value) VALUES
(1, 68),    -- D
(2, 101),   -- e
(4, 97),    -- a
(5, 114),   -- r
(6, 32),    -- (space)
(7, 83),    -- S
(8, 35),    -- # (noise)
```

```sql
-- Blinky's data (letters_b)
INSERT INTO letters_b (id, value) VALUES
(23, 32),   -- (space)
(24, 36),   -- $ (noise)
(25, 108),  -- l
(26, 105),  -- i
(27, 107),  -- k
(28, 101),  -- e
(29, 32),   -- (space)
(30, 97),   -- a
```

#### Solution:

```sql
with selection as (
  select value from letters_a
  union all 
  select value from letters_b
),
content as (
select chr(value) as c from selection
  where (value > 96 and value < 123)
    or (value > 64 and value < 91)
    or value = 33
    or value = 39
    or value = 40
    or value = 41
    or value = 44
    or value = 45
    or value = 46
    or value = 58
    or value = 59
    or value = 63
    or value = 32
)
select string_agg(c, '') from content;
```

#### Output:

string_agg
Dear Santa, I hope this letter finds you well in the North Pole! I want a SQL course for Christmas!
(1 row)
