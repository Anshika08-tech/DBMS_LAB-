Experiment 7
Database Normalization (BCNF Decomposition) and Lossless-Join Verification
Aim
To decompose a denormalized sales relation into Boyce-Codd Normal Form (BCNF) relations (Customers, Stores, Products, Orders, and OrderDetails), migrate data, verify functional dependencies, and prove the decomposition is a lossless-join decomposition using set difference operations.
Objectives
Identify insertion, deletion, and update anomalies present in an unnormalized (Sales_Denormalized) relation.
Analyze functional dependencies (FDs) and decompose the schema into 1NF, 2NF, 3NF, and BCNF.
Implement normalized relational schemas with appropriate primary keys, foreign keys, and constraints.
Populate normalized relations dynamically from existing denormalized data using INSERT INTO ... SELECT DISTINCT.
Verify the lossless-join property by comparing the natural join reconstruction of the decomposed relations against the original table using the EXCEPT operator.
Formulate verification queries to validate candidate key uniqueness and ensure no functional dependency violations exist.
Requirements
DBMS: MySQL 8.0+ / PostgreSQL / Oracle SQL Developer
Tools: MySQL Workbench / macOS Terminal Client
Candidate Key of Denormalized Table: (OrderID,ProductID)
Functional Dependencies Identified:
CustomerID→CustomerName,CustomerCity (Transitive via OrderID / Partial dependency)
StoreID→StoreLocation (Transitive via OrderID / Partial dependency)
ProductID→ProductName,UnitPrice (Partial dependency on composite key)
OrderID→OrderDate,CustomerID,StoreID (Partial dependency on composite key)
(OrderID,ProductID)→Quantity (Full functional dependency)
Since determinants in dependencies 1 through 4 are not superkeys of the denormalized relation, the original table violates 2NF, 3NF, and BCNF. Decomposing into five dedicated relations ensures that for every non-trivial dependency X→Y, X is a superkey.
Implementation Code
1. Denormalized Table & Sample Data Setup
