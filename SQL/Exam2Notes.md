````md
# SQL Notes – Multi-Table Queries, Advanced Joins, Many-to-Many, and RESTful APIs

## 0. Relational Basics (Quick Context)

- **Table** – like a spreadsheet. Rows = records, columns = attributes.
- **Primary Key (PK)** – column(s) that uniquely identifies a row (e.g., `id`).
- **Foreign Key (FK)** – column that references a PK in another table.
- **Relationship types:**
  - **One-to-One (1:1)** – rare (e.g., `User` and `UserProfile`).
  - **One-to-Many (1:N)** – common (e.g., `Customer` → `Orders`).
  - **Many-to-Many (M:N)** – implemented via a **junction (link) table**.

---

## 1. Multi-Table Queries

### 1.1. Why Join Tables?

You normalize data into multiple tables (avoid duplication), then **join** them when you query:

Example schema:

```text
customers(id, name, email)
orders(id, customer_id, order_date, total_amount)
order_items(id, order_id, product_id, quantity, unit_price)
products(id, name, price)
````

Key relationships:

* `orders.customer_id` → `customers.id`
* `order_items.order_id` → `orders.id`
* `order_items.product_id` → `products.id`

### 1.2. Explicit vs. Implicit Joins

**Explicit JOIN (preferred):**

```sql
SELECT
  c.name AS customer_name,
  o.id   AS order_id,
  o.order_date
FROM customers AS c
JOIN orders AS o
  ON o.customer_id = c.id;
```

**Implicit JOIN (old style, avoid):**

```sql
SELECT
  c.name,
  o.id,
  o.order_date
FROM customers c, orders o
WHERE o.customer_id = c.id;
```

* Always use **explicit JOIN** with an `ON` clause. It’s clearer and safer.

### 1.3. Table Aliases

Aliases make queries shorter and easier to read:

```sql
SELECT
  c.name,
  p.name AS product_name,
  oi.quantity
FROM customers c
JOIN orders o      ON o.customer_id = c.id
JOIN order_items oi ON oi.order_id = o.id
JOIN products p    ON p.id = oi.product_id;
```

* Use short, meaningful aliases (`c`, `o`, `oi`, `p`).

### 1.4. Multi-Table Joins (3+ Tables)

You can chain joins as long as each join’s `ON` condition is valid:

```sql
SELECT
  c.name          AS customer_name,
  o.id            AS order_id,
  o.order_date,
  p.name          AS product_name,
  oi.quantity,
  oi.unit_price,
  (oi.quantity * oi.unit_price) AS line_total
FROM customers c
JOIN orders o       ON o.customer_id = c.id
JOIN order_items oi ON oi.order_id = o.id
JOIN products p     ON p.id = oi.product_id
ORDER BY o.order_date DESC, c.name;
```

---

## 2. Advanced Joins

### 2.1. INNER JOIN

Returns rows where there is a match in **both** tables.

```sql
SELECT
  c.name,
  o.id,
  o.order_date
FROM customers c
INNER JOIN orders o
  ON o.customer_id = c.id;
```

### 2.2. LEFT / RIGHT / FULL OUTER JOIN

* **LEFT JOIN** – all rows from left table + matching rows from right (or `NULL`).
* **RIGHT JOIN** – all rows from right table + matching rows from left (or `NULL`).
* **FULL OUTER JOIN** – all rows from both, matched where possible, `NULL` otherwise.

**LEFT JOIN example (customers with or without orders):**

```sql
SELECT
  c.id,
  c.name,
  o.id AS order_id
FROM customers c
LEFT JOIN orders o
  ON o.customer_id = c.id
ORDER BY c.id;
```

* Customers with `order_id = NULL` have **no orders**.

### 2.3. CROSS JOIN

Cartesian product: every row of A with every row of B (rare in real apps).

```sql
SELECT *
FROM colors
CROSS JOIN sizes;
```

### 2.4. SELF JOIN

Join a table to itself (e.g., employees & managers).

```sql
employees(id, name, manager_id)

SELECT
  e.name AS employee,
  m.name AS manager
FROM employees e
LEFT JOIN employees m
  ON e.manager_id = m.id;
```

### 2.5. Non-Equi Joins

Joins on conditions other than equality, e.g., date ranges or numeric ranges:

```sql
-- Example: find tax bracket for each income
SELECT
  p.name,
  p.income,
  b.rate
FROM people p
JOIN tax_brackets b
  ON p.income BETWEEN b.min_income AND b.max_income;
