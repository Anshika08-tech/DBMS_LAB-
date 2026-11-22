# Experiment 9

## Performance Benchmarking Using Single-Column and Composite Indexes in MySQL

### Aim

To create a Railway Booking table populated with at least 5,000 records, and benchmark query execution performance before and after creating single-column and composite indexes using the EXPLAIN command.

### Objectives

- Create a Railway_Booking table with appropriate attributes and constraints.
- Populate the table with at least 5,000 records using a Recursive CTE.
- Understand query execution plans using EXPLAIN and EXPLAIN ANALYZE.
- Analyze query performance before indexing.
- Create single-column and composite indexes.
- Compare query execution performance before and after indexing.
- Understand how indexing reduces the number of rows scanned and improves query efficiency.

### Requirements

- **DBMS:** MySQL 8.0.18 or above
- **Tools:** MySQL Workbench / macOS Terminal Client
- **Concepts:** Indexing, Single-Column Index, Composite Index, Recursive CTE, Query Optimization, EXPLAIN, EXPLAIN ANALYZE.

---

### Part A: Creating Railway_Booking Table

**Step 1: Create Railway_Booking Table**

```sql
CREATE TABLE Railway_Booking (
    PNR INT AUTO_INCREMENT PRIMARY KEY,
    PassengerName VARCHAR(50),
    TrainNo INT,
    SourceStation VARCHAR(30),
    DestStation VARCHAR(30),
    TravelDate DATE,
    SeatClass VARCHAR(10),
    BookingStatus VARCHAR(15)
);
```

### Part B: Populating 5,000+ Records Using Recursive CTE

**Step 2: Set Recursion Limit**

```sql
SET SESSION cte_max_recursion_depth = 6000;
```

**Step 3: Insert 5,500 Records**

```sql
INSERT INTO Railway_Booking
(PassengerName, TrainNo, SourceStation, DestStation,
 TravelDate, SeatClass, BookingStatus)

WITH RECURSIVE seq AS (
    SELECT 1 AS n

    UNION ALL

    SELECT n + 1
    FROM seq
    WHERE n < 5500
)

SELECT
    CONCAT('Passenger_', n),
    12000 + (n % 50),
    ELT(1 + (n % 4), 'NDLS', 'CDG', 'LKO', 'BCT'),
    ELT(1 + ((n + 1) % 4), 'NDLS', 'CDG', 'LKO', 'BCT'),
    DATE_ADD('2026-11-01', INTERVAL (n % 60) DAY),
    ELT(1 + (n % 3), 'SL', '3A', '2A'),
    ELT(1 + (n % 2), 'CNF', 'WL')
FROM seq;
```

**Step 4: Verify Total Records**

```sql
SELECT COUNT(*) AS TotalRecords
FROM Railway_Booking;
```

**Expected Output:**

| TotalRecords |
|---:|
| 5500 |

---

### Part C: Benchmarking Before Indexing

**Step 5: Analyze Query Using TrainNo**

```sql
EXPLAIN
SELECT *
FROM Railway_Booking
WHERE TrainNo = 12015;
```

```sql
EXPLAIN ANALYZE
SELECT *
FROM Railway_Booking
WHERE TrainNo = 12015;
```

**Step 6: Analyze Query Using SourceStation and TravelDate**

```sql
EXPLAIN
SELECT *
FROM Railway_Booking
WHERE SourceStation = 'CDG'
AND TravelDate = '2026-11-15';
```

```sql
EXPLAIN ANALYZE
SELECT *
FROM Railway_Booking
WHERE SourceStation = 'CDG'
AND TravelDate = '2026-11-15';
```

**Observation Before Indexing:**

- The database may perform a full table scan.
- The access type may appear as ALL in EXPLAIN output.
- A larger number of rows may need to be examined.
- Query execution can take more time due to scanning multiple records.

---

### Part D: Creating Single-Column and Composite Indexes

**Step 7: Create Single-Column Index**

```sql
CREATE INDEX idx_train_no
ON Railway_Booking(TrainNo);
```

This index improves searching based on the TrainNo column.

**Step 8: Create Composite Index**

```sql
CREATE INDEX idx_src_date
ON Railway_Booking(SourceStation, TravelDate);
```

This composite index improves searching based on SourceStation and TravelDate together.

---

### Part E: Benchmarking After Indexing

**Step 9: Verify Indexes**

```sql
SHOW INDEX FROM Railway_Booking;
```

**Step 10: Update Table Statistics**

```sql
ANALYZE TABLE Railway_Booking;
```

**Step 11: Benchmark TrainNo Query Again**

```sql
EXPLAIN
SELECT *
FROM Railway_Booking
WHERE TrainNo = 12015;
```

```sql
EXPLAIN ANALYZE
SELECT *
FROM Railway_Booking
WHERE TrainNo = 12015;
```

**Step 12: Benchmark Composite Index Query Again**

```sql
EXPLAIN
SELECT *
FROM Railway_Booking
WHERE SourceStation = 'CDG'
AND TravelDate = '2026-11-15';
```

```sql
EXPLAIN ANALYZE
SELECT *
FROM Railway_Booking
WHERE SourceStation = 'CDG'
AND TravelDate = '2026-11-15';
```

**Observation After Indexing:**

- The optimizer can use the created indexes to locate matching records.
- The access type may change from ALL to ref.
- The number of rows examined may decrease.
- Query execution time may improve.
- The composite index supports searches using both indexed columns.

---

### Part F: Performance Comparison

| Parameter | Before Indexing | After Indexing |
|---|---|---|
| Index availability | No secondary indexes | Single and composite indexes |
| Table scanning | May scan entire table | Can use index lookups |
| Rows examined | Generally higher | May be lower |
| Query execution | May take longer | May become faster |
| Query optimization | Limited | Improved |

### Observations

The Railway_Booking table was successfully populated with 5,500 records using a Recursive CTE. Initially, queries were executed without secondary indexes, and their execution plans were examined using EXPLAIN and EXPLAIN ANALYZE. After creating a single-column index on TrainNo and a composite index on SourceStation and TravelDate, the queries were executed again. The execution plans demonstrated how indexes can reduce unnecessary row scanning and improve query performance.

### Outcome

Successfully created and populated a Railway_Booking table with 5,500 records, implemented single-column and composite indexes, and analyzed query execution plans before and after indexing using EXPLAIN and EXPLAIN ANALYZE.

### Conclusion

This experiment demonstrated the importance of indexing in database performance optimization. Single-column indexes improve searching on individual attributes, while composite indexes improve queries involving multiple attributes. EXPLAIN and EXPLAIN ANALYZE help understand query execution plans and evaluate the impact of indexing on database performance.
