# SQL for Data Analysis

Backend ও ML workflow-এ SQL শুধু data বের করার ভাষা নয়। Database-এ filter, join, aggregate ও window calculation pushdown করলে application-এ অপ্রয়োজনীয় data load কমে। Interview-তে syntax-এর পাশাপাশি NULL, join cardinality, index এবং query plan বোঝা জরুরি।

ধরা যাক PostgreSQL-style example-এর জন্য tables:

~~~sql
customers(id, name, country, created_at)
transactions(id, customer_id, amount, status, created_at)
~~~

Dialect অনুযায়ী date function, limit syntax ও কিছু feature বদলাতে পারে।

## ১. SELECT, WHERE এবং ORDER BY কী করে?

~~~sql
SELECT id, amount, status
FROM transactions
WHERE status = 'success'
  AND amount > 1000
ORDER BY amount DESC;
~~~

- SELECT কোন column ফেরত দেবে।
- WHERE কোন row filter করবে।
- ORDER BY result sort করবে।
- ASC ascending, DESC descending।

প্রয়োজনীয় column-ই select করো; অকারণে SELECT * বড় table-এ network ও memory cost বাড়াতে পারে।

## ২. SQL query-এর logical execution order কী?

SQL লিখতে SELECT আগে এলেও logical processing order সাধারণত FROM/JOIN → WHERE → GROUP BY → HAVING → SELECT → ORDER BY → LIMIT।

~~~mermaid
flowchart TD
    A["FROM / JOIN"] --> B["WHERE"]
    B --> C["GROUP BY"]
    C --> D["HAVING"]
    D --> E["SELECT"]
    E --> F["ORDER BY"]
    F --> G["LIMIT"]
~~~

এটি logical model, database engine-এর physical execution plan নয়। Optimizer physical order পরিবর্তন করতে পারে।

## ৩. GROUP BY এবং HAVING-এর পার্থক্য কী?

~~~sql
SELECT
    customer_id,
    COUNT(*) AS txn_count,
    SUM(amount) AS total_amount
FROM transactions
WHERE status = 'success'
GROUP BY customer_id
HAVING COUNT(*) >= 5
ORDER BY total_amount DESC;
~~~

- WHERE grouping-এর আগে individual row filter করে।
- GROUP BY row-কে group করে।
- HAVING aggregate হওয়ার পরে group filter করে।

~~~mermaid
flowchart TD
    A["Source rows"] --> B["WHERE filters rows"]
    B --> C["GROUP BY creates groups"]
    C --> D["Aggregate calculations"]
    D --> E["HAVING filters groups"]
    E --> F["Result"]
~~~

## ৪. JOIN কী? Common JOIN type কী?

~~~sql
SELECT c.name, t.amount, t.created_at
FROM customers c
JOIN transactions t
    ON t.customer_id = c.id;
~~~

- INNER JOIN: দুই side-এ match থাকা row।
- LEFT JOIN: left side-এর সব row; match না থাকলে right column NULL।
- RIGHT JOIN: right side-এর সব row।
- FULL OUTER JOIN: উভয় side-এর unmatched row-ও রাখে।

~~~mermaid
flowchart TD
    A["Join two tables on key"] --> B["INNER: matched rows"]
    A --> C["LEFT: all left + matching right"]
    A --> D["RIGHT: all right + matching left"]
    A --> E["FULL: all rows from both sides"]
~~~

Join-এর আগে key uniqueness ও expected cardinality বুঝে নাও। One-to-many join-এ left row multiple result row-তে expand হতে পারে।

## ৫. LEFT JOIN-এর পরে unmatched row কীভাবে খুঁজব?

~~~sql
SELECT c.id, c.name
FROM customers c
LEFT JOIN transactions t
    ON t.customer_id = c.id
WHERE t.id IS NULL;
~~~

এটি এমন customer খুঁজে যাদের matching transaction নেই। Filter right-side table-এর WHERE clause-এ রাখলে LEFT JOIN unintentionally INNER JOIN-এর মতো হয়ে যেতে পারে। Condition join matching-এর অংশ হলে ON clause-এ রাখার কথা বিবেচনা করো।

## ৬. NULL কীভাবে কাজ করে?

NULL সাধারণ value নয়; unknown/missing value বোঝায়। তাই equality দিয়ে compare করা যায় না।

~~~sql
SELECT *
FROM transactions
WHERE status IS NULL;
~~~

- IS NULL / IS NOT NULL ব্যবহার করো।
- COUNT(*) সব row গোনে।
- COUNT(column) NULL বাদ দিয়ে non-null value গোনে।
- SUM/AVG সাধারণত NULL ignore করে।
- COALESCE(value, fallback) প্রথম non-NULL value দেয়।

SQL-এ NULL comparison-এর ফলে TRUE/FALSE ছাড়াও UNKNOWN হতে পারে। তাই NOT IN এবং NULL-এর interaction বিশেষ সতর্কতার বিষয়।

## ৭. CTE কী?

Common Table Expression (CTE) complex query readable করতে সাহায্য করে।

