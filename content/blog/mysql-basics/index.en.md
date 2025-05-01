---
title: MySQL Basics
weight: -45
draft: false
description: The deployment tutorial and basic syntax of MySQL
slug: mysql-basics
tags:
  - MySQL
series:
  - Technical Miscellany
series_order: 4
date: 2025-01-15
lastmod: 2025-02-15
authors:
  - Morethan
---
{{< lead >}}
MySQL Installation and Deployment Process + Quick Syntax CookBook + Study Notes
{{< /lead >}}

### Information Source

- [SQL Tutorial - Liao Xuefeng's Official Website](https://liaoxuefeng.com/books/sql/introduction/index.html)  
    This is an extremely user-friendly MySQL tutorial website with an embedded web-based database, making it easy for beginners to get a hands-on understanding of MySQL operations. It also provides concise yet essential explanations of the background of SQL.

### MySQL Installation

The simplest way to install MySQL is through Docker Desktop. It takes only two steps to complete:

Run the following command in your terminal:

```sh
docker pull mysql
```

#### Run MySQL With Shell

Then initialize and run the SQL container:

```sh
docker run -d --name mysql -p 3306:3306 -e MYSQL_ROOT_PASSWORD=password -v /Users/liaoxuefeng/mysql-data:/var/lib/mysql mysql
```

**Parameter Explanation**:

|Parameter|Description|
|---|---|
|`-d`|Run the container in the background|
|`--name`|Assign a name to the container (if not specified, Docker will choose one automatically)|
|`-p 3306:3306`|Map the container's port 3306 to the local machine, allowing you to connect to MySQL through port 3306|
|`-e MYSQL_ROOT_PASSWORD=password`|Set the root password for MySQL (in this case, the password is `password`). If not set, Docker will generate a password, which you’ll need to check the logs to retrieve. **It’s recommended to set a password.**|
|`-v /path:/var/lib/mysql`|Mount a local directory to `/var/lib/mysql` in the container, which will store MySQL data. Replace `/path` with the actual directory path on your machine|
|`mysql`|Specifies the name of the Docker image you want to run|


{{< alert icon="pencil" cardColor="#1E3A8A" textColor="#E0E7FF" >}}
When using Docker to run MySQL, you can always delete the MySQL container and rerun it. If you delete the locally mounted directory, rerunning the container is equivalent to starting with a fresh MySQL instance.
{{< /alert >}}

#### Run MySQL With docker-compose

Create `docker-compose.yml`：

```yml
services:
  mysql:
    image: mysql
    ports:
      - "3306:3306"
    environment:
      MYSQL_ROOT_PASSWORD: password
    volumes:
      - ../data:/var/lib/mysql
    restart: unless-stopped
```

Run `docker-compose build up -d` to start service.

### MySQL Basic Syntax
#### Querying

**Comment Syntax**:

```sql
-- This is a comment
```

**Basic Query Syntax**:

```sql
SELECT * FROM <table_name>;
```

**Conditional Query**:

```sql
SELECT * FROM <table_name> WHERE <condition>;
```

In the condition expression, you can use various logical operators such as: `AND`, `NOT`, `OR`.

**Projection Query**:

```sql
SELECT <column1>, <column2>, <column3> FROM <table_name>;
SELECT <column1> alias1, <column2> alias2, <column3> alias3 FROM <table_name>;
```

**Sorting**:

```sql
-- Sort by score in ascending order:
SELECT id, name, gender, score FROM students ORDER BY score;

-- Sort by score in descending order:
SELECT id, name, gender, score FROM students ORDER BY score DESC;

-- Sort by score, gender:
SELECT id, name, gender, score FROM students ORDER BY score DESC, gender;

-- Sorting with a WHERE condition:
SELECT id, name, gender, score
FROM students
WHERE class_id = 1
ORDER BY score DESC;
```

**Pagination Query**:

```sql
-- Query the first page:
SELECT id, name, gender, score
FROM students
ORDER BY score DESC
LIMIT 3 OFFSET 0; -- Start from the 0th record, fetch up to 3 records, which may be less than 3.
```

**Aggregation Query**, using MySQL’s aggregation functions:

```sql
-- Count the total number of records:
SELECT COUNT(*) FROM students;

-- Set the alias for the count result:
SELECT COUNT(*) num FROM students;

-- Conditional aggregation:
SELECT COUNT(*) boys FROM students WHERE gender = 'M';
```



{{< alert icon="pencil" cardColor="#1E3A8A" textColor="#E0E7FF" >}}
Even though `COUNT(*)` returns a scalar value, the result is still a two-dimensional table, but with only one row and one column.
{{< /alert >}}


Other commonly used aggregation functions: `MAX()`, `MIN()`, `AVG()`, `SUM()`, etc., similar to `COUNT()`.

**Grouped Aggregation Query**:

```sql
-- Group by class_id and count records, similar to a for loop:
SELECT COUNT(*) num FROM students GROUP BY class_id;

-- Include class_id in the result:
SELECT class_id, COUNT(*) num FROM students GROUP BY class_id;

-- Group by multiple fields, e.g., class_id and gender:
SELECT class_id, gender, COUNT(*) num FROM students GROUP BY class_id, gender;
```

**Multi-table Query (Cartesian Product)**:

```sql
-- Querying from students and classes:
SELECT * FROM students, classes;

-- Set aliases and distinguish columns with table names:
SELECT
    students.id sid,
    students.name,
    students.gender,
    students.score,
    classes.id cid,
    classes.name cname
FROM students, classes;

-- Using aliases for both tables to make it cleaner:
SELECT
    s.id sid,
    s.name,
    s.gender,
    s.score,
    c.id cid,
    c.name cname
FROM students s, classes c;
```

The result of a multi-table query is still a two-dimensional table, but this table is organized using a **Cartesian product**, which is why it’s also known as a Cartesian Query.

**Join Queries**, unlike multi-table queries where the tables are combined using Cartesian products, join queries select one table as the main table and combine it with an auxiliary table based on a relationship:

- **Inner Join** (using `INNER`):

```sql
-- Select all students and their corresponding class names:
SELECT s.id, s.name, s.class_id, c.name class_name, s.gender, s.score
FROM students s
INNER JOIN classes c
ON s.class_id = c.id;
```

- **Outer Join**, with three variations: `RIGHT`, `LEFT`, `FULL`:

```sql
-- Using RIGHT OUTER JOIN:
SELECT s.id, s.name, s.class_id, c.name class_name, s.gender, s.score
FROM students s
RIGHT OUTER JOIN classes c
ON s.class_id = c.id;
```

Understanding method: You can think of it as sets, where `tableA` is the main table (also known as the left table) and `tableB` is the related table (the right table).

```sql
-- Here tableA is the main table, also known as the left table, and tableB is the related table.
SELECT ... FROM tableA ??? JOIN tableB ON tableA.column1 = tableB.column2;
```

![JoinQuery.png](img/JoinQuery.png)

#### Modifying Data

- **Inserting Data**:

```sql
-- Insert a new record:
INSERT INTO students (class_id, name, gender, score) VALUES (2, 'Daniu', 'M', 80);

-- Insert multiple records at once:
INSERT INTO students (class_id, name, gender, score) VALUES
  (1, 'Dabao', 'M', 87),
  (2, 'Erbao', 'M', 81),
  (3, 'Sanbao', 'M', 83);
```

- **Updating Data**:

```sql
-- Update the record with id=1:
UPDATE students SET name='Daniu', score=66 WHERE id=1;

-- Update records with score < 80:
UPDATE students SET score=score+10 WHERE score<80;

-- Update record with id=999, but no matching records will be found:
UPDATE students SET score=100 WHERE id=999;

-- Without a WHERE clause, the update will affect all records in the table:
UPDATE students SET score=60;
```

- **Deleting Data**:

```sql
-- Delete the record with id=1:
DELETE FROM students WHERE id=1;

-- Deleting without a WHERE clause will remove all records from the table:
DELETE FROM students;
```

Here's a fluent and natural English translation:

#### Creating Database and Tables

- **Creating the Database**:

```sql
CREATE DATABASE your_db_name -- name of the database
CHARACTER SET utf8mb4 -- character set of the database
COLLATE utf8mb4_unicode_ci; -- collation rule
```

- **Creating Tables**:

```sql
CREATE TABLE table_name (
    column1 datatype constraints,
    column2 datatype constraints,
    column3 datatype constraints,
    ...
    PRIMARY KEY (primary_key_column)
);

-- Example of a user table
CREATE TABLE users (
    id INT AUTO_INCREMENT,
    username VARCHAR(50) NOT NULL UNIQUE,
    email VARCHAR(100) NOT NULL UNIQUE,
    password VARCHAR(255) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    PRIMARY KEY (id)
);
```

- **Creating Triggers**：

```sql
CREATE TRIGGER trigger_name
BEFORE INSERT ON orders
FOR EACH ROW
BEGIN
    SET NEW.order_time = NOW();
END;
```
