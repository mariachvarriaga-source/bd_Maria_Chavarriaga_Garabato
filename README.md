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

INSERT INTO riwi_suppliers (riwi_supplier_name, riwi_city_id) VALUES 
('Aceros del Norte S.A.S', 2), ('Industriales SAS', 1), ('Suministros Global SAS', 3);

INSERT INTO riwi_warehouses (riwi_warehouse_name, riwi_city_id) VALUES 
('Bodega Costa', 3), ('Centro Logístico Norte', 2), ('Bodega Central', 1);

INSERT INTO riwi_products (riwi_product_name, riwi_category_id) VALUES 
('Disco de Corte 4.5', 1), ('Electrodo E6013', 2), ('Guante Nitrilo', 3), 
('Guantes de Nitrilo', 4), ('Soldadura E6013', 2), ('Casco Industrial', 3);

INSERT INTO riwi_inventory_movements (riwi_purchase_order, riwi_movement_date, riwi_movement_type, riwi_quantity, riwi_unit_price, riwi_product_id, riwi_supplier_id, riwi_warehouse_id) VALUES
('PO-1049', '2026-04-01', 'OUT', 148, 115388.00, 1, 1, 1),
('PO-1041', '2026-02-14', 'IN',  27,  35506.00,  2, 1, 1),
('PO-1022', '2026-01-01', 'IN',  70,  14290.00,  3, 2, 1),
('PO-1075', '2026-02-16', 'IN',  160, 117524.00, 4, 1, 2),
('PO-1091', '2026-02-28', 'OUT', 40,  139836.00, 2, 2, 3),
('PO-1041', '2026-03-06', 'IN',  130, 88512.00,  1, 1, 3),
('PO-1059', '2026-01-20', 'OUT', 33,  43746.00,  5, 1, 3),
('PO-1035', '2026-04-13', 'OUT', 119, 23022.00,  3, 2, 1),
('PO-1032', '2026-04-17', 'IN',  185, 123653.00, 4, 3, 3),
('PO-1009', '2026-02-02', 'OUT', 87,  123108.00, 2, 3, 3),
('PO-1040', '2026-05-23', 'IN',  175, 39944.00,  4, 2, 1),
('PO-1023', '2026-03-19', 'OUT', 199, 118291.00, 1, 1, 3),
('PO-1029', '2026-01-25', 'OUT', 131, 71980.00,  3, 2, 2),
('PO-1035', '2026-03-15', 'OUT', 134, 89964.00,  1, 1, 1),
('PO-1094', '2026-03-12', 'IN',  124, 52910.00,  1, 2, 3),
('PO-1046', '2026-04-26', 'IN',  61,  136736.00, 1, 2, 3),
('PO-1043', '2026-03-03', 'OUT', 169, 18022.00,  1, 2, 2),
('PO-1083', '2026-03-21', 'OUT', 192, 108802.00, 6, 1, 1),
('PO-1036', '2026-03-11', 'OUT', 78,  37943.00,  2, 1, 2);
```

---

## Explanation of Each SQL Query

### Query 1: Operational Transaction Metrics
```sql
SELECT 
    riwi_movement_type,
    SUM(riwi_quantity) AS total_units,
    SUM(riwi_quantity * riwi_unit_price) AS total_value
FROM riwi_inventory_movements
GROUP BY riwi_movement_type;
```
* **Explanation**: Aggregates data from the transaction ledger to calculate total units moved and the gross monetary calculation (Quantity × Price) grouped distinctly by operational flow type (`IN` vs `OUT`).

### Query 2: Product Velocity Ranking
```sql
SELECT 
    p.riwi_product_name,
    SUM(m.riwi_quantity) AS units_moved
FROM riwi_inventory_movements m
JOIN riwi_products p ON m.riwi_product_id = p.riwi_product_id
GROUP BY p.riwi_product_name
ORDER BY units_moved DESC;
```
* **Explanation**: Joins the product names table with transactional histories using the foreign key relationship. It tallies total quantities per individual product line and sorts them dynamically from highest to lowest velocity.

### Query 3: Geographical Warehouse Logistical Metrics
```sql
SELECT 
    b.riwi_warehouse_name,
    c.riwi_city_name,
    COUNT(m.riwi_movement_id) AS distinct_operations,
    SUM(m.riwi_quantity) AS dynamic_stock
FROM riwi_warehouses b
JOIN riwi_cities c ON b.riwi_city_id = c.riwi_city_id
JOIN riwi_inventory_movements m ON b.riwi_warehouse_id = m.riwi_warehouse_id
GROUP BY b.riwi_warehouse_name, c.riwi_city_name;
```
* **Explanation**: Resolves multi-table relationships to generate inventory indicators. It counts total transaction events and total volume handled per storage building alongside its respective physical city context.

---

## Developer Information
* **Full Name**: María Chavarriaga Garabato
* **Clan**: [Clan 5 Garabato]

## Entity Relationship Model (ERM)

