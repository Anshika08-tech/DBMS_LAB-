# Experiment 5

## SQL Views, Updatability with CHECK OPTION, and Recursive Common Table Expressions (CTEs)

### Aim
To create SQL views for department salary summary and employee hierarchy, to test the updatability of views, and to implement a recursive CTE that displays reporting chains.

---

### Objectives
- Create complex aggregation views using `GROUP BY`, `JOIN`, and multi-table column projections.
- Construct hierarchical structural views combining self-joins and standard table joins.
- Evaluate view updatability rules (`INSERT`, `UPDATE`, `DELETE`) on single-table vs. aggregated multi-table views.
- Enforce domain integrity on updatable views using `WITH CHECK OPTION`.
- Query metadata using `information_schema.views` to evaluate view properties.
- Design recursive Common Table Expressions (`WITH RECURSIVE`) to traverse top-down and bottom-up organizational reporting hierarchies.

---

### Requirements
- **DBMS**: MySQL Server 8.0+ / MariaDB
- **Tools**: MySQL Workbench / macOS Terminal MySQL Client

---

### Schema Setup & Data Alignment

```sql
-- Align reporting structure so top managers report to employee 107
UPDATE Employee SET manager_id = 107 WHERE emp_id IN (102, 113, 118, 129);
```

---

### SQL Queries & Implementation

#### Query 1: Aggregated Summary View (`dept_salary_summary`)
*Create a view summarizing department headcount, total payroll, average, maximum, and minimum salaries.*
```sql
CREATE VIEW dept_salary_summary AS
SELECT d.dept_id, d.dept_name,
       COUNT(e.emp_id) AS num_employees,
       SUM(e.salary) AS total_salary,
       ROUND(AVG(e.salary), 2) AS avg_salary,
       MAX(e.salary) AS max_salary,
       MIN(e.salary) AS min_salary
FROM Department d
JOIN Employee e ON d.dept_id = e.dept_id
GROUP BY d.dept_id, d.dept_name;

SELECT * FROM dept_salary_summary;
```

---

#### Query 2: Multi-Table Hierarchical View (`employee_hierarchy`)
*Create a view displaying employees, their assigned departments, and their reporting managers.*
```sql
CREATE VIEW employee_hierarchy AS
SELECT e.emp_id, e.emp_name, d.dept_name, m.emp_name AS manager_name
FROM Employee e
JOIN Department d ON e.dept_id = d.dept_id
LEFT JOIN Employee m ON e.manager_id = m.emp_id;

SELECT * FROM employee_hierarchy ORDER BY emp_id;
```

---

#### Query 3: Updatable View with `WITH CHECK OPTION`
*Create a restricted view for the IT department (dept_id = 2) with constraint enforcement.*
```sql
CREATE VIEW emp_it_view AS
SELECT emp_id, emp_name, salary, dept_id
FROM Employee
WHERE dept_id = 2
WITH CHECK OPTION;
```

---

#### Query 4: Testing View Updatability (DML Operations)

1. **Successful Update within Scope:**
   ```sql
   UPDATE emp_it_view SET salary = 61000 WHERE emp_id = 111;
   SELECT emp_id, emp_name, salary FROM Employee WHERE emp_id = 111;
   UPDATE emp_it_view SET salary = 60000 WHERE emp_id = 111;
   ```

2. **Failed Update Violating `WITH CHECK OPTION`:**
   ```sql
   -- Fails: attempting to update dept_id to 3 violates WHERE dept_id = 2
   UPDATE emp_it_view SET dept_id = 3 WHERE emp_id = 111;
   ```
   > **Observed MySQL Error**:  
   > `ERROR 1369 (HY000): CHECK OPTION failed 'dbms_lab.emp_it_view'`

3. **Inserting & Deleting Through View:**
   ```sql
   INSERT INTO emp_it_view VALUES (131, 'Test User', 50000, 2);
   SELECT * FROM Employee WHERE emp_id = 131;
   DELETE FROM emp_it_view WHERE emp_id = 131;
   ```
   

4. **Attempting Update on Aggregated View (Non-Updatable):**
   ```sql
   -- Fails: Views containing GROUP BY or aggregate functions cannot be updated
   UPDATE dept_salary_summary SET total_salary = 0 WHERE dept_id = 1;
   ```
   > **Observed MySQL Error**:  
   > `ERROR 1288 (HY000): The target table dept_salary_summary of the UPDATE is not updatable`

---

#### Query 5: Inspecting View Updatability Metadata
*Check view properties stored within the system catalog.*
```sql
SELECT table_name, is_updatable
FROM information_schema.views
WHERE table_schema = DATABASE();
```


---

#### Query 6: Recursive CTE — Top-Down Reporting Hierarchy Path
*Trace complete reporting chains starting from the root manager down through all levels.*
```sql
WITH RECURSIVE chain AS (
  SELECT emp_id, emp_name, manager_id, 1 AS level,
         CAST(emp_name AS CHAR(300)) AS chain_path
  FROM Employee
  WHERE manager_id IS NULL
  UNION ALL
  SELECT e.emp_id, e.emp_name, e.manager_id, c.level + 1,
         CONCAT(c.chain_path, ' -> ', e.emp_name)
  FROM Employee e
  JOIN chain c ON e.manager_id = c.emp_id
)
SELECT emp_id, emp_name, level, chain_path
FROM chain
ORDER BY chain_path;
```


---

#### Query 7: Recursive CTE — Bottom-Up Chain of Command
*Trace the upward reporting hierarchy for a specific employee (`emp_id = 123`).*
```sql
WITH RECURSIVE up AS (
  SELECT emp_id, emp_name, manager_id, 1 AS step
  FROM Employee WHERE emp_id = 123
  UNION ALL
  SELECT e.emp_id, e.emp_name, e.manager_id, u.step + 1
  FROM Employee e
  JOIN up u ON e.emp_id = u.manager_id
)
SELECT step, emp_id, emp_name FROM up;
```


---

#### Query 8: Recursive CTE — Employee Count by Hierarchy Level
*Aggregate total employees assigned at each reporting tier.*
```sql
WITH RECURSIVE chain AS (
  SELECT emp_id, 1 AS level FROM Employee WHERE manager_id IS NULL
  UNION ALL
  SELECT e.emp_id, c.level + 1
  FROM Employee e JOIN chain c ON e.manager_id = c.emp_id
)
SELECT level, COUNT(*) AS employees FROM chain GROUP BY level;
```


---

### Conclusion
Database views were implemented for aggregation abstraction and organizational reporting. View updatability mechanics were demonstrated: simple single-table views allowed updates and inserts while maintaining boundary integrity via `WITH CHECK OPTION`, whereas aggregated multi-table views were verified as non-updatable. Recursive CTEs (`WITH RECURSIVE`) were applied to traverse graph-like reporting hierarchies in both forward (top-down) and reverse (bottom-up) directions.
