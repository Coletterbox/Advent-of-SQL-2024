# Day 3: The Greatest Christmas Dinner Ever! 🍗

## Skills Used: CTE, parsing XML

----

## Task Info

### Table Schemas

```sql
DROP TABLE IF EXISTS christmas_menus CASCADE;

CREATE TABLE christmas_menus (
  id SERIAL PRIMARY KEY,
  menu_data XML
);
```

### Example Data

Version 1
```sql
INSERT INTO christmas_menus (id, menu_data) VALUES
(1, '<menu version="1.0">
    <dishes>
        <dish>
            <food_item_id>99</food_item_id>
        </dish>
        <dish>
            <food_item_id>102</food_item_id>
        </dish>
    </dishes>
    <total_count>80</total_count>
</menu>');
```

Version 2
```sql
INSERT INTO christmas_menus (id, menu_data) VALUES
(2, '<menu version="2.0">
    <total_guests>85</total_guests>
    <dishes>
        <dish_entry>
            <food_item_id>101</food_item_id>
        </dish_entry>
        <dish_entry>
            <food_item_id>102</food_item_id>
        </dish_entry>
    </dishes>
</menu>');
```

Version that shouldn't be parsed (because there must have been more than 78 guests at this event)
```sql
INSERT INTO christmas_menus (id, menu_data) VALUES
(3, '<menu version="beta">
  <guestCount>15</guestCount>
  <foodList>
      <foodEntry>
          <food_item_id>102</food_item_id>
      </foodEntry>
  </foodList>
</menu>');
```

----

## Solution

### Query

```sql
with info as (
  select (xpath('//total_guests/text()', menu_data)::varchar[]::integer[])[1] as guests,
    (xpath('//total_count/text()', menu_data)::varchar[]::integer[])[1] as guests2,
    (xpath('//guestCount/text()', menu_data)::varchar[]::integer[])[1] as guests3,
    (xpath('//food_item_id/text()', menu_data))::varchar[] as food_ids
  from christmas_menus
)
select unnest(food_ids) as item, count(*) from info
where guests > 78 or guests2 > 78 or guests3 > 78
group by item
order by count desc
limit 1;
```

### Output

| item | count |
------|-------
 493  |   117
 
(1 row)
