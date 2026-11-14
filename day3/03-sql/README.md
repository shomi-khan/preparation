# SQL for Data Analysis

ধরা যাক tables:
- `customers(id, name, country, created_at)`
- `transactions(id, customer_id, amount, status, created_at)`

## ১. SELECT, WHERE, ORDER BY
```sql
SELECT id, amount, status
FROM transactions
WHERE status = 'success' AND amount > 1000
ORDER BY amount DESC;
```

## ২. GROUP BY ও HAVING
```sql
SELECT customer_id, COUNT(*) AS txn_count, SUM(amount) AS total_amount
FROM transactions
WHERE status = 'success'
GROUP BY customer_id
HAVING COUNT(*) >= 5
ORDER BY total_amount DESC;
```
WHERE grouping-এর আগে row filter করে; HAVING aggregate group filter করে।

## ৩. JOIN
```sql
SELECT c.name, t.amount, t.created_at
FROM customers c
JOIN transactions t ON t.customer_id = c.id;
```
INNER JOIN matched row; LEFT JOIN left side-এর সব row রাখে এবং match না থাকলে right-side column NULL হয়।

## ৪. NULL
NULL সাধারণ equality দিয়ে compare হয় না। `IS NULL` / `IS NOT NULL` ব্যবহার করো। `COUNT(column)` NULL বাদ দেয়; `COUNT(*)` row count করে।

## ৫. CTE
```sql
WITH successful AS (
    SELECT customer_id, amount
    FROM transactions
    WHERE status = 'success'
)
SELECT customer_id, SUM(amount) AS revenue
FROM successful
GROUP BY customer_id;
```
CTE complex query readable করতে সাহায্য করে। Performance database optimizer-এর উপর নির্ভরশীল।

## ৬. Subquery
```sql
SELECT *
FROM customers
WHERE id IN (
    SELECT customer_id
    FROM transactions
    WHERE amount > 5000
);
```

## ৭. Window Functions
```sql
SELECT
  customer_id,
  id,
  amount,
  ROW_NUMBER() OVER (
    PARTITION BY customer_id ORDER BY created_at DESC
  ) AS recent_rank,
  SUM(amount) OVER (
    PARTITION BY customer_id
  ) AS customer_total
FROM transactions;
```
Window function row collapse না করে row-এর পাশাপাশি group context দেয়।

## ৮. Monthly Revenue
PostgreSQL syntax:
```sql
SELECT DATE_TRUNC('month', created_at) AS month,
       SUM(amount) AS revenue
FROM transactions
WHERE status = 'success'
GROUP BY 1
ORDER BY 1;
```
Date function database অনুযায়ী বদলাতে পারে।

## ৯. CASE
```sql
SELECT
  CASE
    WHEN amount >= 10000 THEN 'high'
    WHEN amount >= 1000 THEN 'medium'
    ELSE 'low'
  END AS amount_band,
  COUNT(*) AS txn_count
FROM transactions
GROUP BY 1;
```

## ১০. Index ও Query Performance
Index read দ্রুত করতে পারে, কিন্তু write ও storage cost বাড়ায়। WHERE/JOIN/ORDER BY column-এ index বিবেচনা করো। Query plan দেখতে EXPLAIN/EXPLAIN ANALYZE ব্যবহার করো। SELECT * এড়িয়ে প্রয়োজনীয় column নাও।

## ১১. SQL Injection
User input string concatenate করে SQL বানাবে না। Parameterized query ব্যবহার করবে। Dynamic table/column identifier হলে allowlist দরকার।

## Practice
1. Country-wise customer count।
2. Customer-wise successful revenue।
3. প্রতি customer-এর latest transaction।
4. Monthly revenue trend।
5. দ্বিতীয় সর্বোচ্চ transaction amount।
6. যেসব customer কখনও transaction করেনি।
7. প্রতি মাসে top 3 customer by revenue।
