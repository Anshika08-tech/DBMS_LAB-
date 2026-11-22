# Experiment 2

## Relational Schema Mapping and Integrity Constraints in MySQL

### Aim
To convert the E-Commerce ER diagram into a normalized relational schema, implement complete DDL `CREATE TABLE` statements with primary keys, foreign keys, `NOT NULL`, `UNIQUE`, `ON DELETE CASCADE`, and `ON DELETE SET NULL` constraints, insert sample data, and demonstrate referential integrity violations.

---

### Objectives
- Map entities, weak entities, multivalued attributes, and specialization hierarchies into relational tables.
- Enforce domain and integrity constraints using `PRIMARY KEY`, `FOREIGN KEY`, `UNIQUE`, and `NOT NULL`.
- Configure referential integrity actions (`ON DELETE CASCADE` and `ON DELETE SET NULL`).
- Populate the database with valid sample records.
- Demonstrate and analyze referential integrity violation scenarios.

---

### Requirements
- **DBMS**: MySQL Server 8.0+ / MariaDB
- **Tools**: MySQL Workbench / Terminal MySQL Client / phpMyAdmin

---

### Relational Schema Mapping

| Relation | Primary Key | Foreign Key(s) | Constraints & Notes |
| :--- | :--- | :--- | :--- |
| **`address`** | `address_id` | *None* | Pincode length check, Not Null fields |
| **`customer`** | `customer_id` | `address_id` → `address(address_id)` | `ON DELETE SET NULL`, `email` UNIQUE |
| **`customer_phone`** | `(customer_id, phone_number)` | `customer_id` → `customer(customer_id)` | `ON DELETE CASCADE` (Multivalued attr) |
| **`seller`** | `seller_id` | *None* | `gstin` UNIQUE, `email` UNIQUE |
| **`category`** | `category_id` | *None* | `category_name` UNIQUE |
| **`product`** | `product_id` | `seller_id` → `seller(seller_id)`,<br>`category_id` → `category(category_id)` | `ON DELETE RESTRICT` / `CASCADE` |
| **`product_electronics`**| `product_id` | `product_id` → `product(product_id)` | `ON DELETE CASCADE` (ISA Specialization) |
| **`product_clothing`** | `product_id` | `product_id` → `product(product_id)` | `ON DELETE CASCADE` (ISA Specialization) |
| **`orders`** | `order_id` | `customer_id` → `customer(customer_id)` | `ON DELETE CASCADE` |
| **`order_item`** | `(order_id, item_no)` | `order_id` → `orders(order_id)`,<br>`product_id` → `product(product_id)` | `ON DELETE CASCADE` (Weak Entity) |
| **`payment`** | `payment_id` | `order_id` → `orders(order_id)` | `ON DELETE CASCADE`, `order_id` UNIQUE |
| **`delivery`** | `delivery_id` | `order_id` → `orders(order_id)` | `ON DELETE CASCADE`, `tracking_number` UNIQUE |

---

### 1. DDL: Table Creation with Constraints

