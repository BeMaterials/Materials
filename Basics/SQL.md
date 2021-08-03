# Basics

- To use a database:

```sql
USE db_name;
```

- In `SQL`, `White spaces` are not read.

- With `--`, you can comment a line:

```sql
-- This is a comment
```

## SELECT statement

- The order is important:

```sql
SELECT *
FROM table_name
WHERE condition1
GROUP BY column_name1
HAVING condition2
ORDER BY column_name2;
```

For example:

```sql
SELECT first_name, last_name
FROM customers;
```

```sql
SELECT *
FROM customers
WHERE customer_id = 1
ORDER BY first_name;
```

### Alias

```sql
SELECT points, (points + 10) * 100 AS new_points
FROM customer;
```

If you want space in the `alias` column:

```sql
SELECT points, (points + 10) * 100 AS 'new points'
FROM customer;
```

### SELECT DISTINCT

- The following removes the duplicate entries from the result:

```sql
SELECT DISTINCT state
FROM customers;
```

### WHERE clause

```sql
WHERE state = 'va';              -- string search is case-insensitive
WHERE points = 15;
WHERE points != 15;              -- the same as line below
WHERE points <> 15;
WHERE birth_date > '1990-01-01'; -- we put dates in '' as well as strings
```

#### Logical operators

```sql
WHERE cond1 AND cond2;
WHERE cond1 OR cond2;
WHERE NOT cond; -- like ! operator in C#
```

- When combining the conditions, note that the `AND` operator takes precedence over `OR`. So it is better to use `()`.

#### IN operator

- Instead of using multiple `OR`, use `IN` operator:

```sql
WHERE state IN ('va', 'fl', 'ga');
WHERE state NOT IN ('va', 'fl', 'ga');
```

#### BETWEEN operator

- Instead of combining two related conditions with `AND`, use `BETWEEN` operator:

```sql
WHERE points BETWEEN 1000 AND 2000;
```

- It is used for dates too.
- Both margins are `inclusive`.

#### LIKE operator

- Is used to search patterns in a text:

```sql
WHERE last_name LIKE '%b%'; -- % represents any number of characters
WHERE last_name LIKE 'b_';  -- _ represents only one character
```

- The whole thing becomes a condition, so:

```sql
WHERE last_name NOT LIKE '%b%' AND age > 18;
```

- Note that we don't have `REGEXP`in `SQL Server`.

#### NULL

```sql
WHERE phone IS NULL; -- = NULL is not working
WHERE phone IS NOT NULL;
```

### ORDER BY clause

- For reversing the order:

```sql
ORDER BY first_name DESC;
```

- You can have cascading sort:

```sql
ORDER BY state, first_name DESC; -- The order is important
```

- The expression in front of `ORDER BY` can be a mathematical equation calculated based on columns or even aliases.

#### OFFSET-FETCH

- It can only be used when we have `ORDER BY`:

```sql
ORDER BY points
OFFSET 6 ROWS
FETCH NEXT 3 ROWS ONLY;
```

- If you don't want `OFFSET` just put `0`. You can't delete it.
- In `Postgres`, you can use them without `ORDER BY`:

```sql
SELECT *
FROM products
ORDER BY price DESC -- not necessary and can be removed
LIMIT 5   --Only return 5 records
OFFSET 10; --Jump over 10 rows
```

## JOINS

### INNER JOIN

- In `INNER JOIN`, it fetches the data on `ON` condition.
- `INNER JOIN` and `JOIN` is identical. So we use the latter.

```sql
SELECT *
FROM orders
JOIN customers
ON orders.customer_id = customers.customer_id;
```

- To shorten the above syntax, we use aliases:

```sql
SELECT *
FROM orders o
JOIN customers c
ON o.customer_id= c.customer_id;
```

- In case of `self-join`, we have to use aliases:

```sql
SELECT e.first_name, m.first_name AS manager
FROM employees e
JOIN employees m
ON e.reports_to = m.employee_id;
```

- Having consecutive joins:

```sql
SELECT *
FROM orders o
JOIN customers c
ON o.customer_id= c.customer_id;
JOIN addresses a
ON c.address_id= a.address_id;
```

- If a table has a composite primary key of two columns, we have two conditions in front of `ON`.

### LEFT JOIN

- In `LEFT JOIN`, it fetches all the records from the first table and the matched records from the second table. If there is no match in the second table the result will be NULL.
- All the customers, and if they have any order, the order details (If a customer has many orders, there will be many rows with the same customer info but different orders info):

```sql
SELECT *
FROM customers c
LEFT JOIN orders o
ON c.customer_id = o.customer_id;
```

- DO NOT USE <del>RIGHT JOIN</del> as it is exactly like `LEFT JOIN` but we have to swap the order of tables!
- DO NOT USE <del>OUTER JOIN</del> as well. It doesn't mean to return all the customers and all the orders.
- In `LEFT JOIN` order does matter but generally not in `JOIN` (sometimes when there are multiple joins, it matters).

### CROSS JOIN

- It joins all the records of one table to all the records of the other table:

```sql
SELECT *
FROM colors c
CROSS JOIN sizes s
ORDER BY s.size_id;
```

## UNION

- In `UNION`, the number of fields in two tables should be equal.
- The name of the fields will be set from the first table.

```sql
SELECT *, 'Bronze' AS customer_class
FROM customers
WHERE points < 200

UNION

SELECT *, 'Silver' AS customer_class
FROM customers
WHERE points BETWEEN 200 AND 300

UNION

SELECT *, 'Gold' AS customer_class
FROM customers
WHERE points > 300;
```

