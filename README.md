-- =============================================================================
-- 1. SCRIPT DDL: CREACIÓN DE ESTRUCTURAS (EN INGLÉS CON PREFIJO RIWI_)
-- =============================================================================

-- Crear tabla de Ciudades
CREATE TABLE riwi_cities (
    riwi_city_id SERIAL,
    riwi_city_name VARCHAR(100) NOT NULL UNIQUE,
    PRIMARY KEY (riwi_city_id)
);

-- Crear tabla de Categorías
CREATE TABLE riwi_categories (
    riwi_category_id SERIAL,
    riwi_category_name VARCHAR(100) NOT NULL UNIQUE,
    PRIMARY KEY (riwi_category_id)
);

-- Crear tabla de Proveedores
CREATE TABLE riwi_suppliers (
    riwi_supplier_id SERIAL,
    riwi_supplier_name VARCHAR(150) NOT NULL UNIQUE,
    riwi_city_id INT NOT NULL,
    PRIMARY KEY (riwi_supplier_id),
    FOREIGN KEY (riwi_city_id) REFERENCES riwi_cities(riwi_city_id)
);

-- Crear tabla de Bodegas
CREATE TABLE riwi_warehouses (
    riwi_warehouse_id SERIAL,
    riwi_warehouse_name VARCHAR(150) NOT NULL UNIQUE,
    riwi_city_id INT NOT NULL,
    PRIMARY KEY (riwi_warehouse_id),
    FOREIGN KEY (riwi_city_id) REFERENCES riwi_cities(riwi_city_id)
);

-- Crear tabla de Productos
CREATE TABLE riwi_products (
    riwi_product_id SERIAL,
    riwi_product_name VARCHAR(150) NOT NULL UNIQUE,
    riwi_category_id INT NOT NULL,
    PRIMARY KEY (riwi_product_id),
    FOREIGN KEY (riwi_category_id) REFERENCES riwi_categories(riwi_category_id)
);

-- Crear tabla transaccional de Movimientos de Inventario
CREATE TABLE riwi_inventory_movements (
    riwi_movement_id SERIAL,
    riwi_purchase_order VARCHAR(20) NOT NULL,
    riwi_movement_date DATE NOT NULL,
    riwi_movement_type VARCHAR(10) NOT NULL, -- IN / OUT
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


-- =============================================================================
-- 2. SCRIPT DML: CARGA Y LIMPIEZA DE DATOS (CORRIGIENDO INCONSISTENCIAS)
-- =============================================================================

-- Cargar Ciudades unificadas
INSERT INTO riwi_cities (riwi_city_name) VALUES 
('Barranquilla'), 
('Cartagena'), 
('Santa Marta');

-- Cargar Categorías
INSERT INTO riwi_categories (riwi_category_name) VALUES 
('Herramienta'), 
('Consumible'), 
('EPP'), 
('Elementos Protección');

-- Cargar Proveedores corregidos y asociados a sus ciudades
INSERT INTO riwi_suppliers (riwi_supplier_name, riwi_city_id) VALUES 
('Aceros del Norte S.A.S', 2), -- Cartagena
('Industriales SAS', 1),       -- Barranquilla
('Suministros Global SAS', 3); -- Santa Marta

-- Cargar Bodegas asociadas a sus ciudades
INSERT INTO riwi_warehouses (riwi_warehouse_name, riwi_city_id) VALUES 
('Bodega Costa', 3),
('Centro Logístico Norte', 2),
('Bodega Central', 1);

-- Cargar Productos asociados a sus categorías
INSERT INTO riwi_products (riwi_product_name, riwi_category_id) VALUES 
('Disco de Corte 4.5', 1),
('Electrodo E6013', 2),
('Guante Nitrilo', 3),
('Guantes de Nitrilo', 4),
('Soldadura E6013', 2),
('Casco Industrial', 3);

-- Cargar Historial Transaccional de Movimientos
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


-- =============================================================================
-- 3. CONSULTAS SQL OPERACIONALES (REPORTE DE DATOS PARA LA ENTREGA)
-- =============================================================================

-- Reporte 1: Total de unidades y montos monetarios según el tipo de movimiento (IN/OUT)
SELECT 
    riwi_movement_type AS operation_type,
    SUM(riwi_quantity) AS total_units,
    SUM(riwi_quantity * riwi_unit_price) AS total_monetary_value
FROM riwi_inventory_movements
GROUP BY riwi_movement_type;

-- Reporte 2: Ranking de productos con mayor flujo o volumen de movimientos
SELECT 
    p.riwi_product_name AS product_name,
    SUM(m.riwi_quantity) AS units_moved
FROM riwi_inventory_movements m
JOIN riwi_products p ON m.riwi_product_id = p.riwi_product_id
GROUP BY p.riwi_product_name
ORDER BY units_moved DESC;

-- Reporte 3: Movimientos consolidados por Bodega física y su respectiva Ciudad
SELECT 
    w.riwi_warehouse_name AS warehouse_name,
    c.riwi_city_name AS city_location,
    COUNT(m.riwi_movement_id) AS transactional_records,
    SUM(m.riwi_quantity) AS total_volume_handled
FROM riwi_warehouses w
JOIN riwi_cities c ON w.riwi_city_id = c.riwi_city_id
JOIN riwi_inventory_movements m ON w.riwi_warehouse_id = m.riwi_warehouse_id
GROUP BY w.riwi_warehouse_name, c.riwi_city_name;



