# Experiment 7

## Database Normalization (BCNF Decomposition) and Lossless-Join Verification

### Aim
To decompose a denormalized sales relation into Boyce-Codd Normal Form (BCNF) relations (`Customers`, `Stores`, `Products`, `Orders`, and `OrderDetails`), migrate data, verify functional dependencies, and prove the decomposition is a lossless-join decomposition using set difference operations.

---

### Objectives
- Identify insertion, deletion, and update anomalies present in an unnormalized (`Sales_Denormalized`) relation.
- Analyze functional dependencies (FDs) and decompose the schema into 1NF, 2NF, 3NF, and BCNF.
- Implement normalized relational schemas with appropriate primary keys, foreign keys, and constraints.
- Populate normalized relations dynamically from existing denormalized data using `INSERT INTO ... SELECT DISTINCT`.
- Verify the lossless-join property by comparing the natural join reconstruction of the decomposed relations against the original table using the `EXCEPT` operator.
- Formulate verification queries to validate candidate key uniqueness and ensure no functional dependency violations exist.

---

### Requirements
- **DBMS**: MySQL 8.0+ / PostgreSQL / Oracle SQL Developer
- **Tools**: MySQL Workbench / macOS Terminal Client

---

### Theoretical Background & Functional Dependencies

In the original relation `Sales_Denormalized`:
$$\text{Sales\_Denormalized}(\underline{\text{OrderID}}, \text{OrderDate}, \text{CustomerID}, \text{CustomerName}, \text{CustomerCity}, \text{StoreID}, \text{StoreLocation}, \underline{\text{ProductID}}, \text{ProductName}, \text{UnitPrice}, \text{Quantity})$$

- **Candidate Key of Denormalized Table**: `(OrderID, ProductID)`
- **Functional Dependencies Identified**:
  1. $\text{CustomerID} \rightarrow \text{CustomerName}, \text{CustomerCity}$ (Transitive via OrderID / Partial dependency)
  2. $\text{StoreID} \rightarrow \text{StoreLocation}$ (Transitive via OrderID / Partial dependency)
  3. $\text{ProductID} \rightarrow \text{ProductName}, \text{UnitPrice}$ (Partial dependency on composite key)
  4. $\text{OrderID} \rightarrow \text{OrderDate}, \text{CustomerID}, \text{StoreID}$ (Partial dependency on composite key)
  5. $(\text{OrderID}, \text{ProductID}) \rightarrow \text{Quantity}$ (Full functional dependency)

Since determinants in dependencies 1 through 4 are not superkeys of the denormalized relation, the original table violates 2NF, 3NF, and BCNF. Decomposing into five dedicated relations ensures that for every non-trivial dependency $X \rightarrow Y$, $X$ is a superkey.

---

### Implementation Code

#### 1. Denormalized Table & Sample Data Setup

```sql
-- Original Denormalized Table for Testing
DROP TABLE IF EXISTS Sales_Denormalized;

CREATE TABLE Sales_Denormalized (
    OrderID INT,
    OrderDate DATE,
    CustomerID INT,
    CustomerName VARCHAR(50),
    CustomerCity VARCHAR(50),
    StoreID INT,
    StoreLocation VARCHAR(50),
    ProductID INT,
    ProductName VARCHAR(50),
    UnitPrice DECIMAL(10, 2),
    Quantity INT
);

-- Populating Sample Data
INSERT INTO Sales_Denormalized VALUES
(101, '2026-03-01', 1, 'Alice', 'New York', 201, 'Downtown', 501, 'Laptop', 1200.00, 1),
(101, '2026-03-01', 1, 'Alice', 'New York', 201, 'Downtown', 502, 'Mouse', 25.00, 2),
(102, '2026-03-02', 2, 'Bob', 'Chicago', 202, 'Uptown', 501, 'Laptop', 1200.00, 1),
(103, '2026-03-03', 1, 'Alice', 'New York', 201, 'Downtown', 503, 'Keyboard', 75.00, 1);

SELECT * FROM Sales_Denormalized;
