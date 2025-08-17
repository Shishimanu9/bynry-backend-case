#Companies
CREATE TABLE Companies (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL
);

# Warehouses
CREATE TABLE Warehouses (
    id SERIAL PRIMARY KEY,
    company_id INT NOT NULL,
    name VARCHAR(100) NOT NULL,
    location VARCHAR(255),
    FOREIGN KEY (company_id) REFERENCES Companies(id)
);

# Products
CREATE TABLE Products (
    id SERIAL PRIMARY KEY,
    sku VARCHAR(50) UNIQUE NOT NULL,
    name VARCHAR(100) NOT NULL,
    price DECIMAL(10,2) NOT NULL,
    is_bundle BOOLEAN DEFAULT FALSE
);

# Inventory (many-to-many between Product and Warehouse)
CREATE TABLE Inventory (
    id SERIAL PRIMARY KEY,
    product_id INT NOT NULL,
    warehouse_id INT NOT NULL,
    quantity INT NOT NULL DEFAULT 0,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (product_id) REFERENCES Products(id),
    FOREIGN KEY (warehouse_id) REFERENCES Warehouses(id),
    UNIQUE(product_id, warehouse_id)
);

# Suppliers
CREATE TABLE Suppliers (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    contact_email VARCHAR(100)
);

# Product-Supplier relation (many-to-many)
CREATE TABLE ProductSuppliers (
    product_id INT NOT NULL,
    supplier_id INT NOT NULL,
    PRIMARY KEY (product_id, supplier_id),
    FOREIGN KEY (product_id) REFERENCES Products(id),
    FOREIGN KEY (supplier_id) REFERENCES Suppliers(id)
);

# Bundles (which products are part of a bundle)
CREATE TABLE BundleItems (
    bundle_id INT NOT NULL,
    product_id INT NOT NULL,
    quantity INT NOT NULL,
    PRIMARY KEY (bundle_id, product_id),
    FOREIGN KEY (bundle_id) REFERENCES Products(id),
    FOREIGN KEY (product_id) REFERENCES Products(id)
);

# Inventory Change Log (to track history)
CREATE TABLE InventoryLogs (
    id SERIAL PRIMARY KEY,
    inventory_id INT NOT NULL,
    change_type VARCHAR(20), -- e.g. "sale", "restock"
    quantity_change INT NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (inventory_id) REFERENCES Inventory(id)
);

*******************Missing Requirements**********************

1.Bundles

If a bundle is sold, should stock automatically reduce from its child products?

Can bundles contain bundles (nested bundles)?

2.Suppliers

Can a product have multiple suppliers?

If yes, how do we prioritize supplier when reordering?

3.Sales Activity

Do we need a separate Sales table to track customer orders?

How to calculate “recent sales activity” for alerts?

4. Multi-company setup

Can suppliers be shared across companies, or are they company-specific?

**************************Justification****************************
1.sku UNIQUE → ensures product uniqueness.
2.Inventory changes tracked in InventoryLogs → helps audit stock levels and generate alerts.
3.is_bundle flag in Products + BundleItems table → allows both normal products and bundles.Schema allows future expansion (e.g., adding Customers or Orders).
