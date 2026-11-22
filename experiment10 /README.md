# Experiment 10

## SQL Query Performance Optimization Using EXPLAIN, Indexing and Query Rewriting

### Aim

To generate a dataset containing at least 10,000 rows, formulate five unoptimized (slow) SQL queries, analyze their bottlenecks using EXPLAIN, optimize them through indexing and query rewriting, and compare execution times before and after optimization.

### Objectives

- Create an Orders_Perf table containing at least 10,000 records.
- Generate sample data using Recursive CTE.
- Identify inefficient SQL queries and their performance bottlenecks.
- Analyze query execution plans using EXPLAIN and EXPLAIN ANALYZE.
- Understand the impact of non-SARGable functions and leading wildcards.
- Optimize queries using single-column and composite indexes.
- Improve query performance through query rewriting.
- Compare execution time and rows examined before and after optimization.

### Requirements

- **DBMS:** MySQL 8.0.18 or above
- **Tools:** MySQL Workbench / macOS Terminal Client
- **Concepts:** Query Optimization, Indexing, Recursive CTE, EXPLAIN, EXPLAIN ANALYZE, Covering Index, UNION ALL, Subqueries, Joins.

---

### Part A: Dataset Generation (10,000+ Rows)

**Step 1: Create Orders_Perf Table**

```sql
CREATE TABLE Orders_Perf (
    OrderID INT AUTO_INCREMENT PRIMARY KEY,
    CustomerID INT,
    OrderDate DATE,
    Status VARCHAR(20),
    Amount DECIMAL(10,2),
    Email VARCHAR(100)
);
```

**Step 2: Set Recursion Limit**

```sql
SET SESSION cte_max_recursion_depth = 12000;
```

**Step 3: Insert 10,500 Records Using Recursive CTE**

```sql
INSERT INTO Orders_Perf
(CustomerID, OrderDate, Status, Amount, Email)

WITH RECURSIVE seq AS (
    SELECT 1 AS n

    UNION ALL

    SELECT n + 1
    FROM seq
    WHERE n < 10500
)

SELECT
    1 + (n % 1000),
    DATE_ADD('2025-01-01', INTERVAL (n % 700) DAY),
    ELT(1 + (n % 4),
        'COMPLETED', 'PENDING', 'CANCELLED', 'SHIPPED'),
    ROUND(100 + (RAND() * 5000), 2),
    CONCAT('user', n, '@example.com')
FROM seq;
```

**Step 4: Verify Total Records**

```sql
SELECT COUNT(*) AS TotalRecords
FROM Orders_Perf;
```

**Expected Output:**

| TotalRecords |
|---:|
| 10500 |

---

### Part B: Five Slow Queries and Their Optimization

### Query 1: Non-SARGable Function on Indexed Column

**Step 1: Slow Query (Before Optimization)**

```sql
EXPLAIN ANALYZE
SELECT *
FROM Orders_Perf
WHERE YEAR(OrderDate) = 2026;
```

**Bottleneck:**

- YEAR() function is applied to the OrderDate column.
- A normal B-Tree index on OrderDate cannot efficiently perform a direct range lookup for this expression.
- The database may perform a full table scan.

**Step 2: Create Index**

```sql
CREATE INDEX idx_order_date
ON Orders_Perf(OrderDate);
```

**Step 3: Optimized Query (After Optimization)**

```sql
EXPLAIN ANALYZE
SELECT *
FROM Orders_Perf
WHERE OrderDate >= '2026-01-01'
AND OrderDate < '2027-01-01';
```

**Optimization:** Replacing the YEAR() function with a date range allows the optimizer to use an index range scan.

---

### Query 2: Leading Wildcard in LIKE

**Step 1: Slow Query (Before Optimization)**

```sql
EXPLAIN ANALYZE
SELECT *
FROM Orders_Perf
WHERE Email LIKE '%user999@example.com';
```

**Bottleneck:**

- The LIKE condition starts with a leading wildcard (%).
- A normal B-Tree index cannot efficiently locate values using this suffix pattern.
- The query may require scanning the entire table.

**Step 2: Create Index**

```sql
CREATE INDEX idx_email
ON Orders_Perf(Email);
```

**Step 3: Optimized Query (After Optimization)**

```sql
EXPLAIN ANALYZE
SELECT *
FROM Orders_Perf
WHERE Email = 'user999@example.com';
```

**Optimization:** Since the required email is an exact match, using the equality operator allows efficient index lookup.

**Note:** The original suffix search and the optimized exact-match query are equivalent for this dataset's unique email values. For a general suffix search, a normal B-Tree index cannot directly optimize the leading-wildcard pattern.

---

### Query 3: Fetching Unnecessary Columns (SELECT *)

**Step 1: Slow Query (Before Optimization)**

```sql
EXPLAIN ANALYZE
SELECT *
FROM Orders_Perf
WHERE Status = 'COMPLETED'
AND Amount > 4500;
```

**Bottleneck:**

- SELECT * retrieves all columns even when only a few columns may be required.
- More row data may need to be accessed and transferred.
- The query may perform additional table lookups.

