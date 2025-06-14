# Minimart-data-systems
import sqlite3

def setup_database():
    # Create and connect to the database
    conn = sqlite3.connect('minimart.db')
    cursor = conn.cursor()

    # Create tables
    cursor.execute('''
    CREATE TABLE IF NOT EXISTS customers (
        customer_id INTEGER PRIMARY KEY,
        name TEXT NOT NULL,
        email TEXT,
        phone TEXT
    )
    ''')

    cursor.execute('''
    CREATE TABLE IF NOT EXISTS products (
        product_id INTEGER PRIMARY KEY,
        name TEXT NOT NULL,
        price REAL NOT NULL
    )
    ''')

    cursor.execute('''
    CREATE TABLE IF NOT EXISTS orders (
        order_id INTEGER PRIMARY KEY,
        customer_id INTEGER,
        product_id INTEGER,
        quantity INTEGER,
        order_date DATE,
        FOREIGN KEY (customer_id) REFERENCES customers(customer_id),
        FOREIGN KEY (product_id) REFERENCES products(product_id)
    )
    ''')

    # Add new column to products table (ALTER TABLE)
    try:
        cursor.execute("ALTER TABLE products ADD COLUMN category TEXT")
    except sqlite3.OperationalError:
        # Column may already exist
        pass

    # Insert sample customers
    customers = [
        (1, 'John Doe', 'john@example.com', '555-1234'),
        (2, 'Jane Smith', 'jane@example.com', '555-5678'),
        (3, 'Bob Johnson', 'bob@example.com', '555-9012'),
        (4, 'Alice Brown', 'alice@example.com', '555-3456'),
        (5, 'Charlie Davis', 'charlie@example.com', '555-7890')
    ]
    cursor.executemany('INSERT OR IGNORE INTO customers VALUES (?,?,?,?)', customers)

    # Insert sample products
    products = [
        (101, 'Milk', 2.99, 'Dairy'),
        (102, 'Bread', 1.99, 'Bakery'),
        (103, 'Eggs', 3.49, 'Dairy'),
        (104, 'Apples', 0.99, 'Produce'),
        (105, 'Chicken', 5.99, 'Meat'),
        (106, 'Toothpaste', 3.79, 'Personal Care')
    ]
    cursor.executemany('INSERT OR IGNORE INTO products VALUES (?,?,?,?)', products)

    # Insert sample orders
    orders = [
        (1001, 1, 101, 2, '2023-10-01'),
        (1002, 2, 102, 3, '2023-10-01'),
        (1003, 1, 103, 1, '2023-10-02'),
        (1004, 3, 104, 5, '2023-10-02'),
        (1005, 4, 105, 2, '2023-10-03'),
        (1006, 5, 106, 1, '2023-10-03'),
        (1007, 2, 101, 4, '2023-10-04'),
        (1008, 3, 102, 2, '2023-10-05')
    ]
    cursor.executemany('INSERT OR IGNORE INTO orders VALUES (?,?,?,?,?)', orders)

    # Commit changes and close connection
    conn.commit()
    conn.close()
    print("Database setup completed.")

if __name__ == "__main__":
    setup_database()
    import sqlite3

def run_queries():
    conn = sqlite3.connect('minimart.db')
    cursor = conn.cursor()

    print("1. Orders by John Doe:")
    cursor.execute('''
    SELECT orders.*, customers.name 
    FROM orders 
    JOIN customers ON orders.customer_id = customers.customer_id
    WHERE customers.name = 'John Doe'
    ''')
    print(cursor.fetchall())

    print("\n2. Total number of orders:")
    cursor.execute("SELECT COUNT(*) AS total_orders FROM orders")
    print(cursor.fetchone()[0])

    print("\n3. Total revenue:")
    cursor.execute('''
    SELECT SUM(products.price * orders.quantity) AS total_revenue
    FROM orders
    JOIN products ON orders.product_id = products.product_id
    ''')
    print(cursor.fetchone()[0])

    print("\n4. Order details with customer and product names:")
    cursor.execute('''
    SELECT 
        orders.order_id,
        customers.name AS customer_name,
        products.name AS product_name,
        orders.quantity,
        products.price,
        (products.price * orders.quantity) AS total
    FROM orders
    JOIN customers ON orders.customer_id = customers.customer_id
    JOIN products ON orders.product_id = products.product_id
    ''')
    for row in cursor.fetchall():
        print(row)

    print("\n5. All products and any related orders (LEFT JOIN):")
    cursor.execute('''
    SELECT 
        products.name AS product_name,
        orders.order_id,
        orders.quantity,
        orders.order_date
    FROM products
    LEFT JOIN orders ON products.product_id = orders.product_id
    ''')
    for row in cursor.fetchall():
        print(row)

    conn.close()

