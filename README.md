# Pizza-Runner
SQL Case Study #2

### Project Overview:
Danny was sold on the idea, but he knew that pizza alone was not going to help him get seed funding to expand his new Pizza Empire - so he had one more genius idea to combine with it - he was going to Uberize it - and so Pizza Runner was launched!

More information about this case study: https://8weeksqlchallenge.com/case-study-2/


### Entity Relationship Diagram: 

Link: https://dbdiagram.io/d/Pizza-Runner-5f3e085ccf48a141ff558487?utm_source=dbdiagram_embed&utm_medium=bottom_open


### Data cleaning:

Before i start writing my SQL queries however - i want to investigate the data, i want to do something with some of those null values and data types in the customer_orders and runner_orders tables!


##### update and insert order_items

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











