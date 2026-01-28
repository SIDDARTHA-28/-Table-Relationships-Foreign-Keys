# -Table-Relationships-Foreign-Keys
Intern understands relational database design and referential integrity enforcement
-- =============================================
-- Simple E-commerce Database - Learning Foreign Keys
-- MySQL script with tables, FK constraints, and test data
-- Run this in MySQL Workbench (or any MySQL client)
-- =============================================

-- 1. Categories (top-level parent table)
CREATE TABLE categories (
    category_id   INT AUTO_INCREMENT PRIMARY KEY,
    name          VARCHAR(100) NOT NULL,
    description   TEXT
) ENGINE=InnoDB;


-- 2. Products (references Category)
CREATE TABLE products (
    product_id    INT AUTO_INCREMENT PRIMARY KEY,
    name          VARCHAR(150) NOT NULL,
    price         DECIMAL(10, 2) NOT NULL,
    stock         INT NOT NULL DEFAULT 0,
    category_id   INT NOT NULL,
    
    FOREIGN KEY (category_id) 
        REFERENCES categories(category_id)
        ON DELETE RESTRICT          -- Prevents deleting a category if products exist
        ON UPDATE CASCADE           -- If category id changes (rare), update here too
) ENGINE=InnoDB;


-- 3. Customers (independent top-level table)
CREATE TABLE customers (
    customer_id   INT AUTO_INCREMENT PRIMARY KEY,
    full_name     VARCHAR(100) NOT NULL,
    email         VARCHAR(120) UNIQUE NOT NULL,
    phone         VARCHAR(20),
    created_at    DATETIME DEFAULT CURRENT_TIMESTAMP
) ENGINE=InnoDB;


-- 4. Orders (references Customer)
CREATE TABLE orders (
    order_id      INT AUTO_INCREMENT PRIMARY KEY,
    customer_id   INT NOT NULL,
    order_date    DATETIME DEFAULT CURRENT_TIMESTAMP,
    total_amount  DECIMAL(12, 2) NOT NULL DEFAULT 0.00,
    status        ENUM('pending', 'processing', 'shipped', 'delivered', 'cancelled') 
                  DEFAULT 'pending',
    
    FOREIGN KEY (customer_id) 
        REFERENCES customers(customer_id)
        ON DELETE RESTRICT          -- Cannot delete customer if they have orders
        ON UPDATE CASCADE
) ENGINE=InnoDB;


-- 5. Order Items (junction / detail table - references Order & Product)
CREATE TABLE order_items (
    order_item_id INT AUTO_INCREMENT PRIMARY KEY,
    order_id      INT NOT NULL,
    product_id    INT NOT NULL,
    quantity      INT NOT NULL CHECK (quantity > 0),
    unit_price    DECIMAL(10, 2) NOT NULL,     -- price at time of order
    
    FOREIGN KEY (order_id) 
        REFERENCES orders(order_id)
        ON DELETE CASCADE           -- If order is deleted → delete its items
    
    FOREIGN KEY (product_id) 
        REFERENCES products(product_id)
        ON DELETE RESTRICT          -- Cannot delete product if it's in any order
        ON UPDATE CASCADE
) ENGINE=InnoDB;


-- =============================================
--               TEST DATA
-- =============================================

-- Insert categories
INSERT INTO categories (name, description) VALUES
('Electronics', 'Gadgets and devices'),
('Books', 'Physical and e-books'),
('Clothing', 'Fashion apparel');


-- Insert products
INSERT INTO products (name, price, stock, category_id) VALUES
('Wireless Mouse', 29.99, 150, 1),
('SQL Cookbook', 45.00, 40, 2),
('T-Shirt - Black', 19.99, 200, 3),
('Bluetooth Speaker', 79.50, 25, 1);


-- Insert customers
INSERT INTO customers (full_name, email, phone) VALUES
('Rahul Sharma', 'rahul.sharma@example.com', '+91-98765-43210'),
('Priya Patel', 'priya.p@example.in', NULL);


-- Insert orders
INSERT INTO orders (customer_id, total_amount, status) VALUES
(1, 109.49, 'processing'),   -- Rahul
(2, 19.99, 'pending');       -- Priya


-- Insert order items
-- Order 1: Rahul bought mouse + speaker
INSERT INTO order_items (order_id, product_id, quantity, unit_price) VALUES
(1, 1, 1, 29.99),     -- Wireless Mouse
(1, 4, 1, 79.50);     -- Bluetooth Speaker

-- Order 2: Priya bought T-Shirt
INSERT INTO order_items (order_id, product_id, quantity, unit_price) VALUES
(2, 3, 1, 19.99);


-- =============================================
--     DEMONSTRATION QUERIES (try these!)
-- =============================================

-- See the full order details (JOINs)
SELECT 
    c.full_name,
    o.order_id,
    o.order_date,
    p.name AS product,
    oi.quantity,
    oi.unit_price,
    (oi.quantity * oi.unit_price) AS subtotal
FROM customers c
JOIN orders o ON c.customer_id = o.customer_id
JOIN order_items oi ON o.order_id = oi.order_id
JOIN products p ON oi.product_id = p.product_id
ORDER BY o.order_id, oi.order_item_id;


-- Try to delete a product that's in an order → should FAIL (RESTRICT)
-- DELETE FROM products WHERE product_id = 1;


-- Try to delete a category that has products → should FAIL (RESTRICT)
-- DELETE FROM categories WHERE category_id = 1;


-- Delete an order → its order_items should disappear automatically (CASCADE)
-- DELETE FROM orders WHERE order_id = 1;
-- SELECT * FROM order_items;   -- should show no rows for order_id=1


-- Show current data
SELECT * FROM categories;
SELECT * FROM products;
SELECT * FROM customers;
SELECT * FROM orders;
SELECT * FROM order_items;