~~~sql
WITH successful AS (
    SELECT customer_id, amount
    FROM transactions
    WHERE status = 'success'
)
SELECT customer_id, SUM(amount) AS revenue
FROM successful
GROUP BY customer_id;
~~~

CTE query-কে logical step-এ ভাগ করে। এটি সব database-এ materialized temporary table হবে এমন নয়; performance optimizer ও database dialect-এর উপর নির্ভর করে।

## ৮. Subquery এবং EXISTS কখন ব্যবহার করব?

~~~sql
SELECT *
FROM customers c
WHERE EXISTS (
    SELECT 1
    FROM transactions t
    WHERE t.customer_id = c.id
      AND t.amount > 5000
);
~~~

EXISTS matching row আছে কি না পরীক্ষা করে। Related record-এর existence check-এ এটি উপযোগী। IN/EXISTS/JOIN-এর performance database optimizer ও indexes-এর উপর নির্ভর করে; actual plan পরীক্ষা করো।

## ৯. Window Function কী?

Window function row collapse না করে প্রতিটি row-এর পাশে group context-এর calculation দেয়।

~~~sql
SELECT
    customer_id,
    id,
    amount,
    ROW_NUMBER() OVER (
        PARTITION BY customer_id
        ORDER BY created_at DESC
    ) AS recent_rank,
    SUM(amount) OVER (
        PARTITION BY customer_id
    ) AS customer_total
FROM transactions;
~~~

- PARTITION BY: কোন group-এর মধ্যে calculation হবে।
- ORDER BY: window-এর ভিতরে ordering।
- ROW_NUMBER: sequential row number।
- RANK / DENSE_RANK: tie-aware ranking।
- SUM/AVG OVER: running বা group total।

~~~mermaid
flowchart TD
    A["Rows"] --> B["Partition by customer"]
    B --> C["Order within each partition"]
    C --> D["Apply window function"]
    D --> E["Original rows + calculated column"]
~~~

## ১০. প্রতি customer-এর latest transaction কীভাবে বের করব?

~~~sql
WITH ranked AS (
    SELECT
        t.*,
        ROW_NUMBER() OVER (
            PARTITION BY customer_id
            ORDER BY created_at DESC, id DESC
        ) AS row_num
    FROM transactions t
)
SELECT *
FROM ranked
WHERE row_num = 1;
~~~

Tie থাকলে deterministic result-এর জন্য secondary ordering key যোগ করো। শুধু MAX(created_at) join করলে একই timestamp-এর multiple row আসতে পারে।

## ১১. Monthly Revenue কীভাবে বের করব?

PostgreSQL syntax:

~~~sql
SELECT
    DATE_TRUNC('month', created_at) AS month,
    SUM(amount) AS revenue
FROM transactions
WHERE status = 'success'
GROUP BY 1
ORDER BY 1;
~~~

Date function database অনুযায়ী বদলাতে পারে। Timestamp timezone-aware হলে business timezone অনুযায়ী month boundary নির্ধারণ করো।

## ১২. CASE expression কী?

SQL query-এর ভিতরে conditional value তৈরি করতে CASE ব্যবহার হয়।

~~~sql
SELECT
    CASE
        WHEN amount >= 10000 THEN 'high'
        WHEN amount >= 1000 THEN 'medium'
        ELSE 'low'
    END AS amount_band,
    COUNT(*) AS txn_count
FROM transactions
GROUP BY 1;
~~~

CASE দিয়ে category, conditional aggregation এবং custom sorting তৈরি করা যায়।

## ১৩. Index কী এবং কেন দরকার?

Index database-কে নির্দিষ্ট column/key দিয়ে row দ্রুত locate করতে সাহায্য করে। WHERE, JOIN, ORDER BY pattern-এ index উপকারী হতে পারে।

Trade-off:
- Read query দ্রুত হতে পারে।
- Insert/update/delete-এ index maintain করতে হয়, তাই write cost বাড়ে।
- Storage লাগে।
- ভুল/অপ্রয়োজনীয় index query planner-কে সাহায্য নাও করতে পারে।

~~~mermaid
flowchart TD
    A["Query filters on customer_id"] --> B{"Useful index exists?"}
    B -->|হ্যাঁ| C["Index-assisted lookup may be used"]
    B -->|না| D["Planner may scan table"]
    C --> E["Check actual execution plan"]
    D --> E
~~~

Index থাকলেই database অবশ্যই সেটি ব্যবহার করবে এমন নয়। Table size, selectivity ও query cost অনুযায়ী planner সিদ্ধান্ত নেয়।

## ১৪. Composite Index কী?

একাধিক column নিয়ে index তৈরি করা যায়। Column order গুরুত্বপূর্ণ।

উদাহরণ: `(customer_id, created_at)` index customer-specific date range query-তে উপকারী হতে পারে। কিন্তু শুধু created_at filter করা query-তে একই index সবসময় optimal নয়। Database-specific leftmost-prefix behavior বুঝে index design করো।

## ১৫. EXPLAIN এবং EXPLAIN ANALYZE কী?