if __name__ == "__main__":
    run_queries()
    import sqlite3

def analysis():
    conn = sqlite3.connect('minimart.db')
    cursor = conn.cursor()
    
    # Simulate new order data
    new_orders = [
        {'customer_id': 5, 'product_id': 104, 'quantity': 3},
        {'customer_id': 2, 'product_id': 105, 'quantity': 2},
        {'customer_id': 4, 'product_id': 106, 'quantity': 4}
    ]
    
    print("Customers with large orders:")
    for order in new_orders:
        cursor.execute("SELECT price FROM products WHERE product_id=?", (order['product_id'],))
        price = cursor.fetchone()[0]
        total_value = price * order['quantity']
        
        if order['quantity'] > 3 or total_value > 15:
            cursor.execute("SELECT name FROM customers WHERE customer_id=?", (order['customer_id'],))
            customer_name = cursor.fetchone()[0]
            print(f"- {customer_name}: {order['quantity']} items, Total: ${total_value:.2f}")
    
    # Apply discounts
    products_to_update = [
        {'id': 101, 'price': '3.49'},
        {'id': 103, 'price': '4.29'},
        {'id': 105, 'price': '6.99'}
    ]
    
    print("\nApplying 10% discount to updated prices:")
    for product in products_to_update:
        original_price = float(product['price'])
        discounted_price = original_price * 0.9
        cursor.execute("UPDATE products SET price=? WHERE product_id=?", (discounted_price, product['id']))
        print(f"Product ID {product['id']}: ${original_price:.2f} → ${discounted_price:.2f}")
    
    # Create sales report
    report = {'total_products_sold': 0, 'most_popular_product': '', 'revenue_per_customer': {}}
    
    cursor.execute("SELECT SUM(quantity) FROM orders")
    report['total_products_sold'] = cursor.fetchone()[0] or 0
    
    cursor.execute('''
    SELECT products.name, SUM(orders.quantity) AS total_sold
    FROM orders
    JOIN products ON orders.product_id = products.product_id
    GROUP BY products.name
    ORDER BY total_sold DESC
    LIMIT 1
    ''')
    report['most_popular_product'] = cursor.fetchone()[0]
    
    cursor.execute('''
    SELECT 
        customers.name, 
        SUM(products.price * orders.quantity) AS total_spent
    FROM orders
    JOIN customers ON orders.customer_id = customers.customer_id
    JOIN products ON orders.product_id = products.product_id
    GROUP BY customers.name
    ''')
    for row in cursor.fetchall():
        report['revenue_per_customer'][row[0]] = row[1]
    
    print("\nSales Report:")
    print(f"Total Products Sold: {report['total_products_sold']}")
    print(f"Most Popular Product: {report['most_popular_product']}")
    print("Revenue per Customer:")
    for customer, revenue in report['revenue_per_customer'].items():
        print(f"- {customer}: ${revenue:.2f}")
    
    conn.commit()
    conn.close()

if __name__ == "__main__":
    analysis()
    # MiniMart Data System

This project implements a data management system for MiniMart retail company using SQLite for storage and Python for analysis.

## Features

- SQL database setup with customers, products, and orders tables
- SQL queries for data retrieval and analysis
- Python data analysis and reporting
- Discount application logic
- Sales reporting with key metrics

## Setup

1. Create and populate the database:
   ```bash
   python setup_database.py 