```sql
-- Drop tables if they exist in reverse dependency order
DROP TABLE IF EXISTS delivery;
DROP TABLE IF EXISTS payment;
DROP TABLE IF EXISTS order_item;
DROP TABLE IF EXISTS orders;
DROP TABLE IF EXISTS product_clothing;
DROP TABLE IF EXISTS product_electronics;
DROP TABLE IF EXISTS product;
DROP TABLE IF EXISTS category;
DROP TABLE IF EXISTS seller;
DROP TABLE IF EXISTS customer_phone;
DROP TABLE IF EXISTS customer;
DROP TABLE IF EXISTS address;

-- 1. Address Table
CREATE TABLE address (
    address_id INT PRIMARY KEY AUTO_INCREMENT,
    street VARCHAR(100) NOT NULL,
    city VARCHAR(50) NOT NULL,
    state VARCHAR(50) NOT NULL,
    pincode VARCHAR(6) NOT NULL
);

-- 2. Customer Table (SET NULL on Address Deletion)
CREATE TABLE customer (
    customer_id INT PRIMARY KEY AUTO_INCREMENT,
    first_name VARCHAR(50) NOT NULL,
    last_name VARCHAR(50),
    email VARCHAR(100) UNIQUE NOT NULL,
    address_id INT,
    CONSTRAINT fk_customer_address 
        FOREIGN KEY (address_id) REFERENCES address(address_id) 
        ON DELETE SET NULL
);

-- 3. Customer Phone Table (CASCADE on Customer Deletion)
CREATE TABLE customer_phone (
    customer_id INT,
    phone_number VARCHAR(15),
    PRIMARY KEY (customer_id, phone_number),
    CONSTRAINT fk_phone_customer 
        FOREIGN KEY (customer_id) REFERENCES customer(customer_id) 
        ON DELETE CASCADE
);

-- 4. Seller Table
CREATE TABLE seller (
    seller_id INT PRIMARY KEY AUTO_INCREMENT,
    business_name VARCHAR(100) NOT NULL,
    gstin VARCHAR(15) UNIQUE NOT NULL,
    email VARCHAR(100) UNIQUE NOT NULL,
    phone VARCHAR(15) NOT NULL
);

-- 5. Category Table
CREATE TABLE category (
    category_id INT PRIMARY KEY AUTO_INCREMENT,
    category_name VARCHAR(50) UNIQUE NOT NULL
);

-- 6. Product Base Table
CREATE TABLE product (
    product_id INT PRIMARY KEY AUTO_INCREMENT,
    seller_id INT NOT NULL,
    category_id INT NOT NULL,
    product_name VARCHAR(100) NOT NULL,
    price DECIMAL(10, 2) NOT NULL,
    stock_quantity INT NOT NULL DEFAULT 0,
    CONSTRAINT fk_product_seller 
        FOREIGN KEY (seller_id) REFERENCES seller(seller_id) 
        ON DELETE RESTRICT,
    CONSTRAINT fk_product_category 
        FOREIGN KEY (category_id) REFERENCES category(category_id) 
        ON DELETE RESTRICT
);

-- 7. Specialized Table: Electronics (CASCADE on Product Deletion)
CREATE TABLE product_electronics (
    product_id INT PRIMARY KEY,
    warranty_months INT NOT NULL,
    brand VARCHAR(50) NOT NULL,
    CONSTRAINT fk_electronics_product 
        FOREIGN KEY (product_id) REFERENCES product(product_id) 
        ON DELETE CASCADE
);

-- 8. Specialized Table: Clothing (CASCADE on Product Deletion)
CREATE TABLE product_clothing (
    product_id INT PRIMARY KEY,
    size VARCHAR(10) NOT NULL,
    fabric_type VARCHAR(50) NOT NULL,
    CONSTRAINT fk_clothing_product 
        FOREIGN KEY (product_id) REFERENCES product(product_id) 
        ON DELETE CASCADE
);

-- 9. Orders Table (CASCADE on Customer Deletion)
CREATE TABLE orders (
    order_id INT PRIMARY KEY AUTO_INCREMENT,
    customer_id INT NOT NULL,
    order_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    total_amount DECIMAL(10, 2) NOT NULL,
    CONSTRAINT fk_orders_customer 
        FOREIGN KEY (customer_id) REFERENCES customer(customer_id) 
        ON DELETE CASCADE
);

-- 10. Weak Entity: Order Item (CASCADE on Order Deletion)
CREATE TABLE order_item (
    order_id INT,
    item_no INT,
    product_id INT NOT NULL,
    quantity INT NOT NULL CHECK (quantity > 0),
    unit_price DECIMAL(10, 2) NOT NULL,
    PRIMARY KEY (order_id, item_no),
    CONSTRAINT fk_orderitem_order 
        FOREIGN KEY (order_id) REFERENCES orders(order_id) 
        ON DELETE CASCADE,
    CONSTRAINT fk_orderitem_product 
        FOREIGN KEY (product_id) REFERENCES product(product_id) 
        ON DELETE RESTRICT
);

-- 11. Payment Table (1:1 with Order, CASCADE on Order Deletion)
CREATE TABLE payment (
    payment_id INT PRIMARY KEY AUTO_INCREMENT,
    order_id INT UNIQUE NOT NULL,
    payment_mode ENUM('UPI', 'CREDIT_CARD', 'DEBIT_CARD', 'NET_BANKING', 'COD') NOT NULL,
    payment_status ENUM('PENDING', 'COMPLETED', 'FAILED') DEFAULT 'PENDING',
    payment_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    CONSTRAINT fk_payment_order 
        FOREIGN KEY (order_id) REFERENCES orders(order_id) 
        ON DELETE CASCADE
);

-- 12. Delivery Table (1:1 with Order, CASCADE on Order Deletion)
CREATE TABLE delivery (
    delivery_id INT PRIMARY KEY AUTO_INCREMENT,
    order_id INT UNIQUE NOT NULL,
    courier_partner VARCHAR(50) NOT NULL,
    tracking_number VARCHAR(50) UNIQUE NOT NULL,
    delivery_status ENUM('PROCESSING', 'DISPATCHED', 'DELIVERED') DEFAULT 'PROCESSING',
    estimated_date DATE,
    CONSTRAINT fk_delivery_order 
        FOREIGN KEY (order_id) REFERENCES orders(order_id) 
        ON DELETE CASCADE
);

-- Insert Address
INSERT INTO address (street, city, state, pincode) 
VALUES ('Sector 17', 'Chandigarh', 'Punjab', '160017');

-- Insert Customer
INSERT INTO customer (first_name, last_name, email, address_id) 
VALUES ('Aman', 'Sharma', 'aman.sharma@example.com', 1);

-- Insert Multivalued Customer Phone Numbers
INSERT INTO customer_phone (customer_id, phone_number) 
VALUES (1, '+919876543210'), (1, '+919876543211');

-- Insert Seller
INSERT INTO seller (business_name, gstin, email, phone) 
VALUES ('TechRetail India', '07AAAAA0000A1Z5', 'support@techretail.in', '+911122334455');

-- Insert Category
INSERT INTO category (category_name) 
VALUES ('Electronics');

-- Insert Product
INSERT INTO product (seller_id, category_id, product_name, price, stock_quantity) 
VALUES (1, 1, 'Wireless Noise Cancelling Headphones', 7999.00, 25);

-- Insert Specialized Sub-Entity
INSERT INTO product_electronics (product_id, warranty_months, brand) 
VALUES (1, 12, 'Sony');

-- Insert Order
INSERT INTO orders (customer_id, total_amount) 
VALUES (1, 7999.00);

-- Insert Order Item
INSERT INTO order_item (order_id, item_no, product_id, quantity, unit_price) 
VALUES (1, 1, 1, 1, 7999.00);

-- Insert Payment
INSERT INTO payment (order_id, payment_mode, payment_status) 
VALUES (1, 'UPI', 'COMPLETED');

-- Insert Delivery
INSERT INTO delivery (order_id, courier_partner, tracking_number, delivery_status, estimated_date) 
VALUES (1, 'BlueDart', 'BD123456789IN', 'DISPATCHED', '2026-11-05');

-- Violates fk_orders_customer: Customer ID 999 does not exist
INSERT INTO orders (customer_id, total_amount) 
VALUES (999, 1500.00);

-- Violates UNIQUE constraint on customer.email
INSERT INTO customer (first_name, last_name, email, address_id) 
VALUES ('Rahul', 'Verma', 'aman.sharma@example.com', 1);

-- Violates fk_product_seller: Product 1 relies on Seller 1
DELETE FROM seller WHERE seller_id = 1;

-- Deleting customer 1 cascades to orders, order_item, customer_phone, payment, and delivery
DELETE FROM customer WHERE customer_id = 1;

-- Verification: Related tables will have no rows left
SELECT * FROM orders WHERE customer_id = 1;
SELECT * FROM customer_phone WHERE customer_id = 1;

-- Insert a temporary customer referencing address 1
INSERT INTO customer (first_name, last_name, email, address_id) 
VALUES ('Pooja', 'Singh', 'pooja@example.com', 1);

-- Delete address 1
DELETE FROM address WHERE address_id = 1;

-- Check customer record
SELECT customer_id, first_name, address_id FROM customer WHERE email = 'pooja@example.com';

