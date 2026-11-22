# Experiment 8

## Transaction Management, Savepoints, Partial Rollback and Transaction Isolation Levels

### Aim

To demonstrate transaction management in MySQL using COMMIT, ROLLBACK, and SAVEPOINT operations, simulate partial transaction failure, and understand transaction isolation anomalies such as Dirty Read and Non-Repeatable Read using multiple database sessions.

### Objectives

- Understand transaction management and ACID properties.
- Implement transactions using START TRANSACTION and COMMIT.
- Demonstrate SAVEPOINT and ROLLBACK TO SAVEPOINT operations.
- Simulate partial transaction failure using CHECK constraints.
- Understand the concept of Dirty Read and Non-Repeatable Read.
- Demonstrate transaction isolation using multiple MySQL sessions.
- Prevent Dirty Reads by changing the transaction isolation level to READ COMMITTED.

### Requirements

- **DBMS:** MySQL 8.0.16 or above
- **Tools:** MySQL Workbench / macOS Terminal Client
- **Concepts:** Transactions, Savepoints, Rollback, Commit, Isolation Levels, ACID Properties.

---

### Part A: Transactions, Savepoints & Partial Failure Simulation (Single Session)

**Step 1: Create Accounts Table**

```sql
CREATE TABLE Accounts (
    AccNo INT PRIMARY KEY,
    HolderName VARCHAR(50),
    Balance DECIMAL(10,2) CHECK (Balance >= 0)
);
```

**Step 2: Insert Records**

```sql
INSERT INTO Accounts VALUES
(1001, 'Anshika', 10000.00),
(1002, 'Pavni', 5000.00);
```

**Step 3: Start Transaction and Debit Account 1001**

```sql
START TRANSACTION;

UPDATE Accounts
SET Balance = Balance - 2000
WHERE AccNo = 1001;

SAVEPOINT sp_debit_done;
```

**Step 4: Credit Account 1002**

```sql
UPDATE Accounts
SET Balance = Balance + 2000
WHERE AccNo = 1002;

SAVEPOINT sp_credit_done;
```

**Step 5: Simulate Partial Failure**

```sql
UPDATE Accounts
SET Balance = Balance - 50000
WHERE AccNo = 1001;
```

The above statement fails because the CHECK constraint does not allow a negative balance.

**Step 6: Recover Using Savepoint**

```sql
ROLLBACK TO SAVEPOINT sp_credit_done;

RELEASE SAVEPOINT sp_credit_done;

COMMIT;
```

**Step 7: Verify Final Records**

```sql
SELECT * FROM Accounts;
```

**Expected Output:**

| AccNo | HolderName | Balance |
|---|---|---:|
| 1001 | Anshika | 8000.00 |
| 1002 | Pavni | 7000.00 |

**Observation:**

The transaction successfully transferred ₹2000 from Account 1001 to Account 1002. The invalid deduction failed due to the CHECK constraint. Using ROLLBACK TO SAVEPOINT preserved the valid transaction operations, and COMMIT permanently saved the changes.

---

### Part B: Demonstrating Dirty Read & Non-Repeatable Read (Two Sessions)

This experiment requires two separate MySQL terminal tabs or sessions.

#### SESSION 1

**Step 1: Set Isolation Level**

```sql
SET SESSION TRANSACTION ISOLATION LEVEL READ UNCOMMITTED;

START TRANSACTION;
```

**Step 2: Read Uncommitted Balance**

```sql
SELECT Balance
FROM Accounts
WHERE AccNo = 1001;
```

Expected balance: 99999.00

This demonstrates a **Dirty Read** because Session 1 reads data that has not yet been committed by Session 2.

**Step 3: Read Balance Again After Session 2 Rolls Back**

```sql
SELECT Balance
FROM Accounts
WHERE AccNo = 1001;
```

Expected balance: 8000.00

The value has changed because Session 2 rolled back its uncommitted update.

```sql
COMMIT;
```

#### SESSION 2

**Step 1: Start Transaction**

```sql
START TRANSACTION;
```

**Step 2: Update Account Balance Without Committing**

```sql
UPDATE Accounts
SET Balance = 99999
WHERE AccNo = 1001;
```

**Step 3: Rollback Changes**

Execute this after Session 1 performs its first SELECT.

```sql
ROLLBACK;
```

The uncommitted update is discarded, and the original balance is restored.

---

### Fixing the Anomaly

To prevent Dirty Reads, change the isolation level in Session 1:

```sql
SET SESSION TRANSACTION ISOLATION LEVEL READ COMMITTED;
```

Under READ COMMITTED, a transaction can only read committed data, preventing Dirty Reads.

**Note:** READ COMMITTED prevents Dirty Reads but does not completely prevent Non-Repeatable Reads. REPEATABLE READ can be used to prevent Non-Repeatable Reads in MySQL's InnoDB storage engine.

---

### Observations

The experiment demonstrated transaction management using COMMIT, ROLLBACK, and SAVEPOINT. The partial failure simulation showed how savepoints help recover valid operations without cancelling the entire transaction. The two-session experiment demonstrated how READ UNCOMMITTED allows Dirty Reads. Changing the isolation level to READ COMMITTED prevented reading uncommitted changes.

### Outcome

Successfully implemented transaction management, savepoint-based partial rollback, constraint validation, and transaction isolation in MySQL. Observed Dirty Read behaviour and understood how isolation levels improve database consistency and reliability.

---

### Conclusion

This experiment helped understand the importance of transaction management and isolation levels in maintaining database consistency. COMMIT permanently saves changes, ROLLBACK cancels uncommitted changes, and SAVEPOINT allows partial rollback. Isolation levels control how concurrent transactions interact and prevent unwanted data anomalies.
