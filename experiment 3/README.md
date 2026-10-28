# Experiment 3

## Employee–Department–Project Schema and Advanced SQL Queries

### Aim
To create an Employee–Department–Project schema with at least 30 employees, 5 departments, and 8 projects, and to write SQL queries demonstrating selection, projection, aggregate functions, GROUP BY, HAVING, CASE expressions, and ORDER BY.

---

### Objectives
- Create relational tables (`Department`, `Employee`, `Project`, `Works_On`) with primary keys and foreign keys.
- Populate tables with realistic organizational data (30 employees, 5 departments, 8 projects).
- Implement basic querying using **Selection** (`WHERE`) and **Projection** (`SELECT`).
- Apply SQL aggregate functions (`COUNT`, `SUM`, `AVG`, `MAX`, `MIN`).
- Group data using `GROUP BY` and filter grouped aggregates using `HAVING`.
- Categorize records dynamically using `CASE` conditional expressions.
- Sort query outputs using `ORDER BY` (Ascending and Descending).

---

### Requirements
- **DBMS**: MySQL Server 8.0+ / MariaDB / Oracle SQL Developer
- **Tools**: MySQL Workbench / Terminal MySQL Client

---

### Database Schema (DDL & DML)

```sql
-- 1. Create Department Table
CREATE TABLE Department (
  dept_id INT PRIMARY KEY,
  dept_name VARCHAR(30) NOT NULL,
  location VARCHAR(30)
);

INSERT INTO Department VALUES (1, 'HR', 'Kanpur');
INSERT INTO Department VALUES (2, 'IT', 'Lucknow');
INSERT INTO Department VALUES (3, 'Finance', 'Delhi');
INSERT INTO Department VALUES (4, 'Marketing', 'Mumbai');
INSERT INTO Department VALUES (5, 'Operations', 'Pune');

-- 2. Create Employee Table
CREATE TABLE Employee (
  emp_id INT PRIMARY KEY,
  emp_name VARCHAR(40) NOT NULL,
  salary INT,
  hire_date DATE,
  dept_id INT,
  FOREIGN KEY (dept_id) REFERENCES Department(dept_id)
);

INSERT INTO Employee VALUES (101,'Aarav Sharma',45000,'2019-03-15',1);
INSERT INTO Employee VALUES (102,'Priya Verma',52000,'2018-07-01',1);
INSERT INTO Employee VALUES (103,'Rohit Gupta',38000,'2021-01-10',1);
INSERT INTO Employee VALUES (104,'Sneha Singh',41000,'2020-09-25',1);
INSERT INTO Employee VALUES (105,'Kabir Mehta',48000,'2017-11-05',1);
INSERT INTO Employee VALUES (106,'Ananya Iyer',85000,'2016-02-14',2);
INSERT INTO Employee VALUES (107,'Vikram Rao',92000,'2015-06-30',2);
INSERT INTO Employee VALUES (108,'Neha Kapoor',78000,'2019-08-19',2);
INSERT INTO Employee VALUES (109,'Arjun Nair',67000,'2020-12-01',2);
INSERT INTO Employee VALUES (110,'Ishita Joshi',72000,'2018-04-22',2);
INSERT INTO Employee VALUES (111,'Manish Yadav',60000,'2022-05-11',2);
INSERT INTO Employee VALUES (112,'Pooja Mishra',65000,'2017-01-17',3);
INSERT INTO Employee VALUES (113,'Rahul Tiwari',71000,'2016-10-09',3);
INSERT INTO Employee VALUES (114,'Divya Pandey',58000,'2019-06-06',3);
INSERT INTO Employee VALUES (115,'Sanjay Dubey',62000,'2018-02-28',3);
INSERT INTO Employee VALUES (116,'Meera Agarwal',55000,'2021-07-13',3);
INSERT INTO Employee VALUES (117,'Karan Malhotra',49000,'2020-03-03',4);
INSERT INTO Employee VALUES (118,'Simran Kaur',53000,'2019-11-21',4);
INSERT INTO Employee VALUES (119,'Aditya Chauhan',46000,'2022-01-05',4);
INSERT INTO Employee VALUES (120,'Tanvi Desai',57000,'2018-09-12',4);
INSERT INTO Employee VALUES (121,'Nikhil Bansal',44000,'2021-04-18',4);
INSERT INTO Employee VALUES (122,'Riya Saxena',51000,'2017-08-08',4);
INSERT INTO Employee VALUES (123,'Harsh Vardhan',39000,'2022-02-20',5);
INSERT INTO Employee VALUES (124,'Kavya Reddy',43000,'2020-06-15',5);
INSERT INTO Employee VALUES (125,'Yash Pathak',47000,'2019-01-29',5);
INSERT INTO Employee VALUES (126,'Shreya Bose',50000,'2018-12-12',5);
INSERT INTO Employee VALUES (127,'Deepak Chopra',36000,'2023-03-01',5);
INSERT INTO Employee VALUES (128,'Anjali Menon',42000,'2021-09-09',5);
INSERT INTO Employee VALUES (129,'Mohit Khanna',56000,'2017-05-25',5);
INSERT INTO Employee VALUES (130,'Swati Jain',40000,'2022-10-10',5);

-- 3. Create Project Table
CREATE TABLE Project (
  proj_id INT PRIMARY KEY,
  proj_name VARCHAR(40) NOT NULL,
  dept_id INT,
  budget INT,
  FOREIGN KEY (dept_id) REFERENCES Department(dept_id)
);

INSERT INTO Project VALUES (1,'Payroll System',1,300000);
INSERT INTO Project VALUES (2,'Recruitment Portal',1,250000);
INSERT INTO Project VALUES (3,'ERP Migration',2,900000);
INSERT INTO Project VALUES (4,'Mobile App',2,700000);
INSERT INTO Project VALUES (5,'Budget Automation',3,400000);
INSERT INTO Project VALUES (6,'Ad Campaign 2025',4,500000);
INSERT INTO Project VALUES (7,'Market Survey',4,200000);
INSERT INTO Project VALUES (8,'Supply Chain Tracker',5,600000);

-- 4. Create Works_On Table
CREATE TABLE Works_On (
  emp_id INT,
  proj_id INT,
  hours INT,
  PRIMARY KEY (emp_id, proj_id),
  FOREIGN KEY (emp_id) REFERENCES Employee(emp_id),
  FOREIGN KEY (proj_id) REFERENCES Project(proj_id)
);

INSERT INTO Works_On VALUES (101,1,120);
INSERT INTO Works_On VALUES (102,1,150);
INSERT INTO Works_On VALUES (102,2,80);
INSERT INTO Works_On VALUES (103,2,100);
INSERT INTO Works_On VALUES (104,2,90);
INSERT INTO Works_On VALUES (105,1,60);
INSERT INTO Works_On VALUES (106,3,200);
INSERT INTO Works_On VALUES (107,3,220);
INSERT INTO Works_On VALUES (107,4,100);
INSERT INTO Works_On VALUES (108,4,180);
INSERT INTO Works_On VALUES (109,3,140);
INSERT INTO Works_On VALUES (110,4,160);
INSERT INTO Works_On VALUES (111,3,90);
INSERT INTO Works_On VALUES (112,5,130);
INSERT INTO Works_On VALUES (113,5,170);
INSERT INTO Works_On VALUES (114,5,110);
INSERT INTO Works_On VALUES (115,5,95);
INSERT INTO Works_On VALUES (116,5,80);
INSERT INTO Works_On VALUES (117,6,140);
INSERT INTO Works_On VALUES (118,6,160);
INSERT INTO Works_On VALUES (119,7,70);
INSERT INTO Works_On VALUES (120,6,125);
INSERT INTO Works_On VALUES (121,7,85);
INSERT INTO Works_On VALUES (122,7,100);
INSERT INTO Works_On VALUES (123,8,90);
INSERT INTO Works_On VALUES (124,8,110);
INSERT INTO Works_On VALUES (125,8,130);
INSERT INTO Works_On VALUES (126,8,150);
INSERT INTO Works_On VALUES (127,8,60);
INSERT INTO Works_On VALUES (129,8,175);
```