- Also, when we have a query which is challenging to be written in one query -> you can use it.
- There will be no duplicated row in the result of union. If we want duplicates, use `UNION ALL` (like when we want to unite the results of photo_tags and caption_tags).
- If there is `GROUP BY` or `LIMIT` in each query, we should use parenthesis -> because it doesn't know if it is for the results of the UNION or each query.
- We also have `INTERSECT` which only returns the shared records of two queries.
- We also have `EXCEPT` which is like deducting one set from another.
- The above example can be written with `CASE`:

```sql
SELECT *,
    CASE
        WHEN points < 200 THEN 'Bronze'
        WHEN points BETWEEN 200 AND 300 THEN 'Silver'
        ELSE 'Gold' --ELSE is not necessary
    END
FROM customers;
```

## INSERT statement

```sql
INSERT INTO customers ('John', 'Smith', '1990-01-01', NULL, 'a', 'b', 'c', DEFAULT); -- That field is probably nullable and the DEFAULT will set the field to the default value that was given at the time of creating the table
```

- Note that if the `Identity Specification` of the first column (id) is set to `Yes` in the `Table design`, you should not include it in the above statement. BUT, if you want to set it in any case:

```sql
SET IDENTITY_INSERT customers ON;
INSERT INTO customers (customer_id, first_name, last_name, birth_date, phone, address, city, state, points)
VALUES (14, 'John', 'Smith', '1990-01-01', NULL, 'a', 'b', 'c', DEFAULT);
```

- If we don't want to set the id and things that are nullable or have a default value:

```sql
INSERT INTO customers (first_name, last_name, birth_date, address, city, state)
VALUES ('Jack', 'Smith', '1980-01-01', 'a', 'b', 'c');
```

- If we want to insert multiple records at the same time:

```sql
INSERT INTO customers (first_name, last_name, birth_date, address, city, state)
VALUES ('John', 'Smith', '1990-01-01', 'a', 'b', 'c'), ('Jack', 'Smith', '1980-01-01', 'a', 'b', 'c');
```

- If you want to use the last id generated by `SQL Server`, you can use this function:

```sql
SCOPE_IDENTITY();
```

## Copy a table into another table

- Note that the `DEFAULT` values and `primary key`s won't be set.

```sql
SELECT * INTO new_table
FROM old_table;
```

## UPDATE statement

```sql
UPDATE customer
SET first_name = 'Jane', last_name = 'Doe'
WHERE customer_id = 1;
```

- You can set the field value to `NULL` or `DEFAULT` too.
- You can use other columns' values to set in a column.
- In `SQL Server`, it is ok to update multiple values; if the `WHERE` clause returns multiple values.

## DELETE statement

```sql
DELETE FROM customers; --BE CAREFUL! It wipes out the table -> Always use a WHERE clause
```

## AGGREGATE functions

- The aggregate functions can be used on numbers, dates, and strings.

```sql
SELECT MAX(age), COUNT(age), SUM(age) AS total, MIN(age) AS youngest, AVG(age)
FROM employees;
```

- If a value is NULL, it will be skipped in the aggregate functions.
- If we want to know the total number of the records:

```sql
SELECT COUNT(*)
FROM employees;
```

- If we want to know the total number of unique values of a column:

```sql
SELECT COUNT(DISTINCT state)
FROM customers;
```

### GROUP BY clause

- If we want to know the total amount of money each customer spent:

```sql
SELECT client_id, SUM(total_invoice) as total
FROM invoices
GROUP BY client_id
ORDER BY total;
```

#### GROUP BY rules

- It can be used ONLY when we have aggregate functions.
- In front of `GROUP BY`, we have to list all the columns in the `SELECT` CLAUSE (not the ones inside aggregate functions). In other words, we can only `SELECT` things that are in front of `GROUP BY` or we can use aggregate functions.
- We can add more columns in the `GROUP BY` clause (and not add it in the `SELECT` CLAUSE).
- The columns in front of `GROUP BY` means that their combination is unique. (It can be just one like the example above)
- If you have only one column after `GROUP BY column_name`, you can use `WITH ROLLUP` to add another row to the result and perform the aggregate function again on all the result:

```sql
SELECT state, avg(age)
FROM customers
GROUP BY state WITH ROLLUP;
-- 'CA' 20
-- 'FL' 30
-- NULL 25
```

### HAVING clause

- Is used to filter data after GROUPING.
- We only can use aggregate functions (or their aliases) in the HAVING clause.

```sql
SELECT client_id, SUM(total_invoice) as total
FROM invoices
GROUP BY client_id
HAVING total > 100
ORDER BY total;
```

# Database Design

## Data types

- String types:

```sql
CHAR(2)      -- Fixed length. If we gave it only one character, the second character will be ' '
VARCHAR(5)   --Variable length. It is ok to give less characters.
VARCHAR(50)  --For short strings
VARCHAR(255) --For medium-sized strings
VARCHAR(max) --For large strings like JSON, CSV
TEXT         --For very large strings
```

- The followings support `Unicode` as well. So if you want to add support for another languages, use these:

```sql
NCHAR(2)
NVARCHAR(5)
NVARCHAR(50)
NVARCHAR(255)
NVARCHAR(max)
NTEXT
```

- Numeric data types:

```sql
TINYINT  --0-255
SMALLINT --+/-32000
INT      --+/-2.1B
BIGINT   --...

--For numbers with exact value like money
DECIMAL(9,2) --The first arg is the total digits (1-38), and the second one is number of digits after `.` (0-first arg).

--Has scientific form (for storing very big or very small numbers)
FLOAT
REAL
```

- Boolean types:

```sql
BIT --0-1
```

- Dates and times types:

```sql
DATE
TIME
DATETIME
TIMESTAMP
```

- Binary Large Object, `Blob`, types (which is a very bad practice):