- EXPLAIN query planner-এর estimated execution plan দেখায়।
- EXPLAIN ANALYZE query execute করে actual timing ও row count দেখায় (PostgreSQL-এ)।

~~~sql
EXPLAIN ANALYZE
SELECT *
FROM transactions
WHERE customer_id = 42;
~~~

Production query-তে EXPLAIN ANALYZE চালানোর আগে side effect ও workload বিবেচনা করো। Slow query optimize করার আগে plan, actual rows, index usage এবং data distribution inspect করো।

## ১৬. SQL Injection কীভাবে prevent করবে?

User input string concatenate করে SQL বানাবে না।

খারাপ:

~~~python
query = "SELECT * FROM users WHERE name = '" + user_input + "'"
~~~

ভালো: parameterized query ব্যবহার করো।

~~~python
cursor.execute(
    "SELECT * FROM users WHERE name = %s",
    (user_input,),
)
~~~

Placeholder syntax database driver অনুযায়ী বদলাতে পারে। Table/column identifier parameter দিয়ে bind করা যায় না; dynamic identifier লাগলে strict allowlist ব্যবহার করো।

## ১৭. Transaction এবং ACID কী?

Transaction একাধিক database operation-কে একটি logical unit হিসেবে execute করে।

ACID:
- Atomicity: সব operation commit হবে, নাহলে rollback।
- Consistency: transaction database constraints বজায় রাখে।
- Isolation: concurrent transaction একে অন্যকে কীভাবে দেখে তা নিয়ন্ত্রণ করে।
- Durability: commit-এর পর data persist থাকে।

~~~mermaid
flowchart TD
    A["BEGIN"] --> B["Execute statements"]
    B --> C{"All successful?"}
    C -->|হ্যাঁ| D["COMMIT"]
    C -->|না| E["ROLLBACK"]
~~~

Isolation level অনুযায়ী dirty read, non-repeatable read ও phantom read-এর মতো behavior নিয়ন্ত্রণ হয়। Database-specific semantics জেনে নাও।

## ১৮. Pagination কীভাবে করবে?

Offset pagination:

~~~sql
SELECT id, amount
FROM transactions
ORDER BY created_at DESC, id DESC
LIMIT 20 OFFSET 40;
~~~

বড় offset-এ database-কে অনেক row skip করতে হতে পারে। Cursor/keyset pagination বড় ordered dataset-এ বেশি efficient হতে পারে:

~~~sql
SELECT id, amount, created_at
FROM transactions
WHERE (created_at, id) < (:last_created_at, :last_id)
ORDER BY created_at DESC, id DESC
LIMIT 20;
~~~

Cursor field-এর ordering stable ও indexed হওয়া জরুরি।

## Quick Interview Revision

| প্রশ্ন | সংক্ষিপ্ত উত্তর |
|---|---|
| WHERE vs HAVING? | Row filter vs aggregate group filter |
| INNER vs LEFT JOIN? | Matched rows vs all left rows plus matches |
| NULL compare? | IS NULL / IS NOT NULL |
| COUNT(*) vs COUNT(col)? | All rows vs non-null values |
| CTE? | Named query expression |
| EXISTS? | Matching row exists কি না |
| Window function? | Row preserve করে group context calculation |
| ROW_NUMBER vs RANK? | Unique sequential number vs ties share rank |
| Index trade-off? | Faster reads, extra storage/write cost |
| Composite index order? | Query predicate/order pattern অনুযায়ী |
| EXPLAIN ANALYZE? | Actual execution behavior measure |
| SQL injection? | Untrusted input query structure বদলায় |
| ACID? | Atomicity, Consistency, Isolation, Durability |
| Offset vs keyset pagination? | Skip rows vs continue from last key |

## Practice

1. Country-wise customer count।
2. Customer-wise successful revenue।
3. প্রতি customer-এর latest transaction।
4. Monthly revenue trend।
5. দ্বিতীয় সর্বোচ্চ transaction amount।
6. যেসব customer কখনও transaction করেনি।
7. প্রতি মাসে top 3 customer by revenue।
8. Customer-এর transaction না থাকলেও customer row রাখার query লেখো।
9. COUNT(*) ও COUNT(amount)-এর difference NULL data দিয়ে দেখাও।
10. Duplicate join key result row count-এ কী প্রভাব ফেলে পরীক্ষা করো।
11. Query plan-এ index scan ও sequential scan compare করো।
12. Parameterized query দিয়ে user input safeভাবে filter করো।
13. Transaction দিয়ে debit/credit pair atomicভাবে update করার example লেখো।
14. Offset ও keyset pagination compare করো।

## Practice করার পদ্ধতি

1. Query লেখার আগে expected row count ও output schema predict করো।
2. NULL, duplicate key, empty result ও tie case test করো।
3. Join cardinality এবং index usage verify করো।
4. Query plan দেখে optimize করো; অনুমান করে index যোগ কোরো না।
5. SQL dialect উল্লেখ করো এবং parameterized query ব্যবহার করো।