---

### SQL Queries & Demonstration

#### Query 1: Selection and Projection
*Retrieve employee names, hire dates, and salaries for employees earning more than ₹60,000.*
```sql
SELECT emp_name, hire_date, salary 
FROM Employee 
WHERE salary > 60000;
```
![Query 1](query1.png)

---

#### Query 2: Aggregate Functions (COUNT, AVG, MAX, MIN, SUM)
*Calculate overall metrics across the employee workforce.*
```sql
SELECT 
    COUNT(*) AS total_employees,
    AVG(salary) AS average_salary,
    MAX(salary) AS maximum_salary,
    MIN(salary) AS minimum_salary,
    SUM(salary) AS total_payroll
FROM Employee;
```
![Query 2](query2.png)

---

#### Query 3: GROUP BY and Aggregate Functions
*Find the total number of employees and average salary per department.*
```sql
SELECT 
    dept_id, 
    COUNT(emp_id) AS employee_count, 
    AVG(salary) AS avg_salary
FROM Employee
GROUP BY dept_id;
```
![Query 3](query3.png)

---

#### Query 4: HAVING Clause
*Display departments that have an average salary greater than ₹50,000.*
```sql
SELECT 
    dept_id, 
    AVG(salary) AS avg_salary
FROM Employee
GROUP BY dept_id
HAVING AVG(salary) > 50000;
```
![Query 4](query4.png)

---

#### Query 5: CASE Expression
*Categorize employees into salary brackets ('High', 'Medium', 'Low').*
```sql
SELECT 
    emp_name, 
    salary,
    CASE 
        WHEN salary >= 70000 THEN 'High Salary'
        WHEN salary BETWEEN 45000 AND 69999 THEN 'Medium Salary'
        ELSE 'Entry/Junior Salary'
    END AS salary_tier
FROM Employee;
```
![Query 5](query5.png)

---

#### Query 6: ORDER BY with Multi-Level Sorting
*List all projects ordered by budget in descending order.*
```sql
SELECT proj_id, proj_name, budget 
FROM Project 
ORDER BY budget DESC;
```
![Query 6](query6.png)

---

### Conclusion
The Employee–Department–Project database schema was constructed with sample data. SQL operations demonstrating selection, projection, aggregate calculations, data categorization using `CASE`, grouping using `GROUP BY` and `HAVING`, and sorting using `ORDER BY` were verified.
