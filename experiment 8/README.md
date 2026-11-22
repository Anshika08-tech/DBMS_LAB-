Part A: Transactions, Savepoints & Partial Failure Simulation (Single Session)

CREATE TABLE Accounts (
    AccNo INT PRIMARY KEY,
    HolderName VARCHAR(50),
    Balance DECIMAL(10,2) CHECK (Balance >= 0)
);

INSERT INTO Accounts VALUES (1001, 'Anshika', 10000.00), (1002, 'Pavni', 5000.00);

-- Simulate Funds Transfer with Savepoint and Partial Rollback
START TRANSACTION;

-- Step 1: Debit Account 1001
UPDATE Accounts SET Balance = Balance - 2000 WHERE AccNo = 1001;
SAVEPOINT sp_debit_done;

-- Step 2: Credit Account 1002
UPDATE Accounts SET Balance = Balance + 2000 WHERE AccNo = 1002;
SAVEPOINT sp_credit_done;

-- Step 3: Simulate a failed bonus/fee deduction (partial failure)
UPDATE Accounts SET Balance = Balance - 50000 WHERE AccNo = 1001; -- Fails CHECK constraint

-- Recover by rolling back only to the valid transfer state
ROLLBACK TO SAVEPOINT sp_credit_done;
RELEASE SAVEPOINT sp_credit_done;

COMMIT;

SELECT * FROM Accounts; -- Shows 8000 and 7000

Part B: Demonstrating Dirty Read & Non-Repeatable Read (Run across two terminal tabs)

-- ==================== SESSION 1 ====================
SET SESSION TRANSACTION ISOLATION LEVEL READ UNCOMMITTED;
START TRANSACTION;
-- Step 2: Reads uncommitted balance (99999) -> DIRTY READ
SELECT Balance FROM Accounts WHERE AccNo = 1001;

-- Step 4: Reads again after Session 2 rolls back -> NON-REPEATABLE READ
SELECT Balance FROM Accounts WHERE AccNo = 1001;
COMMIT;

-- SESSION 2 
START TRANSACTION;
-- Step 1: Update balance without committing
UPDATE Accounts SET Balance = 99999 WHERE AccNo = 1001;

-- Step 3: Rollback changes (execute after Session 1 runs Step 2)
ROLLBACK;

-- FIXING THE ANOMALY
-- Run in Session 1 to prevent Dirty Reads:
SET SESSION TRANSACTION ISOLATION LEVEL READ COMMITTED;
