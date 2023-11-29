# Pizza-Runner
SQL Case Study #2

### Project Overview:
Danny was sold on the idea, but he knew that pizza alone was not going to help him get seed funding to expand his new Pizza Empire - so he had one more genius idea to combine with it - he was going to Uberize it - and so Pizza Runner was launched!

More information about this case study: https://8weeksqlchallenge.com/case-study-2/


### Entity Relationship Diagram: 

Link: https://dbdiagram.io/d/Pizza-Runner-5f3e085ccf48a141ff558487?utm_source=dbdiagram_embed&utm_medium=bottom_open


### Data cleaning:

Before i start writing my SQL queries however - i want to investigate the data, i want to do something with some of those null values and data types in the customer_orders and runner_orders tables!


#### update and insert order_items

```sql

-- add id to table
ALTER TABLE customer_orders
ADD COLUMN id SERIAL PRIMARY KEY;
SELECT order_id,

-- add order_items column
ALTER TABLE customer_orders
ADD COLUMN order_items INTEGER;

-- add order_items number to its field
UPDATE customer_orders AS co 
SET order_items = subquery.item_number 
FROM
	( SELECT order_id, ID, ROW_NUMBER ( ) OVER ( PARTITION BY order_id ORDER BY ID ) AS item_number FROM customer_orders ) AS subquery 
WHERE
	co.ID = subquery.ID;
```

#### update the 'distance' field in the runner_orders

```sql
UPDATE runner_orders
SET distance = regexp_replace(distance, '[^\d.]+', '', 'g')
WHERE distance IS NOT NULL;
```

#### update the 'duration' field in the runner_orders

```sql
UPDATE runner_orders
SET duration = regexp_replace(duration, '[^\d]+', '', 'g')
WHERE duration IS NOT NULL;
```

#### create and store to order_extras

```sql
CREATE TABLE order_extras (
  "order_id" INTEGER,
	"order_items" INTEGER,
  "pizza_id" INTEGER,
  "ingredient_id" INTEGER
);


INSERT INTO order_extras ("order_id", "pizza_id","order_items","ingredient_id")
SELECT "order_id", "pizza_id","order_items", regexp_split_to_table("extras", ', ')::INTEGER
FROM customer_orders
WHERE "extras" IS NOT NULL
AND "extras" != ''
AND "extras" != 'null';
```

#### create and store to order_exclusions

```sql
CREATE TABLE order_exclusions (
  "order_id" INTEGER,
	"order_items" INTEGER,
  "pizza_id" INTEGER,
  "ingredient_id" INTEGER
);


INSERT INTO order_exclusions ("order_id", "pizza_id","order_items","ingredient_id")
SELECT "order_id", "pizza_id","order_items", regexp_split_to_table("exclusions", ', ')::INTEGER
FROM customer_orders
WHERE "exclusions" IS NOT NULL
AND "exclusions" != ''
AND "exclusions" != 'null';
```


### Case Study Questions

#### A. Pizza Metrics


##### For each customer, how many delivered pizzas had at least 1 change and how many had no changes?

```sql
SELECT 
-- *
sub2.customer_id,
sum(
case when sub2.changes='*'
then 1
ELSE 0
END
) as had_change,
sum(
case when sub2.changes='-'
then 1
ELSE 0
END
) as no_change

FROM
(
SELECT
sub.id,
sub.customer_id,
sub.order_items,
(
case WHEN count(DISTINCT ex_order_id) = 1 or count(DISTINCT exclusions_order_id) = 1
THEN '*'
ELSE '-'
END
)as changes
FROM
(SELECT 
customer_orders.id,
customer_orders.order_id,
customer_orders.customer_id,
customer_orders.order_items,
order_extras.order_id as ex_order_id,
order_exclusions.order_id as exclusions_order_id
FROM 
customer_orders
LEFT JOIN order_extras
ON customer_orders.order_id=order_extras.order_id
AND customer_orders.order_items=order_extras.order_items

LEFT JOIN order_exclusions
ON customer_orders.order_id=order_exclusions.order_id
AND customer_orders.order_items=order_exclusions.order_items) as sub
GROUP BY sub.id,sub.customer_id,sub.order_items) as sub2
GROUP BY sub2.customer_id
;
```

##### How many pizzas were delivered that had both exclusions and extras?

```sql
SELECT
count(DISTINCT customer_orders.id) as pizzas_count
FROM 
customer_orders
INNER JOIN order_extras 
ON customer_orders.order_id=order_extras.order_id
AND customer_orders.order_items=order_extras.order_items

INNER JOIN order_exclusions
ON customer_orders.order_id=order_exclusions.order_id
AND customer_orders.order_items=order_exclusions.order_items;
```

##### What was the total volume of pizzas ordered for each hour of the day?

```sql
SELECT
extract(hour FROM order_time) as hour,
count(*) as total_volume
FROM
customer_orders
GROUP BY extract(hour FROM order_time)
;
```







