**Step 2: Create Composite Covering Index**

```sql
CREATE INDEX idx_status_amount
ON Orders_Perf(Status, Amount);
```

**Step 3: Optimized Query (After Optimization)**

```sql
EXPLAIN ANALYZE
SELECT Status, Amount
FROM Orders_Perf
WHERE Status = 'COMPLETED'
AND Amount > 4500;
```

**Optimization:**

- The composite index supports filtering using Status and Amount.
- Selecting only required columns reduces unnecessary data retrieval.
- The query may use a covering index instead of accessing complete table rows.

---

### Query 4: OR Condition Causing Full Table Scan

**Step 1: Slow Query (Before Optimization)**

```sql
EXPLAIN ANALYZE
SELECT *
FROM Orders_Perf
WHERE CustomerID = 45
OR Status = 'CANCELLED';
```

**Bottleneck:**

- OR conditions involving different columns can make query optimization more difficult.
- Without suitable indexes, the database may perform a full table scan.
- More records may need to be examined.

**Step 2: Create Indexes**

```sql
CREATE INDEX idx_cust_id
ON Orders_Perf(CustomerID);

CREATE INDEX idx_status
ON Orders_Perf(Status);
```

**Step 3: Optimized Query Using UNION ALL**

```sql
EXPLAIN ANALYZE

SELECT *
FROM Orders_Perf
WHERE CustomerID = 45

UNION ALL

SELECT *
FROM Orders_Perf
WHERE Status = 'CANCELLED'
AND (CustomerID <> 45 OR CustomerID IS NULL);
```

**Optimization:**

- UNION ALL separates the OR condition into two independent queries.
- Each query can use its respective index.
- The second condition avoids duplicate records.
- NULL CustomerID records are also preserved.

---

### Query 5: Correlated Subquery in WHERE Clause

**Step 1: Slow Query (Before Optimization)**

```sql
EXPLAIN ANALYZE

SELECT OrderID, CustomerID, Amount
FROM Orders_Perf o1
WHERE Amount > (
    SELECT AVG(Amount)
    FROM Orders_Perf o2
    WHERE o2.CustomerID = o1.CustomerID
);
```

**Bottleneck:**

- The subquery depends on the outer query.
- The average amount is calculated for the corresponding customer.
- Repeated execution of the correlated subquery can increase query processing cost.

**Step 2: Create Index**

```sql
CREATE INDEX idx_customer_amount
ON Orders_Perf(CustomerID, Amount);
```

**Step 3: Optimized Query Using JOIN**

```sql
EXPLAIN ANALYZE

SELECT o.OrderID, o.CustomerID, o.Amount

FROM Orders_Perf o

JOIN (
    SELECT CustomerID, AVG(Amount) AS AvgAmt
    FROM Orders_Perf
    GROUP BY CustomerID
) avg_tbl
ON o.CustomerID = avg_tbl.CustomerID

WHERE o.Amount > avg_tbl.AvgAmt;
```

**Optimization:**

- The correlated subquery is replaced with a JOIN.
- Average amount is calculated once per customer in the derived table.
- The composite index supports grouping and accessing customer amounts.
- This can reduce repeated subquery processing.

---

### Part C: Performance Analysis and Comparison

**Step 1: Update Table Statistics**

```sql
ANALYZE TABLE Orders_Perf;
```

**Step 2: Verify Created Indexes**

```sql
SHOW INDEX FROM Orders_Perf;
```

**Step 3: Compare Execution Plans**

Execute EXPLAIN ANALYZE for all five slow queries and their optimized versions.

Observe the following parameters:

- Actual execution time
- Number of rows examined
- Index usage
- Table scan operations
- Query execution plan

### Performance Comparison

| Query | Optimization Technique |
|---|---|
| Query 1 | Date range and indexing |
| Query 2 | Email indexing and equality search |
| Query 3 | Covering index and column selection |
| Query 4 | Indexing and UNION ALL |
| Query 5 | Composite index and JOIN |

### Observations

The Orders_Perf table was successfully populated with 10,500 records using a Recursive CTE. Five unoptimized SQL queries were executed and analyzed using EXPLAIN ANALYZE. Their bottlenecks were identified, including non-SARGable functions, leading wildcards, unnecessary column retrieval, OR conditions, and correlated subqueries. Appropriate indexes and query rewriting techniques were applied to improve query efficiency. The execution plans were compared to observe changes in index usage, rows examined, and execution time.

### Outcome

Successfully generated a dataset containing 10,500 records, identified performance bottlenecks in five SQL queries, implemented indexing and query rewriting techniques, and analyzed query performance before and after optimization using EXPLAIN ANALYZE.

### Conclusion

This experiment demonstrated the importance of SQL query optimization in improving database performance. Indexing, covering indexes, range-based conditions, UNION ALL, and JOIN operations help reduce unnecessary data scanning and improve query efficiency. EXPLAIN ANALYZE provides useful information about actual query execution plans and helps identify performance bottlenecks.
