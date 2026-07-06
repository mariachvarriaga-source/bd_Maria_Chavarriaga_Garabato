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


* **riwi_cities (1) -> (N) riwi_suppliers**: A city can host many unique suppliers.
* **riwi_cities (1) -> (N) riwi_warehouses**: A city can host multiple warehouse hubs.
* **riwi_categories (1) -> (N) riwi_products**: A product classification category covers multiple stock keeping units (SKUs).
* **riwi_suppliers / riwi_warehouses / riwi_products (1) -> (N) riwi_inventory_movements**: Entity units feed multiple inbound and outbound transaction historical lines.

---

## Instructions to Create the Database

1. Open **DBeaver** and connect to your local **PostgreSQL** instance.
2. Open a new SQL Editor window inside your specific database environment (`db_Maria_Chavarriaga_Garabato`).
3. Paste and execute the following DDL script by highlighting the block and pressing **`Alt + X`**:

```sql
CREATE TABLE riwi_cities (
    riwi_city_id SERIAL,
    riwi_city_name VARCHAR(100) NOT NULL UNIQUE,
    PRIMARY KEY (riwi_city_id)
);

CREATE TABLE riwi_categories (
    riwi_category_id SERIAL,
    riwi_category_name VARCHAR(100) NOT NULL UNIQUE,
    PRIMARY KEY (riwi_category_id)
);

CREATE TABLE riwi_suppliers (
    riwi_supplier_id SERIAL,
    riwi_supplier_name VARCHAR(150) NOT NULL UNIQUE,
    riwi_city_id INT NOT NULL,
    PRIMARY KEY (riwi_supplier_id),
    FOREIGN KEY (riwi_city_id) REFERENCES riwi_cities(riwi_city_id)
);

CREATE TABLE riwi_warehouses (
    riwi_warehouse_id SERIAL,
    riwi_warehouse_name VARCHAR(150) NOT NULL UNIQUE,
    riwi_city_id INT NOT NULL,
    PRIMARY KEY (riwi_warehouse_id),
    FOREIGN KEY (riwi_city_id) REFERENCES riwi_cities(riwi_city_id)
);

CREATE TABLE riwi_products (
    riwi_product_id SERIAL,
    riwi_product_name VARCHAR(150) NOT NULL UNIQUE,
    riwi_category_id INT NOT NULL,
    PRIMARY KEY (riwi_product_id),
    FOREIGN KEY (riwi_category_id) REFERENCES riwi_categories(riwi_category_id)
);

CREATE TABLE riwi_inventory_movements (
    riwi_movement_id SERIAL,
    riwi_purchase_order VARCHAR(20) NOT NULL,
    riwi_movement_date DATE NOT NULL,
    riwi_movement_type VARCHAR(10) NOT NULL,
    riwi_quantity INT NOT NULL,
    riwi_unit_price DECIMAL(12, 2) NOT NULL,
    riwi_product_id INT NOT NULL,
    riwi_supplier_id INT NOT NULL,
    riwi_warehouse_id INT NOT NULL,
    PRIMARY KEY (riwi_movement_id),
    FOREIGN KEY (riwi_product_id) REFERENCES riwi_products(riwi_product_id),
    FOREIGN KEY (riwi_supplier_id) REFERENCES riwi_suppliers(riwi_supplier_id),
    FOREIGN KEY (riwi_warehouse_id) REFERENCES riwi_warehouses(riwi_warehouse_id)
);
```

---

## Instructions to Load Data

Clear your active SQL Editor window, paste this clean DML dataset execution script, and trigger it using **`Alt + X`**:

```sql
INSERT INTO riwi_cities (riwi_city_name) VALUES 
('Barranquilla'), ('Cartagena'), ('Santa Marta');

INSERT INTO riwi_categories (riwi_category_name) VALUES 
('Herramienta'), ('Consumible'), ('EPP'), ('Elementos Protección');


