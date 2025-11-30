# PostgreSQL Intervyusidagi Eng Ko'p Beriladigan Savollar va Javoblar

## 1. **PostgreSQL nima va u boshqa ma'lumotlar bazalaridan qanday farq qiladi?**

PostgreSQL - bu kuchli, ochiq kodli, ob'ektga yo'naltirilgan relational database management system (RDBMS). 

**Asosiy xususiyatlari:**
- ACID (Atomicity, Consistency, Isolation, Durability) prinsiplariga to'liq rioya qiladi
- JSONB, Array, hstore kabi murakkab ma'lumot turlarini qo'llab-quvvatlaydi
- Kengaytiriluvchan (custom types, functions, operators)
- Multi-Version Concurrency Control (MVCC)
- Full-text search, GIS ma'lumotlari (PostGIS)

## 2. **Primary Key va Foreign Key o'rtasidagi farq nima?**

**Primary Key:**
- Jadvalning har bir qatorini noyob identifikatsiya qiladi
- NULL bo'lishi mumkin emas
- Bir jadvalda faqat bitta primary key bo'ladi

```sql
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    username VARCHAR(50) NOT NULL
);
```

**Foreign Key:**
- Boshqa jadvaldagi primary key'ga havola qiladi
- Jadvallar o'rtasida bog'lanish yaratadi

```sql
CREATE TABLE orders (
    id SERIAL PRIMARY KEY,
    user_id INTEGER REFERENCES users(id),
    order_date DATE
);
```

## 3. **INNER JOIN, LEFT JOIN, RIGHT JOIN va FULL JOIN farqlari?**

```sql
-- INNER JOIN: Ikkala jadvalda ham mos keladigan qatorlar
SELECT u.username, o.order_date
FROM users u
INNER JOIN orders o ON u.id = o.user_id;

-- LEFT JOIN: Chap jadvaldagi barcha qatorlar + mos keladiganlar
SELECT u.username, o.order_date
FROM users u
LEFT JOIN orders o ON u.id = o.user_id;

-- RIGHT JOIN: O'ng jadvaldagi barcha qatorlar + mos keladiganlar
SELECT u.username, o.order_date
FROM users u
RIGHT JOIN orders o ON u.id = o.user_id;

-- FULL OUTER JOIN: Ikkala jadvaldagi barcha qatorlar
SELECT u.username, o.order_date
FROM users u
FULL OUTER JOIN orders o ON u.id = o.user_id;
```

## 4. **Index nima va qanday turlari bor?**

Index - ma'lumotlarni tezroq qidirish uchun ishlatiladigan ma'lumotlar strukturasi.

**Turlari:**
- **B-tree** (default): Ko'pchilik holatlar uchun
- **Hash**: Faqat tenglik operatorlari uchun
- **GiST**: Geometrik ma'lumotlar
- **GIN**: Full-text search, JSONB
- **BRIN**: Juda katta jadvallar uchun

```sql
-- Oddiy index
CREATE INDEX idx_users_email ON users(email);

-- Unique index
CREATE UNIQUE INDEX idx_users_username ON users(username);

-- Composite index
CREATE INDEX idx_orders_user_date ON orders(user_id, order_date);

-- Partial index
CREATE INDEX idx_active_users ON users(username) WHERE active = true;
```

## 5. **VACUUM nima va nima uchun kerak?**

VACUUM - o'chirilgan yoki yangilangan qatorlar tomonidan band qilingan joyni tozalaydi va statistikani yangilaydi.

```sql
-- Oddiy VACUUM
VACUUM users;

-- VACUUM FULL (jadvalni qayta yozadi)
VACUUM FULL users;

-- VACUUM ANALYZE (statistikani ham yangilaydi)
VACUUM ANALYZE users;

-- AUTOVACUUM (avtomatik)
-- postgresql.conf da sozlanadi
```

## 6. **Transaction nima va ACID xususiyatlari?**

Transaction - bir nechta operatsiyalarni bitta bo'lak sifatida bajarish.

```sql
BEGIN;

UPDATE accounts SET balance = balance - 100 WHERE id = 1;
UPDATE accounts SET balance = balance + 100 WHERE id = 2;

COMMIT; -- yoki ROLLBACK;
```

**ACID:**
- **Atomicity**: Barchasi yoki hech narsa
- **Consistency**: Ma'lumotlar har doim to'g'ri holatda
- **Isolation**: Transactionlar bir-biriga ta'sir qilmaydi
- **Durability**: Commit qilingan ma'lumotlar saqlanadi

## 7. **Isolation Levels nima?**

```sql
-- READ UNCOMMITTED (PostgreSQL qo'llab-quvvatlamaydi)

-- READ COMMITTED (default)
SET TRANSACTION ISOLATION LEVEL READ COMMITTED;

-- REPEATABLE READ
SET TRANSACTION ISOLATION LEVEL REPEATABLE READ;

-- SERIALIZABLE
SET TRANSACTION ISOLATION LEVEL SERIALIZABLE;
```

**Muammolar:**
- Dirty Read
- Non-repeatable Read
- Phantom Read

## 8. **View va Materialized View farqi?**

