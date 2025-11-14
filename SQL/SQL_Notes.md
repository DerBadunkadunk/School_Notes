# SQL Databases & Commands Notes


## Table of Contents
- [SQL Databases & Commands Notes](#sql-databases--commands-notes)
  - [What is SQL?](#what-is-sql)
  - [Common SQL Data Types](#common-sql-data-types)
  - [Database & Table Management](#database--table-management)
  - [Basic CRUD Commands](#basic-crud-commands)
  - [Filtering & Sorting](#filtering--sorting)
  - [Useful Clauses](#useful-clauses)
  - [Joins](#joins)
  - [Aggregation & Grouping](#aggregation--grouping)
  - [Indexes & Keys](#indexes--keys)
  - [User & Permissions](#user--permissions)
  - [Handy Tips](#handy-tips)
- [Entity-Relationship Diagram](#entity-relationship-diagram-erd-notes)
  - [What is an ERD?](#what-is-an-erd)
  - [Core Components](#core-components)
    - [Entities](#entities)
    - [Attributes](#attributes)
    - [Keys](#keys)
    - [Relationships](#relationships)
  - [Cardinality & Participation](#cardinality--participation)
  - [Example ERD Scenario](#example-erd-scenario)
  - [Steps to Create an ERD](#steps-to-create-an-erd)
  - [ERD Notations](#erd-notations)
  - [Quick Reference](#quick-reference)
  - [Column Attributes](#column-attributes)
---

## What is SQL?

- **SQL (Structured Query Language)** is used to manage and manipulate relational databases.
- Databases are collections of **tables** that store data in rows and columns.
- Each table has:
  - **Columns (fields/attributes)** → define the type of data stored.
  - **Rows (records)** → individual data entries.

---

## Common SQL Data Types
- `INT` → whole numbers
- `DECIMAL(p,s)` → fixed-point numbers
- `VARCHAR(n)` → variable-length text
- `TEXT` → long text
- `DATE`, `TIME`, `DATETIME` → temporal values
- `BOOLEAN` → true/false

---

## Database & Table Management
```sql
-- Create a new database
CREATE DATABASE db_name;

-- Use a database
USE db_name;

-- Delete a database
DROP DATABASE db_name;

-- Create a table
CREATE TABLE table_name (
    id INT PRIMARY KEY AUTO_INCREMENT,
    column1 VARCHAR(100) NOT NULL,
    column2 INT,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP
);

-- View tables in a database
SHOW TABLES;

-- View table structure
DESCRIBE table_name;

-- Delete a table
DROP TABLE table_name;
```
---

## Basic CRUD Commands

CRUD = Create, Read, Update, Delete

```sql
-- Insert data
INSERT INTO table_name (column1, column2) 
VALUES ('value1', 123);

-- Select (read) data
SELECT * FROM table_name;
SELECT column1, column2 FROM table_name WHERE column2 > 100;

-- Update data
UPDATE table_name
SET column1 = 'new_value'
WHERE id = 1;

-- Delete data
DELETE FROM table_name
WHERE id = 1;
```
---

## Filtering & Sorting

```sql
-- Filtering with WHERE
SELECT * FROM table_name WHERE column1 = 'value';

-- Comparison operators: =, !=, <, >, <=, >=
-- Logical operators: AND, OR, NOT

-- Sorting results
SELECT * FROM table_name ORDER BY column1 ASC;
SELECT * FROM table_name ORDER BY column1 DESC;

-- Limit number of rows returned
SELECT * FROM table_name LIMIT 10;
```
---

## Useful Clauses

```sql
-- DISTINCT (unique values only)
SELECT DISTINCT column1 FROM table_name;

-- BETWEEN
SELECT * FROM table_name WHERE column2 BETWEEN 10 AND 20;

-- IN
SELECT * FROM table_name WHERE column1 IN ('a', 'b', 'c');

-- LIKE (pattern matching, % = wildcard)
SELECT * FROM table_name WHERE column1 LIKE 'abc%';
```
---

## Joins

```sql
-- INNER JOIN (only matching rows)
SELECT a.id, b.info
FROM table_a a
INNER JOIN table_b b ON a.id = b.a_id;

-- LEFT JOIN (all rows from left, matched from right)
SELECT a.id, b.info
FROM table_a a
LEFT JOIN table_b b ON a.id = b.a_id;

-- RIGHT JOIN (all rows from right, matched from left)
SELECT a.id, b.info
FROM table_a a
RIGHT JOIN table_b b ON a.id = b.a_id;

-- FULL JOIN (all rows from both tables, if supported)
```
---

## Aggregation & Grouping
```sql
-- Aggregate functions: COUNT(), SUM(), AVG(), MIN(), MAX()

SELECT COUNT(*) FROM table_name;
SELECT AVG(column2) FROM table_name;

-- GROUP BY
SELECT column1, COUNT(*) 
FROM table_name
GROUP BY column1;

-- HAVING (filter groups)
SELECT column1, COUNT(*) 
FROM table_name
GROUP BY column1
HAVING COUNT(*) > 2;
```
---

## Indexes & Keys

- Primary Key → unique identifier for rows.

- Foreign Key → reference to another table’s primary key.

- Index → speeds up searching.

```sql
-- Add index
CREATE INDEX idx_col1 ON table_name (column1);

-- Drop index
DROP INDEX idx_col1 ON table_name;
```
---

## User & Permissions

```sql
-- Create user
CREATE USER 'username'@'localhost' IDENTIFIED BY 'password';

-- Grant privileges
GRANT ALL PRIVILEGES ON db_name.* TO 'username'@'localhost';

-- Revoke privileges
REVOKE ALL PRIVILEGES ON db_name.* FROM 'username'@'localhost';

-- Show users/privileges
SELECT User, Host FROM mysql.user;
SHOW GRANTS FOR 'username'@'localhost';
```
---

## Handy Tips

- Use ; to end each SQL statement.

- Use -- for single-line comments, /* ... */ for multi-line comments.

- Always back up your database before running DROP or mass DELETE.

- Start queries with SELECT * FROM ... to explore before writing complex filters.

- Test joins with small datasets first.
---



# Entity-Relationship Diagram (ERD) Notes

## What is an ERD?
- **Entity-Relationship Diagram (ERD)** is a visual representation of how entities (tables) in a database relate to each other.
- Used during **database design** to model structure before implementation.
- Helps clarify:
  - Entities (objects)
  - Attributes (fields)
  - Relationships (how entities connect)

---

## Core Components

### Entities
- Represent **tables** in a database.
- Shown as **rectangles** in ERDs.
- Two types:
  - **Strong Entity** → exists independently (e.g., `Customer`, `Product`)
  - **Weak Entity** → depends on another entity (e.g., `OrderItem` depends on `Order`)

### Attributes
- Represent **columns (fields)** inside tables.
- Shown as **ellipses (ovals)**.
- Types:
  - **Simple Attribute** → cannot be divided further (e.g., `FirstName`)
  - **Composite Attribute** → can be broken down (e.g., `FullName` → `FirstName`, `LastName`)
  - **Derived Attribute** → calculated from other attributes (e.g., `Age` from `DOB`)
  - **Multivalued Attribute** → can have multiple values (e.g., `PhoneNumbers`)

### Keys
- **Primary Key (PK)** → uniquely identifies each record.
- **Foreign Key (FK)** → links to the PK of another entity.

### Relationships
- Represent associations between entities.
- Shown as **diamonds** (or just lines in modern notation).
- Types:
  - **One-to-One (1:1)** → one entity relates to exactly one of another.
  - **One-to-Many (1:N)** → one entity relates to many of another.
  - **Many-to-Many (M:N)** → many entities relate to many others (usually broken into a junction table).

---

## Cardinality & Participation
- **Cardinality** → number of instances involved in a relationship.
  - 1:1 → Each entity instance relates to only one other.
  - 1:N → A parent entity can have multiple children, but each child has one parent.
  - M:N → Multiple records from each side can relate to each other.
- **Participation**:
  - **Total participation** → all entities must be related (double line).
  - **Partial participation** → some entities may not be related (single line).

---

## Example ERD Scenario
Entities:
- `Customer` (CustomerID, Name, Email)
- `Order` (OrderID, Date, CustomerID)
- `Product` (ProductID, Name, Price)
- `OrderItem` (OrderID, ProductID, Quantity)

Relationships:
- A **Customer places Orders** → 1:N
- An **Order contains Products** → M:N resolved by `OrderItem`
- A **Product appears in many Orders** → M:N

---

## Steps to Create an ERD
1. Identify entities (nouns in requirements).
2. Define attributes (properties of entities).
3. Mark primary keys and foreign keys.
4. Determine relationships between entities.
5. Assign cardinality (1:1, 1:N, M:N).
6. Normalize (resolve M:N into junction tables).
7. Draw ERD using notation (Chen, Crow’s Foot, UML).

---

## ERD Notations
- **Chen Notation** → entities (rectangles), attributes (ovals), relationships (diamonds).
- **Crow’s Foot Notation** → popular in database design, shows cardinality with "feet" symbols.
- **UML Class Diagram** → often used in software engineering, represents tables as classes.

---

## Quick Reference
- **Entity** → Table
- **Attribute** → Column
- **Primary Key** → Unique identifier
- **Foreign Key** → Reference to another table
- **Relationship** → Link between tables
- **1:1, 1:N, M:N** → Types of cardinality
---

## Column Attributes
- **DataType** - The type of data that can be stored in the column.
- **Primary Key** - A unique identifier for each row in the table.
- **Not Null** - The column cannot be empty.
- **Unique** - The column cannot contain duplicate values.
- **Binary** - The column contains binary data.
- **Unsigned** - The column can only contain positive values.
- **Zerofill** - If the column is not long enough to display the value, it will be padded with zeros.
- **Auto Increment** - The column will automatically increment by one for each new row.
- **Generated** - The column will be computed using an expression rather than stored in the table.
- **Default** - The column will be set to the specified value if no value is provided.
---

# Creating Tables in SQL

## Overview
- Tables store data in **rows (records)** and **columns (fields/attributes)**.
- Each column has a **data type** that defines what kind of values it can hold.
- A table can have **constraints** to enforce rules (e.g., uniqueness, not null).

---

## Basic Syntax
```sql
CREATE TABLE table_name (
    column1 datatype constraints,
    column2 datatype constraints,
    ...
);
```

### Example
```sql
CREATE TABLE Customers (
    CustomerID INT PRIMARY KEY AUTO_INCREMENT,
    FirstName VARCHAR(50) NOT NULL,
    LastName VARCHAR(50) NOT NULL,
    Email VARCHAR(100) UNIQUE,
    CreatedAt DATETIME DEFAULT CURRENT_TIMESTAMP
);
```

# SQL Schema Notes

## What is a Schema?
- A **schema** is a logical container for database objects such as:
  - Tables
  - Views
  - Indexes
  - Stored procedures
  - Functions
- Helps organize and group objects within a database.
- Think of it as a **namespace** or **folder** inside a database.

---

## Why Use Schemas?
- **Organization** → separates objects by function, project, or module.
- **Security** → permissions can be applied at schema level.
- **Collaboration** → multiple teams can work in the same database but in different schemas.
- **Avoiding Name Conflicts** → same table name can exist in different schemas (e.g., `sales.Customers` vs `hr.Customers`).

---

## Syntax Examples

### Create Schema
```sql
-- Create a new schema
CREATE SCHEMA sales;

-- Create a schema with authorization (SQL Server / PostgreSQL)
CREATE SCHEMA hr AUTHORIZATION hr_admin;
```

## Create Table in Schema
```sql
CREATE TABLE sales.Orders (
    OrderID INT PRIMARY KEY,
    CustomerID INT,
    OrderDate DATE,
    Total DECIMAL(10,2)
);
```

## Referencing a Schema
```sql
-- Explicitly reference schema
SELECT * FROM sales.Orders;

-- If schema is not specified, defaults to 'dbo' (SQL Server) or 'public' (PostgreSQL/MySQL).
```

## Alter or Drop Schema
```sql
-- Rename schema (PostgreSQL)
ALTER SCHEMA hr RENAME TO human_resources;

-- Drop schema (must be empty)
DROP SCHEMA sales;
DROP SCHEMA hr CASCADE; -- PostgreSQL: also removes contained objects
```
---



# SQL Wildcards & LIKE Operator

## What is LIKE?
- The **`LIKE` operator** is used in a `WHERE` clause to search for a specified pattern in a column.
- Works with **wildcards** to match flexible patterns.
- Commonly used for string/text matching.

---

## Basic Syntax
```sql
SELECT column1, column2
FROM table_name
WHERE column_name LIKE 'pattern';
```

## Wildcards in SQL

### % (PERCENT)
- Represents 0, 1, or multiple characters.
```sql
-- Find all names starting with 'A'
SELECT * FROM Customers WHERE Name LIKE 'A%';

-- Find all names ending with 'son'
SELECT * FROM Customers WHERE Name LIKE '%son';

-- Find all names containing 'an'
SELECT * FROM Customers WHERE Name LIKE '%an%';
```

### _ (Underscore)
- Represents a single character
```sql
-- Find all names with 4 letters, starting with 'J'
SELECT * FROM Customers WHERE Name LIKE 'J___';

-- Find names where the second letter is 'a'
SELECT * FROM Customers WHERE Name LIKE '_a%';
```

### [] (Bracket Expression) - SQL Server / Access
- Matches any single character within the brackets
```sql
-- Find names starting with 'A' or 'B'
SELECT * FROM Customers WHERE Name LIKE '[AB]%';

-- Find names starting with letters A through F
SELECT * FROM Customers WHERE Name LIKE '[A-F]%';
```

### [^] (Not in a Bracket Expression)
- Matches any single character not in the brackets
```sql
-- Find names NOT starting with A, B, or C
SELECT * FROM Customers WHERE Name LIKE '[^ABC]%';
```

### Escape Characters
- Used to search for wildcards literally (e.g., find % or _ in text).
```sql
-- Example in SQL Server / PostgreSQL
SELECT * FROM Comments
WHERE Text LIKE '%\%%' ESCAPE '\';
```

### Examples
```sql
-- Find emails from Gmail
SELECT * FROM Users WHERE Email LIKE '%@gmail.com';

-- Find phone numbers starting with 555
SELECT * FROM Contacts WHERE Phone LIKE '555%';

-- Find products with exactly 5-character codes starting with 'AB'
SELECT * FROM Products WHERE Code LIKE 'AB___';
```

### Quick Reference
**%** → zero or more characters

- **_** → exactly one character

- **[abc]** → matches one character from the set

- **[a-z]** → matches one character from the range

- **[^abc]** → matches one character NOT in the set

- **ESCAPE** → treat wildcard as literal
---



# SQL GROUP BY and HAVING Notes

## GROUP BY
- The **`GROUP BY`** clause groups rows that have the same values in specified columns into summary rows.
- Often used with **aggregate functions** like:
  - `COUNT()` → number of rows
  - `SUM()` → total
  - `AVG()` → average
  - `MIN()` / `MAX()` → smallest / largest value

### Syntax
```sql
SELECT column1, aggregate_function(column2)
FROM table_name
GROUP BY column1;
```

### Example
```sql
-- Total balance per representative
SELECT Rep_Num, SUM(Balance) AS "Total Balance"
FROM Customers
GROUP BY Rep_Num;
```

## HAVING
- The **HAVING** clause filters results after grouping (unlike **WHERE**, which filters before grouping).

- Useful when you want to filter groups based on aggregate conditions.

### Syntax
```sql
SELECT column1, aggregate_function(column2)
FROM table_name
GROUP BY column1
HAVING condition;
```

### Example
```sql
-- Show reps with total balances greater than 10,000
SELECT Rep_Num, SUM(Balance) AS "Total Balance"
FROM Customers
GROUP BY Rep_Num
HAVING SUM(Balance) > 10000;
```

## Key Differences: WHERE vs HAVING
- **WHERE** → filters rows before grouping.

- **HAVING** → filters groups after aggregation.

### Example
```sql
-- Filter before grouping (only customers in ZIPs starting with '5')
SELECT Rep_Num, SUM(Balance) AS "Total Balance"
FROM Customers
WHERE ZIP LIKE '5%'
GROUP BY Rep_Num;

-- Filter after grouping (only reps whose customers total over 10,000)
SELECT Rep_Num, SUM(Balance) AS "Total Balance"
FROM Customers
GROUP BY Rep_Num
HAVING SUM(Balance) > 10000;
```

### Quick Reference
- **GROUP BY** → groups rows by column(s).

- **HAVING** → applies conditions to grouped data.

- **WHERE** → applies conditions to individual rows.