```

### 2.6. Semi-Joins (EXISTS) and Anti-Joins (NOT EXISTS)

**Semi-join** – rows where a match exists in another table:

```sql
-- Customers who have placed at least one order
SELECT DISTINCT c.*
FROM customers c
WHERE EXISTS (
  SELECT 1
  FROM orders o
  WHERE o.customer_id = c.id
);
```

**Anti-join** – rows where **no** match exists in another table:

```sql
-- Customers who have NEVER placed an order
SELECT c.*
FROM customers c
WHERE NOT EXISTS (
  SELECT 1
  FROM orders o
  WHERE o.customer_id = c.id
);
```

> Use `EXISTS` / `NOT EXISTS` for performance and clarity in complex filters.

---

## 3. Many-to-Many Relationships

### 3.1. Structure with Junction Table

Example: **students** and **courses**:

```text
students(id, name, email)
courses(id, title, credits)
student_courses(student_id, course_id, enrolled_at)
```

* `student_courses.student_id` → `students.id`
* `student_courses.course_id` → `courses.id`
* PK for the junction table is often `(student_id, course_id)` (composite key).

```sql
CREATE TABLE student_courses (
  student_id INT NOT NULL,
  course_id  INT NOT NULL,
  enrolled_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  PRIMARY KEY (student_id, course_id),
  FOREIGN KEY (student_id) REFERENCES students(id),
  FOREIGN KEY (course_id)  REFERENCES courses(id)
);
```

### 3.2. Query: All Courses for a Student

```sql
SELECT
  s.name  AS student_name,
  c.title AS course_title
FROM students s
JOIN student_courses sc ON sc.student_id = s.id
JOIN courses c         ON c.id = sc.course_id
WHERE s.id = 42;
```

### 3.3. Query: All Students in a Course

```sql
SELECT
  c.title AS course_title,
  s.name  AS student_name
FROM courses c
JOIN student_courses sc ON sc.course_id = c.id
JOIN students s         ON s.id = sc.student_id
WHERE c.id = 10;
```

### 3.4. Insert into a Many-to-Many Relationship

1. Ensure `students` and `courses` rows exist.
2. Insert into junction table:

```sql
INSERT INTO student_courses (student_id, course_id)
VALUES (42, 10);
```

### 3.5. Delete or Update Relationships

Remove a student from a course **without** deleting the student or course:

```sql
DELETE FROM student_courses
WHERE student_id = 42
  AND course_id  = 10;
```

---

## 4. Subqueries and CTEs in Multi-Table Context

### 4.1. Subquery in WHERE

```sql
-- Customers whose total order amount is above the average
SELECT
  c.id,
  c.name
FROM customers c
WHERE c.id IN (
  SELECT o.customer_id
  FROM orders o
  GROUP BY o.customer_id
  HAVING SUM(o.total_amount) >
         (SELECT AVG(total_amount) FROM orders)
);
```

### 4.2. Common Table Expressions (CTEs)

CTEs make complex queries more readable:

```sql
WITH order_totals AS (
  SELECT
    o.customer_id,
    SUM(o.total_amount) AS total_spent
  FROM orders o
  GROUP BY o.customer_id
)
SELECT
  c.name,
  ot.total_spent
FROM customers c
JOIN order_totals ot ON ot.customer_id = c.id
WHERE ot.total_spent > 1000
ORDER BY ot.total_spent DESC;
```

---

## 5. SQL Patterns for RESTful APIs

### 5.1. Mapping Resources to Tables

Typical REST mapping:

* **Resource** → table or view (e.g., `/customers`, `/orders`).
* **HTTP methods** ↔ CRUD operations:

  * `GET`    → `SELECT`
  * `POST`   → `INSERT`
  * `PUT`    → `UPDATE` (replace)
  * `PATCH`  → `UPDATE` (partial)
  * `DELETE` → `DELETE`

### 5.2. Example: Customers API

**GET /customers**

```sql
SELECT id, name, email
FROM customers
ORDER BY name;
```

**GET /customers/{id}**

```sql
SELECT id, name, email
FROM customers
WHERE id = :id;
```

**POST /customers**

```sql
INSERT INTO customers (name, email)
VALUES (:name, :email)
RETURNING id;
```

**PUT /customers/{id}**

```sql
UPDATE customers
SET name  = :name,
    email = :email