```sql
BINARY(n)     --up to 8 kilobytes
VARBINARY(n)  --up to 8 kilobytes
VARBINARY(max)--up to 2 Gigabytes
IMAGE         --up to 2 Gigabytes
```

- In `SQL Server`, we don't have `enum` types; instead we store that constants in another table (`look-up table`).

## Design process

### Requirement gathering

### Conceptual model

- We use `ER` diagrams to show entities, their attributes, and their relationships:
  ![](/md/erd.jpg)

### Logical model

- It has more details than the `conceptual method`.
- The type of each attribute has been specified. For example, `name (string)`. We won't ude `varchar(50)`, because we don't want to couple this model with a specific `DBMS`.
- We create a `new entity` for `many-to-many` relationships and we give it its attributes. Therefore, we have two `one-to-many` relationships instead.

### Physical model

- In fact, it is the implementation of logical model on a specific `DBMS`.
- Use `plural` names for tables.
- Use `_` for column names.
- Do not prefix column names with table name: <del>enrollment_date</del>.
- `email` is a bad choice for `primary key` because it is too long to be `pk`.
- Every `one-to-many` relationships means a `foreign key`.
- Do not use a `composite primary key` in a parent table. Because you have to pass it as a foreign key to its children. So define an `id` as a `pk` and use `UNIQUE` constraint for those two columns.
- In the `Foreign Key Relationships` window, in the `INSERT AND UPDATE Specification` section, we have two sections for `Delete rule` and `Update rule` which can be `No Action` and `Cascade`:
  - `Cascade on Update` means that if `pk` has changed (which is very unlikely!), `fk` in the `child` table has to change too.
  - `Cascade on delete` means that if parent record is deleted, all its children should be deleted too (very dangerous!). This option is `bad`!
  - `No Action on Update` means that it won't allow the parent `pk` change if it has children.
  - `No Action on delete` means that it won't allow the parent record be deleted if it has children.
- It is common to use: `Cascade on Update` and `No Action on delete`.

## Normalization forms

### 1NF: First Normal Form

- Each cell only stores one piece of data.
- We also shouldn't define columns like `tag1`, `tag2`, ...
- Instead, we create a table for `tags` with `tag_id` but because we have `many-to-many` relationships, we create a `link table` called `course_tags`, pass `course_id` and `tag_id` as `fk`s and make the combination of these two as a `composite pk`.

### 2NF: Second Normal Form

- There is no column which can be determined by a `candidate key`. If there is we have to extract another table.

### 3NF: Third Normal Form

- There is no column which can be determined by `non-prime` columns. For example, a column is a summation of two other columns.
- This is bad because we might forget to update the third column and our table will be in a bad state (inconsistent).

## CREATE and DROP database

```sql
CREATE DATABASE crr_prod;
```

```sql
DROP DATABASE IF EXISTS crr_prod;
```

## CREATE statement

```sql
CREATE TABLE customers (
    customer_id int PRIMARY KEY IDENTITY, --Like AI in MySQL
    name varchar(50) NOT NULL,
    points int NOT NULL DEFAULT 0 CHECK (points > 0), --Validation
    email varchar(255) NOT NULL UNIQUE,   --It will be added to keys as a constraint
);
```

```sql
CREATE TABLE orders (
    order_id int PRIMARY KEY IDENTITY,
    customer_id int NOT NULL,
    CONSTRAINT fk_orders_customers     --Convention: fk_childTable_parentTable
    FOREIGN KEY (customer_id)          --(column_name_in_the_present_table)
    REFERENCES customers(customer_id)  --parent_table(column_name_in_the_parent_table)
    ON UPDATE CASCADE ON DELETE NO ACTION --NO ACTION actually throws an error
    --ON DELETE CASCADE will delete the orders when the customer is deleted.
    --ON DELETE SET NULL will set the customer_id to NULL when the customer is deleted.
    --ON DELETE SET DEFAULT will set the customer_id to a default value if one is provided.
);
```

- In `Postgres`:

```sql
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    name varchar(50),
);
```

```sql
CREATE TABLE photos (
    id SERIAL PRIMARY KEY,
    url varchar(200),
    user_id INTEGER REFERENCES users(id)
);
```

## ALTER statement

### ADD

```sql
ALTER TABLE customers
ADD last_name varchar(50) NOT NULL;
```

- You can add many columns in one go. Just add `,` after `...NOT NULL` and keep writing.

### ALTER COLUMN

```sql
ALTER TABLE customers
ALTER COLUMN last_name varchar(60) NOT NULL;
```

```sql
ALTER TABLE customers
ALTER COLUMN points DEFAULT 1000;
```

```sql
ALTER TABLE products
ADD UNIQUE (name);
```

- For add uniqueness to multiple columns:

```sql
ALTER TABLE products
ADD CONSTRAINT UC_Product UNIQUE (name, department);
```

### DROP COLUMN

```sql
ALTER TABLE customers
DROP COLUMN last_name;
```

### Rename a column

```sql
sp_rename 'customers.name', 'first_name', 'COLUMN';
```

### ADD CONSTRAINT

- If you want to add a `foreign key`:

```sql
ALTER TABLE orders
ADD CONSTRAINT fk_orders_customers --Convention: fk_childTable_parentTable
FOREIGN KEY (customer_id)          --(column_name_in_the_present_table)
REFERENCES customers(customer_id); --parent_table(column_name_in_the_parent_table)
```

### Remove a constraint

- If you want to remove a `foreign key`:

```sql
ALTER TABLE orders
DROP CONSTRAINT fk_orders_customers;
```

## Indexing

