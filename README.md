# Riwi Inventory Management & Normalization Project

## Project Description
This project focuses on the transformation, cleaning, and normalization of a flat transactional inventory dataset (`raw_data`) into a highly efficient relational database model. The objective is to eliminate redundant information, resolve spelling inconsistencies across entries (such as suppliers and city names), prevent data insertion anomalies, and guarantee data integrity across historical inventory movements.

---

## Technologies Used
* **DBeaver Community Edition (v26.1.1)**: Database administration interface and SQL query editor.
* **SQL (Structured Query Language)**: Used for complete Data Definition Language (DDL) and Data Manipulation Language (DML) workflows.
* **Markdown**: Standard documentation format.

---

## Database Engine Used
* **PostgreSQL**: An advanced, open-source object-relational database management system (RDBMS) configured locally.

---

## Explanation of Normalization Process

### First From Normal (1FN)
* **Action**: Ensured all table attributes contain atomic values (indivisible units). 
* **Justification**: The primary key for the transactional ledger was defined logically as a composite unique identifier (`riwi_purchase_order`, `riwi_movement_date`, `riwi_product_id`), ensuring no repeated groups or duplicated records existed within the system.

### Second Form Normal (2FN)
* **Action**: Extracted partial dependencies where non-key columns depended only on part of the primary key.
* **Justification**: Properties such as supplier location and warehouse identity were determined solely by the purchase order, whereas categories were directly linked to products. Separate tables were created for entities vs. transactional actions.

### Third Form Normal (3FN)
* **Action**: Eliminated transitive dependencies where attributes relied on non-key descriptors.
* **Justification**: City names depended on supplier/warehouse fields instead of the transaction itself. Separate master entities (`riwi_cities` and `riwi_categories`) were established. Changing a city's spelling now requires modifying exactly one single record, eliminating global operational updates.

---

## Database Structure

The schema consists of 6 tables optimized with explicit primary keys (`PK`), foreign keys (`FK`), `NOT NULL` requirements, and `UNIQUE` data constraints:

1. **riwi_cities**: Stores cleansed location references.
2. **riwi_categories**: Stores inventory classification groupings.
3. **riwi_suppliers**: Maps unique supply vendors to their registered operating city.
4. **riwi_warehouses**: Maps unique operating logistical storage sites to a specific city.
5. **riwi_products**: Defines distinct items linked to their structural inventory category.
6. **riwi_inventory_movements**: High-density transaction table logging operational quantities, unit costs, timeline stamps, and directional movements (IN/OUT).

---

## Entity Relationship Model (ERM)