WHERE id  = :id;
```

**DELETE /customers/{id}**

```sql
DELETE FROM customers
WHERE id = :id;
```

> `:id`, `:name`, etc. are **parameters** from the API layer. Never concatenate raw strings.

### 5.3. Endpoints that Use Joins

**GET /orders/{id}** – order with customer + item details:

```sql
SELECT
  o.id           AS order_id,
  o.order_date,
  c.id           AS customer_id,
  c.name         AS customer_name,
  p.id           AS product_id,
  p.name         AS product_name,
  oi.quantity,
  oi.unit_price,
  (oi.quantity * oi.unit_price) AS line_total
FROM orders o
JOIN customers c   ON c.id = o.customer_id
JOIN order_items oi ON oi.order_id = o.id
JOIN products p     ON p.id = oi.product_id
WHERE o.id = :order_id;
```

Your REST endpoint can then serialize this into JSON like:

```json
{
  "id": 123,
  "order_date": "2025-01-01",
  "customer": {
    "id": 7,
    "name": "Alice"
  },
  "items": [
    { "product_id": 3, "product_name": "Widget", "quantity": 2, "unit_price": 9.99, "line_total": 19.98 }
  ]
}
```

### 5.4. Filtering, Sorting, and Pagination

**Filtering (query params):**

`GET /orders?customer_id=42&min_total=100`

```sql
SELECT
  o.*
FROM orders o
WHERE o.customer_id = :customer_id
  AND o.total_amount >= :min_total
ORDER BY o.order_date DESC;
```

**Sorting (query params):**

`GET /products?sort=price&direction=asc`

```sql
SELECT
  p.*
FROM products p
ORDER BY
  CASE WHEN :sort = 'price' THEN p.price END
  -- add more fields as needed
  ,
  CASE WHEN :direction = 'asc' THEN 1 ELSE 0 END;
```

(Real code usually builds the `ORDER BY` dynamically in the API layer, not via CASE.)

**Pagination (limit/offset):**

`GET /products?limit=20&offset=40`

```sql
SELECT
  p.*
FROM products p
ORDER BY p.id
LIMIT :limit OFFSET :offset;
```

### 5.5. Handling Many-to-Many in REST

For `students` ↔ `courses`:

* `GET /students/{id}/courses`
* `POST /students/{id}/courses` (enroll)
* `DELETE /students/{id}/courses/{course_id}` (unenroll)

**GET /students/{id}/courses**

```sql
SELECT
  c.*
FROM student_courses sc
JOIN courses c ON c.id = sc.course_id
WHERE sc.student_id = :student_id;
```

**POST /students/{id}/courses**

```sql
INSERT INTO student_courses (student_id, course_id)
VALUES (:student_id, :course_id);
```

**DELETE /students/{id}/courses/{course_id}**

```sql
DELETE FROM student_courses
WHERE student_id = :student_id
  AND course_id  = :course_id;
```

### 5.6. N+1 Query Problem

If your API loops and queries inside the loop:

```pseudo
GET /customers:
  customers = SELECT * FROM customers
  for each customer:
     orders = SELECT * FROM orders WHERE customer_id = customer.id
```

This can lead to **N+1 queries** (1 big query + 1 per customer). Instead, use one big join:

```sql
SELECT
  c.id        AS customer_id,
  c.name,
  o.id        AS order_id,
  o.order_date,
  o.total_amount
FROM customers c
LEFT JOIN orders o ON o.customer_id = c.id;
```

Then group/shape the data in the API layer.

### 5.7. Security and Best Practices

* **Use parameterized queries / prepared statements**.
* **Limit columns** you select (no `SELECT *` in public APIs).
* Implement **row-level security** in the API (e.g., user can only see their own data).
* Validate and sanitize query parameters (e.g., allowed sort fields).
* Use least-privilege database users for the API (no `DROP TABLE` powers).

---

## 6. Quick Reference Summary

* Use **explicit JOINs** (`JOIN ... ON ...`).
* Understand join types:

  * `INNER` (matches only),
  * `LEFT` (all left),
  * `RIGHT` (all right),
  * `FULL` (all rows),
  * `CROSS` (cartesian),
  * **self joins**, **semi/anti joins** (`EXISTS`, `NOT EXISTS`).
* Model **many-to-many** with a **junction table** (composite PK of FKs).
* Use **CTEs** (`WITH`) and subqueries to organize complex logic.
* Map REST operations to SQL:

  * `GET` → `SELECT`
  * `POST` → `INSERT`
  * `PUT/PATCH` → `UPDATE`
  * `DELETE` → `DELETE`
* Implement filtering, sorting, and pagination via params.
* Be mindful of:

  * N+1 queries → fix with joins and batched queries.
  * Security: parameterized queries, limited permissions, validation.

```
```