- When you index a column, the `DBMS` will store that column (along with the primary key) in another place. And when we want to read it, it looks it up in like a `sorted contact list`, retrieves the `pk` and then quickly retrieves the record.
- Indexing is costly (requires storage space and it takes more time during insert/update/delete). So we use it only for critical and slow queries.
- Be aware that sometimes (specially when we query for majority of data), the index will not even be used! It is because the database will plan a query and calculate a estimate of the cost (in terms of time) for the query to run and if we have an index, it will compare it (its estimated scored) to the sequential scan and decides how to perform the query (because there is some random hard disk access when using index and its cost is higher compared to sequential access -> sometimes the index won't be used at all). So It is better you add `EXPLAIN` keyword to your query (before select) and make sure that the database will use your index, otherwise `DROP` the index. With `EXPLAIN ANALYZE` you can see the execution time of your query.
- `primary keys` and columns that have `UNIQUE` constraints are automatically indexed.
- Other indexes have `primary key` in them. So if you just query for the `pk`, it doesn't even look into the table.
- `foreign keys` are also automatically indexed to boost the `JOIN`ing process.

### CREATE INDEX

```sql
CREATE INDEX idx_state ON customers(state);
```

### Composite index

- In the real world, we use `composite indexes` most of the time (usually 4 to 6):

```sql
CREATE INDEX idx_state_points ON customers(state, points);
```

- In a `composite index`, **it is better** to order the columns with `cardinality` (the number of unique values).
- If a column has a larger cardinality, it should come before a column with smaller cardinality; because it narrows down the search more quickly. For example the cardinality of `gender` is two so it should come after `state`. To get the cardinality of a column:

```sql
SELECT COUNT(DISTINCT state)
FROM customers;
```

- Remember that the rule above is highly dependent on the query itself. In the following query, it is better that the `state` comes before `last_name` even though it has less cardinality:

```sql
SELECT *
FROM customer
WHERE state = 'NY' AND last_name LIKE 'A%';
```

- If we want to **`force index`**:

```sql
SELECT *
FROM customer
WITH (INDEX(idx_last_name_state))
WHERE state = 'NY' AND last_name LIKE 'A%';
```

- **Another rule** for designing the order of the columns in a composite index is that the column that more queries has been written based on, should come before the others; because for simpler queries you can easily use a `composite index` with no problem. `idx_state_last_name` is perfectly suitable for searching based on `state` only.

- It is also important to design composite indexes in a way that they are suitable both for `filtering` and `sorting`. Because `sorting` is a very expensive operation. So the **rule** is that the order should be the same as in the `ORDER BY` clause, otherwise the index won't be used for sorting. If one of them is `DESC`, the index won't be used. If all of the is `DESC`, the index will be used.
- It is very good that in the `SELECT` clause only use the columns that exist in the composite index. This way, it doesn't even look into the table.
- In summary:
  - Look at `WHERE` clauses and select the ones that are very frequent.
  - Then, look at `ORDER BY` clauses and see that if you can include them in the index.
  - Finally, look at the `SELECT` clause to include them.
- Examine the indexes and remove `redundant` or `duplicated` ones:

```sql
(A, B)
(A)    -->Is redundant
(B)    -->Is NOT redundant
```

- Sometimes you have to modify the queries based on the indexes that you have:

```sql
SELECT *
FROM customers
WHERE state = 'ca' OR points > 1000;
```

is better to be written:

```sql
SELECT *
FROM customers
WHERE state = 'ca';
UNION
SELECT *
FROM customers
WHERE points > 1000;
```

- Or in this case, it checks the condition for every record which is very time-consuming:

```sql
SELECT *
FROM customers
WHERE  points + 10 > 1000;
```

and should be written:

```sql
SELECT *
FROM customers
WHERE  points > 990;
```

# Advanced

## Sub-queries

```sql
SELECT name, price
FROM products
WHERE price > (
    SELECT MAX(price)
    FROM products
    WHERE department = 'Toys'
);
```

- Sub-queries can be placed as a value (`SELECT COUNT(*) FROM orders;`), values (source of a column: `SELECT id FROM orders;`), or source of rows (`SELECT * FROM orders;`).

```sql
SELECT
    (SELECT MAX(price) FROM products) AS max_price,
    (SELECT MIN(price) FROM products) AS min_price,
    (SELECT AVG(price) FROM products) AS avg_price;
```

- Instead of VALUES in the INSERT statement, we can use the result of any arbitrary queries:

```sql
INSERT INTO table_name
SELECT * FROM another_table; -- It doesn't have to be a simple query, it can be a join,... but it has to start with SELECT
```

- Whenever you put a sub-query in front of a `FROM`, you should give it an alias:

```sql
SELECT * INTO new_table
FROM (SELECT *
      FROM old_table
      JOIN another_table
      ON cond1
      WHERE cond2) AS seeder;
```

- You can use sub-queries in an expression but you have to be careful about the number of records that it returns:

```sql
UPDATE invoices
SET payment_total = invoice_total * 0.5
WHERE client_id = (
    SELECT client_id
    FROM clients
    WHERE first_name = 'Ben'); --Just one value!
```

- If we have used `IN` operator:

```sql
UPDATE invoices
SET payment_total = invoice_total * 0.5
WHERE client_id IN ( --this is where sub-queries are really useful -> when they return a source for a column
--in this way, they can be used instead of a JOIN
    SELECT client_id
    FROM clients
    WHERE first_name = 'Ben'); --Can return any number of records
```

- Or in this case:

```sql
UPDATE invoices
SET payment_total = invoice_total * 0.5
WHERE age > (
    SELECT age
    FROM clients
    WHERE client_id = 1); --Just one value!
```

- If we wanted to return multiple values from the `sub-query`:

```sql
UPDATE invoices
SET payment_total = invoice_total * 0.5
WHERE age > ANY ( --SOME is exactly like ANY
--We also have similar concept > ALL
    SELECT age
    FROM clients
    WHERE state = 'ca'); --Can return any number of records!
```

- The `IN` operator is identical to `= ANY`.

### Correlated sub-queries

- Sub-queries that is dependent on the outer query (like find the employees that have greater salary than the average of the department they are belonging to):
- Think of the `correlated sub-queries` as `double-nested for-loops`.

```sql
SELECT *
FROM employees e
WHERE salary > (
    SELECT AVG(salary)
    FROM employees
    WHERE office_id = e.office_id
);
```

- With correlated sub-queries, we can re-write this:

```sql
SELECT *
FROM clients
WHERE client_id IN (
    SELECT client_id
    FROM invoices;
);
```

to:

```sql
SELECT *
FROM clients c
WHERE EXISTS (
    SELECT client_id
    FROM invoices
    WHERE client_id = c.client_id
); -- here, the sub-query runs first and if it returns something, the outer condition will be true
```

## Common Table Expression

- It helps us improve the readability by defining a temporary table _before_ the main query:

```sql
WITH tags AS {
    SELECT user_id, created_at FROM caption_tags
    UNION ALL
    SELECT user_id, created_at FROM photo_tags
}

SELECT username, tags.create_at
FROM users
JOIN tags ON tags.user_id = users.id
WHERE tags.created_at > '2020-01-01';
```

### Recursive CTE

- The recursive query will be run again and again and the result will be appended to the end of the resulting table.

```sql
WITH RECURSIVE countdown(val) AS ( --every variable in the () will be a column name in the resulting table
    SELECT 3 AS val                --Initial, Non-recursive query
    UNION                          --there will always be a UNION
    SELECT val - 1 FROM countdown WHERE val > 1 -- Recursive query. The countdown table here refers to the last table produced (here it is just a row)
)

SELECT *
FROM countdown;

--val
--3
--2
--1
```

- Think of `suggestions` list in Instagram. It is basically followings of our followings. So the first generation can be derived from a simple join but what if we exhaust that list? It should go one level deeper and show followings of followings of followings. Basically when we deal with `tree` or `graph` (in graph each node can different direction of relation) types of data structure, we usually need recursive CTEs:

```sql
WITH RECURSIVE suggestions(leader_id, follower_id, depth) AS (
        SELECT leader_id, follower_id, 1 AS depth
        FROM followers
        WHERE follower_id = 100 -- we want to build this suggestion table for a specific user
    UNION
        SELECT f.leader_id, f.follower_id, depth + 1
        FROM followers f
        JOIN suggestions s ON s.leader_id = f.follower_id
        WHERE depth < 3
)

SELECT DISTINCT users.id, users.username
FROM suggestions
JOIN users ON users.id = suggestions.leader_id
WHERE depth > 1
LIMIT 30;
```

## Built-in functions

```sql
SELECT ROUND(5.73, 1); --> 5.7
SELECT CEILING(6.2); --> 7
SELECT FLOOR(6.7); --> 6
SELECT ABS(-1.1); --> 1.1
SELECT RAND(); --> 0.628103045171397
SELECT LEN('sky'); --> 3
SELECT UPPER('sky'); --> 'SKY'
SELECT LOWER('SKY'); --> 'sky'
SELECT TRIM(' ben   '); --> 'ben'
SELECT LTRIM(' ben   '); --> 'ben  '
SELECT RTRIM(' ben   '); -->' ben'
SELECT LEFT('Ben Abe', 3); --> 'Ben'
SELECT RIGHT('Ben Abe', 3); --> 'Abe'
SELECT SUBSTRING('John', 4, 3); --> 'nam' :index (1-based) and size
SELECT REPLACE('Ben', 'b', 'a'); --> 'aen'
SELECT CONCAT('Ben', ' ', 'Abe'); --> 'Ben Abe'
SELECT GETDATE(); --> 2020-04-15 10:58:51.597
SELECT CURRENT_TIMESTAMP; --> 2020-04-15 10:58:51.597
SELECT YEAR('2020-01-01'); --> 2020
SELECT MONTH(CURRENT_TIMESTAMP); --> 4
SELECT DAY(CURRENT_TIMESTAMP); --> 15
SELECT FORMAT(GETDATE(), 'dd-MM-yy'); --> 15-04-20
SELECT DATEADD(DAY, 1, GETDATE()); --> 2020-04-16 10:58:53.543
SELECT DATEDIFF(DAY, '2020-01-01', GETDATE()); --> 105

--Maybe just Postgres
SELECT name, weight, GREATEST(2 * weight, 30) FROM products; --> if shipping cost is below 30$, make it 30$
SELECT LEAST(1, 20, 5); --> 1
```

```sql
SELECT first_name, ISNULL(invoice_total, 0) -- if it is null, returns 0
FROM customers;

SELECT first_name, COALESCE(invoice_total, age) -- returns the first non-null value
FROM customers;

SELECT first_name, IIF(YEAR(GETDATE()) - YEAR(birth_date) > 35, 'Old', 'Young' ) AS 'Age Group'
FROM customers;
```

## Views

- With views, we can store complicated queries or sub-queries or very repetitive ones and have code-reuse.
- It is recommended to write the app based on the `VIEW`s and not `TABLE`s. Because in this way, you can change the table but keeps the view intact and so the queries against the view is still valid.
- Views are also good for security. Because we can limit the user to certain columns or records.

```sql
CREATE VIEW sales_by_client AS
SELECT c.client_id, c.name, SUM(invoice_total) AS total
FROM clients c
JOIN invoices i
ON c.client_id = i.client_id
GROUP BY c.client_id, name;
```

- After executing the script, a view will be created in the `Views` folder and is accessible like other tables.
- Views are like `virtual` tables but do not store any data.
- Another example of a very useful view is when we think we made a mistake in our design but changing it will be very challenging:

```sql
CREATE VIEW tags AS (
    SELECT id, created_at, user_id, post_id, 'photo_tag' AS type FROM photo_tags
    UNION ALL
    SELECT id, created_at, user_id, post_id, 'caption_tag' AS type FROM caption_tags
)
```

### Dropping a view

```sql
DROP VIEW sales_by_client;
```

### Changing a view

- Instead of `CREATE`, use `CREATE OR ALTER`:

```sql
CREATE OR ALTER VIEW sales_by_client AS
SELECT c.client_id, SUM(invoice_total) AS total
FROM clients c
JOIN invoices i
ON c.client_id = i.client_id
GROUP BY c.client_id;
```

### Updatable views

- If we have a view that we haven't used `aggregate functions`, `UNION`, `GROUP BY`, or `DISTINCT` in it, that view is `updatable` meaning that we can change the original table (`INSERT`, `UPDATE`, or `DELETE`) with that view.
- Of course for `INSERT`, the view should include all `non-nullable` columns.
- If we use `WITH CHECK OPTION;` clause at the end of `CREATE VIEW` statement, it won't allow us to modify a record in a way that it doesn't show up in the view after modification.

### Materialized views

- For views, every time we access them, a new query will be run but with materialized view, we can run a expensive query and hold on to the results and refer to it later. It is best for queries that their results don't change too often.

```sql
CREATE MATERIALIZED VIEW weekly_likes AS (
    SELECT
        date_trunc('week', COALESCE(posts.crated_at, comments.created_at)) AS week
        COUNT(posts.id) AS num_likes_for_posts, -- null posts will not be counted
        COUNT(comments.id) AS num_likes_for_comments
    FROM likes
    LEFT JOIN posts ON posts.id = likes.posts_id
    LEFT JOIN comments ON comments.id = likes.comment_id
    GROUP BY week
    ORDER BY week
) WITH DATA;

SELECT * FROM weekly_likes; --will be instantaneous

REFRESH MATERIALIZED VIEW weekly_likes; --to re-run the query and update its cache.
```

## Stored procedures

- As a general rule, if we are not using an `ORM`s, we don't write queries directly in the app and we use `stored procedures`.
- A stored procedure is a database object which contains the SQL code and we call it in the app.
- Stored procedures return some records or perform a task on a/some table(s).
- It is cleaner, faster and we can restrict the access to the db and just expose some stored procedures for data manipulation with authorization settings.
- In `SQL Server`, you can only write one procedure per batch. So the last `SELECT` clause is what is returned from the stored procedure.

```sql
CREATE PROCEDURE getCustomers AS
SELECT * FROM customers;
```

- To run a stored procedure:

```sql
EXEC getCustomers;
```

- To delete a stored procedure:

```sql
DROP PROCEDURE IF EXISTS getCustomers;
```

### Stored procedures with parameter

```sql
CREATE PROCEDURE getCustomersByState @state char(2) AS
SELECT *
FROM customers
WHERE state = @state;
```

- For executing:

```sql
EXEC getCustomersByState @state = 'ca';
```

- If it has multiple parameters, we use `,` in both the procedure and its calling.

### Using conditionals inside a stored procedure

- `IF` example:

```sql
CREATE PROCEDURE getCustomersByState @state char(2) AS
IF @state IS NULL
    SET @state = 'CA';
SELECT *
FROM customers
WHERE state = @state;
```

- If we wanted to have multiple statements in each path, we would wrap them inside a `BEGIN...END` block.
- When calling this, the customers from `CA` will be returned:

```sql
EXEC getCustomersByState @state = NULL;
```

- `IF...ELSE IF...` example:

```sql
CREATE PROCEDURE getCustomersByState @state char(2) AS
IF @state = 'CALIFORNIA'
    SET @state = 'CA'
ELSE IF @state IS NULL
    SET @state = 'FL';
SELECT *
FROM customers
WHERE state = @state;
```

- `IF...ELSE...` example:

```sql
CREATE PROCEDURE getCustomersByState @state char(2) AS
IF @state IS NULL
    SELECT *
    FROM customers;
ELSE
    SELECT *
    FROM customers
    WHERE state = @state;
```

- A clever way to re-write the code above is:

```sql
CREATE PROCEDURE getCustomersByState @state char(2) AS
SELECT *
FROM customers c
WHERE c.state = ISNULL(@state, c.state);
```

### THROW exception

```sql
CREATE PROCEDURE getCustomersByState @state varchar(50) AS
IF @state = 'Gibberish' THROW 22003, 'State not found', 1 --22003 is code for out of range, error code can be between 0 and 255
SELECT *
FROM customers c
WHERE c.state = ISNULL(@state, c.state);
```

### Declaring a variable

- To declare a variable:

```sql
DECLARE @variable_name INT;
set @variable_name = 0;
```

- An important example:

```sql
CREATE PROCEDURE get_risk AS
DECLARE @risk_factor DECIMAL(9,2);
DECLARE @invoice_total DECIMAL(9,2);
DECLARE @invoices_count INT;
SET @invoice_total = (
    SELECT SUM(invoice_total)
    FROM invoices
    );
SET @invoices_count = (
    SELECT COUNT(*)
    FROM invoices
    );
SET @risk_factor = @invoice_total / @invoices_count * 5;
SELECT @risk_factor;
```

- We can re-write the code above with the technique below:

```sql
CREATE PROCEDURE get_risk AS
DECLARE @risk_factor DECIMAL(9,2);
DECLARE @invoice_total DECIMAL(9,2);
DECLARE @invoices_count INT;
SELECT @invoices_count = COUNT(*), @invoice_total = SUM(invoice_total)
FROM invoices;
SET @risk_factor = @invoice_total / @invoices_count * 5;
SELECT @risk_factor;
```

## Functions

- They are similar to procedures but they return only `one` value.

```sql
CREATE FUNCTION sqr(@n INT)
RETURNS INT
AS
BEGIN
    RETURN @N * @N;
END;
```

- To use a function:

```sql
SELECT dbo.sqr(5);
```

- If we wanted to use a function in the example from `stored procedures`:

```sql
CREATE FUNCTION get_risk(@client_id INT)
RETURNS INT
AS
BEGIN
	DECLARE @risk_factor DECIMAL(9,2);
	DECLARE @invoice_total DECIMAL(9,2);
	DECLARE @invoices_count INT;

	SELECT @invoices_count = COUNT(*), @invoice_total = SUM(invoice_total)
	FROM invoices i
	WHERE i.client_id = @client_id;

	SET @risk_factor = @invoice_total / @invoices_count * 5;
    RETURN @risk_factor;
END;
```

### Delete a function

```sql
DROP FUNCTION IF EXISTS get_risk;
```

## Triggers

- When something has happened (is going to happen) on a table, some `SQL` statements needs to be run after (before) it, which can affect that table or other tables:

```sql
CREATE TRIGGER payments_after_insert -- Convention: tableName_before/after_task
ON payments
AFTER INSERT
AS
DECLARE @amount DECIMAL(9,2);
DECLARE @id INT;

SELECT @amount = amount, @id = invoice_id
FROM INSERTED; -- We have access to virtual record of `INSERTED` and `DELETED`

UPDATE invoices
SET payment_total = payment_total + @amount
WHERE invoice_id = @id;
```

- For a trigger that inserts something (the example above is for update), we can use this pattern:

```sql
CREATE TRIGGER payments_after_insert
ON payments
AFTER INSERT
AS
INSERT INTO payments_audit
SELECT i.client_id, i.date, i.amount, 'Insert', GETDATE()
FROM INSERTED i;
```

- Triggers used to see who did what and when.

## Working with JSONs

- Suppose that we store a `JSON` object as `NVARCHAR(MAX)` in a field called `prop` : `{"info":{"manufacturer":[{"name":"LG"},{"name":"sony"}]}}`.

```sql
SELECT JSON_VALUE(prop, '$.info.manufacturer[0].name') AS first_manufacturer, --returns a scalar value //LG
JSON_QUERY(prop, '$.info.manufacturer') AS manufacturers --returns an array or an object //[{"name":"LG"},{"name":"sony"}]
FROM products
WHERE ISJSON(prop) > 0; --Checks if a string a of type JSON or not
```

- To update a `JSON` field:

```sql
UPDATE products
SET  prop = JSON_MODIFY(prop, '$.info.manufacturer[0].name','Samsung');
```

- With the function above, if the path does not exist it will create new fields in the `JSON` object:

```sql
UPDATE products
SET  prop = JSON_MODIFY(prop, '$.info.manufacturer[0].abbr','ss'); -- [{"name":"Samsung", "abbr":"ss"},{"name":"sony"}]
```

- If we set the path to `NULL`, it will remove it from `JSON` object:

```sql
UPDATE products
SET  prop = JSON_MODIFY(prop, '$.info.manufacturer[0].abbr',NULL); -- [{"name":"Samsung"},{"name":"sony"}]
```

## Transactions

- A set of related statements that needs to be executed together (a `Unit of Work`). Like withdrawing from one customer's balance and adding to another person's.

### ACID

- ACID (Atomicity, Consistency, Isolation, Durability) refers to a standard set of properties that guarantee database transactions are processed reliably.

#### Atomicity

- All changes must be committed or rolled back.

#### Consistency

- The database should be in a valid state (regarding constraints, cascades, and triggers) at all time.

#### Isolation

- Only one transaction can modify a record at a given time.

#### Durability

- After commit, the effects of the transaction must remain in the system.
- Nothing will happen to the tables:

```sql
BEGIN TRANSACTION;
	INSERT INTO customers (name)
	VALUES ('John');
	INSERT INTO products (prop)
	VALUES ('{"info":"null"}');
ROLLBACK; --dump all pending changes and delete this separate workspace
```

- Database will be updated:

```sql
BEGIN TRANSACTION;
	INSERT INTO customers (name)
	VALUES ('John');
	INSERT INTO products (prop)
	VALUES ('{"info":"null"}');
COMMIT; --merge changes into main data pool so other connections can see it.
```

### Concurrency problems

- When two transactions want to read or modify the same record at the same, some problems might rise.
- These problem will be prevented by using `isolation` strategies.

#### Lost update

- Transaction 1 reads the value.
- Transaction 2 reads the value.
- Transaction 1 modifies the value.
- Transaction 2 modifies the value.
- Transaction 1 commits.
- Transaction 2 commits.
- _Problem_: Only the changes made by transaction 2 will persist.
- _Solution_: The default lock of DBMSs will prevent this issue.

#### Dirty reads

- Transaction 1 reads the value.
- Transaction 1 modifies the value.
- Transaction 2 reads the modified value.
- Transaction 1 rolls back.
- _Problem_: Transaction 2 has a dirty read.
- _Solution_: We have to set the `isolation level` to `read committed` to only read the committed values.

#### Non-repeatable reads

- Transaction 1 reads the value.
- Transaction 1 modifies the value.
- Transaction 2 reads the value.
- Transaction 1 commits.
- Transaction 2 reads the value again.
- _Problem_: Transaction 2 has made non-repeatable reads.
- _Solution_: We have to set the `isolation level` to `repeatable read` to only read the committed values.

#### Phantom reads

- Transaction 1 has a query in it that returns some records
- Transaction 2 adds or removes some records which would have affected the result of the above query.
- _Problem_: Transaction 2 doesn't include some records or has extra records.
- _Solution_: We have to set the `isolation level` to `serializable`. In this way, transaction 2 is constantly watching all the implications of other transactions.

### Isolation levels

- From left to right, more performance problems and fewer concurrency ones:

> read uncommitted -> read committed (default in SQL Server) -> repeatable read (default in MySQL) -> serializable

- You can see the isolation level by:

```sql
DBCC USEROPTIONS;
```

- And you can change it with:

```sql
SET TRANSACTION ISOLATION LEVEL READ UNCOMMITTED
SET TRANSACTION ISOLATION LEVEL READ COMMITTED
SET TRANSACTION ISOLATION LEVEL REPEATABLE READ
SET TRANSACTION ISOLATION LEVEL SERIALIZABLE
```

### Deadlock

- Because of the default lock that `DBMS`s put on records that have been modified (but not yet committed) by a transaction, we might have this situation:

```sql
BEGIN TRANSACTION                          --BEGIN TRANSACTION
UPDATE orders SET status = 1;              --UPDATE customers SET state = 'ca';
UPDATE customers SET state = 'fl';--(x)    --UPDATE orders SET status = 2; (x)
COMMIT;                                    --COMMIT;
```

- One of the transactions will time-out and the other one will be committed.
- _Solution 1_: The orders of modifying tables should be identical in different transactions.
- _Solution 2_: Lengthy transactions should be performed during mid-nights.

## Schema migration

- How to change database schema in PROD!

  1. Changes to the DB structure and changes to clients (like an API server) need to be made at precisely the same time. But because database deployment is so fast but API is taking some time, there will be a window of time with errors! -> so we should announce scheduled down-time?! No.
  2. When working with others, we need a really easy way to tie the structure of our database to our code.

- Whenever we want to modify our database, we should write a `schema migration file` which is a code to describe changes to make to the database. In such a file, there should be a section for `Up` (code to update database) and a section for `Down` (code to undo the 'up' command).
- Migration files will solve the first problem, by writing a script that when the deployment is ready, apply the migration files and then start receiving traffic.
- There are many libraries to create/run schema migration files. For node.js, `node-pg-migrate`, `sequelize`, `typeorm`, and `db-migrate`.
- It is better when using these libraries, to write all migrations manually using plain SQL and not rely on automatically-generated migrations.

```
npm init -y
npm i node-pg-migrate pg
```

add this script to `package.json` file:

```json
"migrate" : "node-pg-migrate"
```

then (whatever after create is the migration name):

```
npm run migrate create table comments
```

A migration file with the current timestamp in the `migrations` folder will be created. Open that file and edit it like:

```js
exports.shorthands = undefined;

exports.up = (pgm) => {
  pgm.sql(`
        CREATE TABLE comments (
            id SERIAL PRIMARY KEY,
            created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
            updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
            contents VARCHAR(240) NOT NULL
        );
    `);
};

exports.down = (pgm) => {
  pgm.sql(`
        DROP TABLE comments;
    `);
};
```

Then, you have to set `DATABASE_URL` env variable: `postgres://<username>:<password>@localhost:5432/<db_name>` and then run `npm run migrate up`. For example in windows Git Bash:

```
DATABASE_URL=postgres://<username>:<password>@localhost:5432/<db_name> npm run migrate up
```

or in CMD:

```
set DATABASE_URL=postgres://<username>:<password>@localhost:5432/<db_name>&&npm run migrate up
```

- The `up` command will apply all the migrations up to this point.
- with:

```
set DATABASE_URL=postgres://<username>:<password>@localhost:5432/<db_name>&&npm run migrate down
```

we will undo the `most recent` migration.

## Data migration

- We should not have a migration that has schema and data migrations at the same migration because data migration is usually time-consuming and the data has been added during that will be lost. Solution:

  - Add new column by creating a new migration
  - Deploy new API which writes values on both columns
  - Populate new column based on old column (it doesn't need to be a schema migration file). Instead, we create a folder `data` inside `migrations` to indicated that it is a data migration. Then, inside that, we create a JS file named `01-lng-lat-to-loc.js` and run it by `node 01-lng-lat-to-loc.js` (we could write the following sql in pg admin but in that case we wouldn't have a record so this is better):

  ```js
  const pg = require("pg");

  const pool = new pg.pool({
    host: "localhost",
    port: 5432,
    database: "socialnetwork",
    user: "postgres",
    password: "password",
  });

  pool
    .query(
      `
    UPDATE posts
    SET loc = POINT(lng, lat)
    WHERE loc IS NULL;    
  `
    )
    .then(() => {
      console.log("Update complete");
      pool.end();
    })
    .catch((err) => {
      console.error(err.message);
    });
  ```

  - Deploy new API which writes only on new column
  - Drop old column by creating a new migration

# Node Repository Pattern

- Read the following project to learn about how to connect to a database from a node API, repository pattern, SQL injection, test database, parallel testing using one database.

[Node Repository Pattern](https://github.com/JohnDoe/node-repository-pattern)

# Security

- In `SQL Server`, a `LOGIN` is to access the server and `USER` is to access the database.

```sql
CREATE LOGIN john WITH PASSWORD = '1234';
```

- To list all the logins (alternatively, you can see all the logins by browsing `Security/Logins` on the left panel):

```sql
SELECT *
FROM sys.server_principals;
```

- To delete a login:

```sql
DROP LOGIN john;
```

- To change a password:

```sql
ALTER LOGIN john WITH PASSWORD = 'pass';
```

- After creating a `LOGIN`, we create a `USER` which is a client (e.g a web app).

```sql
CREATE USER ben FOR LOGIN john
GRANT DELETE ON products TO ben;
```

- To revoke privilege:

```sql
REVOKE DELETE ON products To ben;
```
