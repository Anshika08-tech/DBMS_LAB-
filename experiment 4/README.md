# Experiment 4

## Advanced SQL Joins, Subqueries, Set Operations, and Execution Plan Analysis

### Aim
To write SQL queries on the Employee schema using INNER JOIN, LEFT JOIN, self-join, 3-way join, correlated subqueries, EXISTS, and simulated INTERSECT and EXCEPT, and to compare execution plans using EXPLAIN.

---

### Objectives
- Alter existing relational schema to represent hierarchical organizational structures (`manager_id` self-referencing relationship).
- Retrieve combined dataset records using `INNER JOIN`, `LEFT JOIN`, `SELF JOIN`, and multi-table (`3-way`) joins.
- Formulate correlated subqueries evaluated row-by-row against outer queries.
- Utilize existence checks using `EXISTS` and `NOT EXISTS`.
- Simulate relational set operations (`INTERSECT` and `EXCEPT` / `MINUS`) using joins and subquery membership checks.
- Analyze query execution strategies, table scans, key lookups, and index optimization using `EXPLAIN`.

---

### Requirements
- **DBMS**: MySQL Server 8.0+ / MariaDB
- **Tools**: MySQL Workbench / macOS Terminal MySQL Client

---

### Schema Setup & Data Modification

```sql
-- Add self-referencing foreign key column for Manager hierarchy
ALTER TABLE Employee ADD manager_id INT;
ALTER TABLE Employee ADD CONSTRAINT fk_emp_manager FOREIGN KEY (manager_id) REFERENCES Employee(emp_id);

-- Update manager assignments across departments
UPDATE Employee SET manager_id = 102 WHERE dept_id = 1 AND emp_id <> 102;
UPDATE Employee SET manager_id = 107 WHERE dept_id = 2 AND emp_id <> 107;
UPDATE Employee SET manager_id = 113 WHERE dept_id = 3 AND emp_id <> 113;
UPDATE Employee SET manager_id = 118 WHERE dept_id = 4 AND emp_id <> 118;
UPDATE Employee SET manager_id = 129 WHERE dept_id = 5 AND emp_id <> 129;
```

---

### SQL Queries & Implementation

#### Query 1: INNER JOIN
*Retrieve employee details along with their corresponding department name and location.*
```sql
SELECT e.emp_name, e.salary, d.dept_name, d.location
FROM Employee e
INNER JOIN Department d ON e.dept_id = d.dept_id
ORDER BY d.dept_name, e.emp_name;
```

---

#### Query 2: LEFT JOIN (Unassigned Records)
*Identify employees who are currently not assigned to any project.*
```sql
SELECT e.emp_id, e.emp_name, w.proj_id
FROM Employee e
LEFT JOIN Works_On w ON e.emp_id = w.emp_id
WHERE w.proj_id IS NULL;
```

---

#### Query 3: Self-Join
*List all employees alongside their respective reporting managers.*
```sql
SELECT e.emp_name AS employee, m.emp_name AS manager
FROM Employee e
LEFT JOIN Employee m ON e.manager_id = m.emp_id
ORDER BY m.emp_name, e.emp_name;
```

---

#### Query 4: 3-Way Table Join
*Retrieve employee project allocations and budgets for allocations exceeding 150 hours.*
```sql
SELECT e.emp_name, p.proj_name, p.budget, w.hours
FROM Employee e
JOIN Works_On w ON e.emp_id = w.emp_id
JOIN Project p ON w.proj_id = p.proj_id
WHERE w.hours > 150
ORDER BY w.hours DESC;
```


---

#### Query 5: Correlated Subquery
*Find employees whose salary exceeds the average salary of their specific department.*
```sql
SELECT e.emp_name, e.salary, e.dept_id
FROM Employee e
WHERE e.salary > (SELECT AVG(e2.salary)
                  FROM Employee e2
                  WHERE e2.dept_id = e.dept_id)
ORDER BY e.dept_id, e.salary DESC;
```


---

#### Query 6: EXISTS Clause
*Retrieve all departments funding high-budget projects (budget ≥ ₹600,000).*
```sql
SELECT d.dept_name
FROM Department d
WHERE EXISTS (SELECT 1 FROM Project p
              WHERE p.dept_id = d.dept_id AND p.budget >= 600000);
```

---

#### Query 7: NOT EXISTS Clause
*Retrieve all employees who have zero assigned project hours.*
```sql
SELECT e.emp_name
FROM Employee e
WHERE NOT EXISTS (SELECT 1 FROM Works_On w WHERE w.emp_id = e.emp_id);
```


---

#### Query 8: Simulated INTERSECT
*Find employees working concurrently on both Project 1 and Project 2.*
```sql
SELECT DISTINCT e.emp_id, e.emp_name
FROM Employee e
JOIN Works_On w ON e.emp_id = w.emp_id
WHERE w.proj_id = 1
  AND e.emp_id IN (SELECT emp_id FROM Works_On WHERE proj_id = 2);
```


---

#### Query 9: Simulated EXCEPT / MINUS
*Find employees working on Project 1 but NOT on Project 2.*
```sql
SELECT DISTINCT e.emp_id, e.emp_name
FROM Employee e
JOIN Works_On w ON e.emp_id = w.emp_id
WHERE w.proj_id = 1
  AND e.emp_id NOT IN (SELECT emp_id FROM Works_On WHERE proj_id = 2);
```

---

### Execution Plan Analysis (`EXPLAIN`)

#### 1. Correlated Subquery vs Derived Table Join
*Correlated subquery execution plan (dependent row-by-row evaluation):*
```sql
EXPLAIN SELECT e.emp_name, e.salary
FROM Employee e
WHERE e.salary > (SELECT AVG(e2.salary) FROM Employee e2
                  WHERE e2.dept_id = e.dept_id);
```


*Derived table join execution plan (pre-aggregated table optimization):*
```sql
EXPLAIN SELECT e.emp_name, e.salary
FROM Employee e
JOIN (SELECT dept_id, AVG(salary) AS avg_sal
      FROM Employee GROUP BY dept_id) a
  ON e.dept_id = a.dept_id
WHERE e.salary > a.avg_sal;
```

---

#### 2. Query Performance Before vs After Indexing
*Before Index Creation (Full Table Scan - `type: ALL`):*
```sql
EXPLAIN SELECT * FROM Employee WHERE dept_id = 2;
```

*Creating the Index:*
```sql
CREATE INDEX idx_emp_dept ON Employee(dept_id);
```

*After Index Creation (Index Range / Ref Scan - `type: ref`):*
```sql
EXPLAIN SELECT * FROM Employee WHERE dept_id = 2;
```

---

### Conclusion
Complex SQL querying mechanisms were evaluated on the Employee database. Joins (`INNER`, `LEFT`, `SELF`, multi-table) and subquery filtering expressions (`EXISTS`, `NOT EXISTS`, correlated evaluations) were implemented. Relational set operations `INTERSECT` and `EXCEPT` were simulated. Execution plan analysis with `EXPLAIN` verified the performance advantage of derived joins over correlated subqueries and confirmed the transition from full table scans to indexed lookups following index creation on foreign key columns.