**View (Virtual jadval):**
```sql
CREATE VIEW active_users AS
SELECT id, username, email
FROM users
WHERE active = true;

SELECT * FROM active_users;
```

**Materialized View (Fizik saqlangan):**
```sql
CREATE MATERIALIZED VIEW user_statistics AS
SELECT 
    DATE(created_at) as date,
    COUNT(*) as user_count
FROM users
GROUP BY DATE(created_at);

-- Ma'lumotlarni yangilash
REFRESH MATERIALIZED VIEW user_statistics;
```

## 9. **Subquery va CTE (Common Table Expression) nima?**

**Subquery:**
```sql
SELECT username
FROM users
WHERE id IN (
    SELECT user_id 
    FROM orders 
    WHERE order_date > '2024-01-01'
);
```

**CTE:**
```sql
WITH recent_orders AS (
    SELECT user_id, COUNT(*) as order_count
    FROM orders
    WHERE order_date > '2024-01-01'
    GROUP BY user_id
)
SELECT u.username, ro.order_count
FROM users u
JOIN recent_orders ro ON u.id = ro.user_id;
```

## 10. **Window Functions nima?**

```sql
-- ROW_NUMBER
SELECT 
    username,
    salary,
    ROW_NUMBER() OVER (ORDER BY salary DESC) as rank
FROM employees;

-- PARTITION BY
SELECT 
    department,
    username,
    salary,
    AVG(salary) OVER (PARTITION BY department) as dept_avg
FROM employees;

-- LAG va LEAD
SELECT 
    date,
    revenue,
    LAG(revenue) OVER (ORDER BY date) as prev_revenue,
    LEAD(revenue) OVER (ORDER BY date) as next_revenue
FROM sales;
```

## 11. **Trigger nima?**

```sql
-- Trigger function yaratish
CREATE OR REPLACE FUNCTION update_modified_column()
RETURNS TRIGGER AS $$
BEGIN
    NEW.updated_at = NOW();
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

-- Trigger yaratish
CREATE TRIGGER update_users_modtime
    BEFORE UPDATE ON users
    FOR EACH ROW
    EXECUTE FUNCTION update_modified_column();
```

## 12. **Explain va Explain Analyze nima?**

```sql
-- Query rejasini ko'rish
EXPLAIN SELECT * FROM users WHERE email = 'test@example.com';

-- Amaliy bajarilish statistikasi
EXPLAIN ANALYZE 
SELECT u.username, COUNT(o.id)
FROM users u
LEFT JOIN orders o ON u.id = o.user_id
GROUP BY u.username;
```

## 13. **Normalizatsiya nima?**

Ma'lumotlar bazasini dizayn qilishda takrorlanishni kamaytirish jarayoni.

**Normal formalar:**
- **1NF**: Har bir ustun atomik qiymatga ega
- **2NF**: 1NF + partial dependency yo'q
- **3NF**: 2NF + transitive dependency yo'q
- **BCNF**: Kuchli 3NF

## 14. **JSONB bilan ishlash:**

```sql
-- JSONB ustun yaratish
CREATE TABLE products (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100),
    attributes JSONB
);

-- Ma'lumot qo'shish
INSERT INTO products (name, attributes)
VALUES ('Laptop', '{"brand": "Dell", "ram": "16GB", "storage": "512GB"}');

-- JSONB dan qidirish
SELECT * FROM products 
WHERE attributes->>'brand' = 'Dell';

-- JSONB index
CREATE INDEX idx_products_attributes ON products USING GIN (attributes);

-- JSONB operatorlari
SELECT * FROM products WHERE attributes ? 'ram';
SELECT * FROM products WHERE attributes @> '{"brand": "Dell"}';
```

## 15. **Performance Optimization:**

```sql
-- Index qo'shish
CREATE INDEX CONCURRENTLY idx_users_email ON users(email);

-- Query optimization
ANALYZE users; -- statistikani yangilash

-- Partitioning
CREATE TABLE orders (
    id SERIAL,
    order_date DATE,
    amount DECIMAL
) PARTITION BY RANGE (order_date);

CREATE TABLE orders_2024_q1 PARTITION OF orders
    FOR VALUES FROM ('2024-01-01') TO ('2024-04-01');

-- Connection pooling
-- pg_bouncer yoki application-level pooling
```

## 16. **Backup va Recovery:**

```sql
-- Backup
pg_dump -U username -d database_name > backup.sql
pg_dump -U username -d database_name -F c > backup.dump

-- Restore
psql -U username -d database_name < backup.sql
pg_restore -U username -d database_name backup.dump

-- Point-in-time recovery (PITR)
-- WAL (Write-Ahead Logging) dan foydalanish
```

## 17. **Replication turlari:**

- **Streaming Replication**: Real-time ma'lumotlar ko'chirish
- **Logical Replication**: Table-level replication
- **Synchronous vs Asynchronous**: Ma'lumotlar bir vaqtda yoki kechikish bilan

Bu savollar PostgreSQL bilan ishlashda zarur bo'lgan asosiy bilimlarni qamrab oladi va intervyularda tez-tez uchraydi.
