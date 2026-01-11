CREATE TABLE users (
    id BIGINT PRIMARY KEY,
    name VARCHAR(100),
    email VARCHAR(150),
    city VARCHAR(50),
    created_at TIMESTAMP
);

CREATE TABLE orders (
    order_id BIGINT PRIMARY KEY,
    user_id BIGINT,
    order_status VARCHAR(20),
    total_amount DECIMAL(10,2),
    created_at TIMESTAMP,
    FOREIGN KEY (user_id) REFERENCES users(id)
);

CREATE TABLE payments (
    payment_id BIGINT PRIMARY KEY,
    order_id BIGINT,
    payment_mode VARCHAR(20),
    payment_status VARCHAR(20),
    paid_amount DECIMAL(10,2),
    created_at TIMESTAMP,
    FOREIGN KEY (order_id) REFERENCES orders(order_id)
);

CREATE INDEX idx_users_city ON users(city);
CREATE INDEX idx_orders_user_id ON orders(user_id);
CREATE INDEX idx_orders_created_at ON orders(created_at);
CREATE INDEX idx_payments_order_id ON payments(order_id);
CREATE INDEX idx_payments_status ON payments(payment_status);


-----------

2️⃣ COMPLEX QUERY #1 – Multi-Join + Filter + Sort
EXPLAIN
SELECT u.name,
       COUNT(o.order_id) AS total_orders,
       SUM(p.paid_amount) AS total_paid
FROM users u
JOIN orders o ON u.id = o.user_id
JOIN payments p ON o.order_id = p.order_id
WHERE u.city = 'Mumbai'
  AND o.created_at >= '2024-01-01'
  AND p.payment_status = 'SUCCESS'
GROUP BY u.name
ORDER BY total_paid DESC
LIMIT 10;

What to Observe

Join order

Index usage on city, created_at, payment_status

Using temporary / Using filesort

3️⃣ COMPLEX QUERY #2 – Subquery (Very Common Interview Case)
EXPLAIN
SELECT *
FROM orders o
WHERE o.total_amount >
      (SELECT AVG(total_amount)
       FROM orders
       WHERE created_at >= '2024-01-01');

Interview Talking Points

Subquery execution

Full scan vs index scan

Can be optimized using CTE or materialized result

4️⃣ COMPLEX QUERY #3 – EXISTS vs IN
Using EXISTS
EXPLAIN
SELECT u.id, u.name
FROM users u
WHERE EXISTS (
    SELECT 1
    FROM orders o
    WHERE o.user_id = u.id
      AND o.total_amount > 5000
);

Using IN
EXPLAIN
SELECT u.id, u.name
FROM users u
WHERE u.id IN (
    SELECT o.user_id
    FROM orders o
    WHERE o.total_amount > 5000
);

Compare Plans

Which uses semi-join

Which scans fewer rows

Optimizer behavior

5️⃣ COMPLEX QUERY #4 – LEFT JOIN + NULL Filter (Tricky One)
EXPLAIN
SELECT u.id, u.name
FROM users u
LEFT JOIN orders o ON u.id = o.user_id
WHERE o.order_id IS NULL;

Meaning

👉 Users who never placed an order

Execution Plan Insight

Join type

Index on orders.user_id critical

6️⃣ COMPLEX QUERY #5 – Pagination Problem (Very Important)
EXPLAIN
SELECT *
FROM orders
ORDER BY created_at DESC
LIMIT 10000, 20;

Red Flag

High offset causes full scan ❌

Optimized Version
EXPLAIN
SELECT *
FROM orders
WHERE created_at < '2024-10-01 10:00:00'
ORDER BY created_at DESC
LIMIT 20;

7️⃣ COMPLEX QUERY #6 – Aggregation with HAVING
EXPLAIN
SELECT user_id,
       COUNT(*) AS order_count,
       SUM(total_amount) AS total_spent
FROM orders
GROUP BY user_id
HAVING SUM(total_amount) > 100000;

Watch For

Temporary table usage

Full scan vs index scan

8️⃣ COMPLEX QUERY #7 – Real BFSI-Style Query
EXPLAIN
SELECT u.id,
       u.name,
       SUM(p.paid_amount) AS total_paid_last_30_days
FROM users u
JOIN orders o ON u.id = o.user_id
JOIN payments p ON o.order_id = p.order_id
WHERE p.payment_status = 'SUCCESS'
  AND p.created_at >= CURRENT_DATE - INTERVAL '30 DAY'
GROUP BY u.id, u.name
ORDER BY total_paid_last_30_days DESC;

9️⃣ Use EXPLAIN ANALYZE (Best Practice)
EXPLAIN ANALYZE
SELECT u.name,
       SUM(p.paid_amount)
FROM users u
JOIN orders o ON u.id = o.user_id
JOIN payments p ON o.order_id = p.order_id
WHERE p.payment_status = 'SUCCESS'
GROUP BY u.name;
