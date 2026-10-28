# Experiment 1

## ER Diagram for an Indian E-Commerce Platform

### Aim
To design an Entity-Relationship (ER) diagram for an Indian e-commerce platform and represent its entities, attributes, relationships, cardinalities, and participation constraints.

---

### Objectives
- Identify the entities and their respective attributes required for an e-commerce database system.
- Define primary keys, foreign keys, and relationships between entities.
- Represent cardinality ratios and participation constraints (total vs. partial participation).
- Identify composite and multivalued attributes.
- Model weak entities and identifying relationships.
- Demonstrate specialization and generalization hierarchies using categorized product lines.

---

### Requirements
- **Software**: Web browser / Diagramming tool (Draw.io, Mermaid.js, or MySQL Workbench).
- **Concepts**: ER Modeling, Relational Schema Design, Integrity Constraints, Normalization.

---

### ER Diagram
The following diagram represents the ER model of the Indian e-commerce platform:

![ER Diagram](Experiment1.png)

---

### Key Structural Mappings

1. **Entities & Primary Keys**:
   - `Customer` (`customer_id` PK)
   - `Product` (`product_id` PK)
   - `Order` (`order_id` PK)
   - `Seller` (`seller_id` PK)
   - `Category` (`category_id` PK)
   - `Payment` (`payment_id` PK)
   - `Delivery` (`delivery_id` PK)
   - `Address` (`address_id` PK)

2. **Weak Entity & Identifying Relationship**:
   - `OrderItem` is a weak entity dependent on `Order`.
   - Identifying Relationship: `contains`
   - Partial Key / Discriminator: `item_no`
   - Primary Key: `(order_id, item_no)`

3. **Composite & Multivalued Attributes**:
   - **Composite Attributes**: `name` (`first_name`, `last_name`), `address` (`street`, `city`, `state`, `pincode`).
   - **Multivalued Attribute**: `phone_number` (a customer or seller can have multiple phone numbers).

4. **Specialization / Generalization (ISA Hierarchy)**:
   - Superclass: `Product`
   - Subclasses: `Electronics` (attributes: `warranty`, `brand`) and `Clothing` (attributes: `size`, `fabric_type`).

5. **Participation Constraints & Cardinality**:
   - `Order` has total participation in `Customer` (Every order must be placed by a customer; 1:N).
   - `Order` has total participation in `Payment` and `Delivery` (1:1).
   - `OrderItem` has total participation in `Order` (An order must have at least one line item; 1:N).

---

### Conclusion
The ER diagram for the Indian e-commerce platform was successfully conceptualized and designed. All essential database design constraints—including primary keys, foreign key references, composite/multivalued attributes, weak entity dependencies, specialization hierarchies, and participation constraints—were accurately modeled to serve as the foundation for a relational schema implementation.
