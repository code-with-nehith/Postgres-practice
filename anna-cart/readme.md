# AnnaCart

Following is the  **proper e-commerce business example** with **4 tables** and clear **business meaning** for each.

## Business Context

This database is for an **online shopping system**:

* **Users** → customers who register and buy products
* **Products** → items available for sale
* **Orders** → one order per checkout (belongs to one user)
* **Order_Items** → products inside an order (order can have many products)

This design supports:

* One order with **multiple products**


## Users Table (Customers)

Stores customer account information.

```sql
CREATE TABLE users (
    user_id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    email VARCHAR(150) UNIQUE NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

- One user can place **many orders**

## Products Table (Catalog)

Stores products that can be purchased.

```sql
CREATE TABLE products (
    product_id SERIAL PRIMARY KEY,
    product_name VARCHAR(150) NOT NULL,
    price NUMERIC(10, 2) NOT NULL,
    stock INT NOT NULL DEFAULT 0,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

- One product can appear in **many order items**

## Orders Table (Checkout / Invoice)

Represents a **single purchase transaction** by a user.

```sql
CREATE TABLE orders (
    order_id SERIAL PRIMARY KEY,
    user_id INT NOT NULL,
    order_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    status VARCHAR(20) DEFAULT 'PLACED',

    CONSTRAINT fk_orders_user
        FOREIGN KEY (user_id)
        REFERENCES users(user_id)
        ON DELETE CASCADE
);
```

- One order belongs to **one user**
- One order has **many order_items**

## Order_Items Table (Line Items)

Stores **each product inside an order** (quantity + price at purchase time).

```sql
CREATE TABLE order_items (
    order_item_id SERIAL PRIMARY KEY,
    order_id INT NOT NULL,
    product_id INT NOT NULL,
    quantity INT NOT NULL,
    price NUMERIC(10, 2) NOT NULL,

    CONSTRAINT fk_items_order
        FOREIGN KEY (order_id)
        REFERENCES orders(order_id)
        ON DELETE CASCADE,

    CONSTRAINT fk_items_product
        FOREIGN KEY (product_id)
        REFERENCES products(product_id)
);
```

- Stores **price at the time of purchase**
- Even if product price changes later, order history remains correct


## Relationships Summary

```
users (1) ----< orders (1) ----< order_items >---- (1) products
```

| Table                  | Relationship |
| ---------------------- | ------------ |
| users → orders         | One-to-Many  |
| orders → order_items   | One-to-Many  |
| products → order_items | One-to-Many  |

## Real Example (Business Flow)

### Step 1: User places an order

Rahul buys:

* 1 Laptop (₹55,000)
* 2 Mouse (₹500 each)

### orders table:

| order_id | user_id |
| -------- | ------- |
| 1        | 1       |

### order_items table:

| order_id | product_id | quantity | price |
| -------- | ---------- | -------- | ----- |
| 1        | 1 (Laptop) | 1        | 55000 |
| 1        | 2 (Mouse)  | 2        | 500   |


## Sample JOIN Query

### Get full order details:

```sql
SELECT
    u.name,
    o.order_id,
    p.product_name,
    oi.quantity,
    oi.price,
    (oi.quantity * oi.price) AS total_price
FROM orders o
JOIN users u ON o.user_id = u.user_id
JOIN order_items oi ON o.order_id = oi.order_id
JOIN products p ON oi.product_id = p.product_id;
```