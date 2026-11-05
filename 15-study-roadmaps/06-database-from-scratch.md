# Databases From Scratch — Connect the Dots

You said you can't connect the dots on DB. This builds it up from **one mental model** and **one running example**, then bridges to the AWS/RDS stuff you *already* know. Read top to bottom once — it's designed so each piece clicks into the last.

---

## The one mental model (start here)
A relational database is just **spreadsheets that can reference each other**, plus a **query engine**.
- A **table** = a spreadsheet. **Rows** = records (one user, one order). **Columns** = fields (name, email).
- **SQL** = you **describe *what* you want**; the engine figures out **how** to get it. (Sound familiar? It's the same **declarative** idea as Terraform — you already think this way.)
- Tables reference each other by **IDs**, so you can combine them. That "combine" is a **JOIN**, and it's the dot most people miss.

### Our running example (everything below uses this)
```
users                          orders
+----+---------+-----------+   +----+---------+--------+
| id | name    | country   |   | id | user_id | amount |
+----+---------+-----------+   +----+---------+--------+
|  1 | Anshul  | India     |   | 10 |    1    |  500   |
|  2 | Bob     | USA       |   | 11 |    1    |  300   |
|  3 | Chen    | India     |   | 12 |    2    |  900   |
+----+---------+-----------+   +----+---------+--------+
```
`orders.user_id` points at `users.id`. That link is the whole game.

---

## SQL, built up one clause at a time

**SELECT / WHERE — pick columns + filter rows**
```sql
SELECT name, country FROM users WHERE country = 'India';
-- → Anshul/India, Chen/India
```
Read it as: *"give me name and country, from the users table, where country is India."*

**ORDER BY / LIMIT — sort + cap**
```sql
SELECT name, amount FROM orders ORDER BY amount DESC LIMIT 1;
```

**JOIN — the dot that connects two tables**
You want each order *with the user's name*. The name lives in `users`, the amount in `orders`. Join them on the linking IDs:
```sql
SELECT u.name, o.amount
FROM users u
JOIN orders o ON u.id = o.user_id;
-- Anshul 500, Anshul 300, Bob 900
```
- **INNER JOIN** (just `JOIN`) = rows that **match in both** tables.
- **LEFT JOIN** = **all** rows from the left table + matches from the right (NULL where no match). Use it for "all users, even those with no orders."
```sql
SELECT u.name, o.amount
FROM users u
LEFT JOIN orders o ON u.id = o.user_id;
-- ... Chen | NULL   (Chen has no orders)
```

**GROUP BY + aggregates — collapse rows into summaries**
"Total spend per user":
```sql
SELECT user_id, SUM(amount) AS total, COUNT(*) AS num_orders
FROM orders
GROUP BY user_id;
-- user 1 → total 800, 2 orders ; user 2 → total 900, 1 order
```
Aggregates: `COUNT`, `SUM`, `AVG`, `MIN`, `MAX`.

**WHERE vs HAVING — filter *before* vs *after* grouping (classic question)**
```sql
SELECT user_id, COUNT(*) AS n
FROM orders
WHERE amount > 100        -- filters individual rows FIRST
GROUP BY user_id
HAVING COUNT(*) > 1;      -- filters the GROUPS after aggregation
```
**WHERE filters rows; HAVING filters groups.** That one line is a common gotcha — memorize it.

---

## Keys (why tables can trust each other)
- **Primary key (PK):** the column that **uniquely identifies a row** (`users.id`). Unique + not null.
- **Foreign key (FK):** a column that **references another table's PK** (`orders.user_id → users.id`). It enforces **referential integrity** — you can't create an order for a user that doesn't exist.

---

## Indexes — the #1 "why is it slow" concept
**Analogy:** a book without an index → you read every page to find a word (a **full table scan**). A book *with* an index → you jump straight to the page. A DB **index** is that back-of-book index for a column.
```sql
CREATE INDEX idx_orders_user ON orders(user_id);   -- now "find orders for user X" is fast
```
- **Speeds up** `WHERE`, `JOIN`, and `ORDER BY` on the indexed column(s).
- **Costs** extra storage and **slower writes** (every insert/update must also update the index).
- **Composite index** `(a, b)` → helps queries filtering on `a`, or `a AND b` (**leftmost-prefix** rule); doesn't help a query on `b` alone.
- **Covering index** = the index contains all columns a query needs, so it never touches the table.
- **Too many indexes = slow writes.** It's a read-vs-write trade-off.

**How you find a slow query:** `EXPLAIN <query>` shows the **query plan** — look for a **full table scan** or "index not used." That's your signal to add/fix an index or rewrite the query.

---

## Transactions & ACID (money-transfer analogy)
A **transaction** groups steps so they **all succeed or all fail**. Transfer ₹100 from A to B = debit A **and** credit B — if only one happens, money vanishes.
- **ACID:**
  - **A**tomicity — all-or-nothing (both the debit and credit, or neither).
  - **C**onsistency — the DB moves from one valid state to another (rules/constraints hold).
  - **I**solation — concurrent transactions don't corrupt each other.
  - **D**urability — once committed, it survives a crash (on disk).
- **Isolation levels** (higher = safer, less concurrency): Read Uncommitted → **Read Committed** (common default) → **Repeatable Read** (MySQL/InnoDB default) → Serializable. The anomalies they prevent: **dirty read, non-repeatable read, phantom read.**
- **Deadlock** = two transactions each wait on a lock the other holds → the DB kills one. Avoid with **short transactions** and **consistent lock ordering.**

---

## Design: normalization vs denormalization
- **Normalization** = split data to remove redundancy (store a user once, reference by `user_id`). Fewer update bugs. This is our example's design.
- **Denormalization** = deliberately duplicate data (e.g., store `user_name` on the order) to **avoid joins** and make **reads faster** — used for read-heavy/reporting workloads.
- Trade-off: normalized = clean writes; denormalized = fast reads. Pick by workload.

---

## The bridge — you already know the operational half (AWS/RDS)
This is where **your existing knowledge connects** — you've done RDS/DocumentDB/Secrets Manager work, so lead with this in the interview:
- **Multi-AZ vs read replicas** (say it crisply): **Multi-AZ = a synchronous standby in another AZ for automatic failover / HA — NOT readable.** **Read replica = an asynchronous copy you send reads to for scaling — readable, can be cross-region.** Don't confuse them.
- **Backups / PITR:** automated snapshots + transaction logs → point-in-time recovery; test your restores.
- **Connection pooling:** DB connections are expensive to open; apps **pool and reuse** them. **RDS Proxy** helps Lambda/serverless avoid exhausting connections.
- **Scaling reads:** read replicas + a cache (ElastiCache/Redis) in front of hot reads.
- **SQL vs NoSQL:** relational (joins, ACID, structured) vs NoSQL/DynamoDB (flexible schema, horizontal scale, simple key-based access patterns) — pick by access pattern.

---

## "Debug a slow query" — the playbook (a likely question)
1. **Find it** — slow query log / **RDS Performance Insights**.
2. **`EXPLAIN`** it — full table scan? index not used? bad join order? huge row estimate?
3. **Fix** — add/fix an **index**, or **rewrite** (avoid `SELECT *`, avoid **N+1** where the app fires one query per row instead of a join/batch).
4. **Check** locks/contention and **connection limits**; can the read be **offloaded to a replica or cache**?
5. **Re-measure.**

---

## Interview Q&A (crisp answers)
- **INNER vs LEFT JOIN?** → inner = matches in both; left = all left rows + matches (NULL if none).
- **WHERE vs HAVING?** → WHERE filters rows *before* grouping; HAVING filters *groups* after.
- **What's an index, and the downside?** → speeds reads on that column; costs storage + slows writes; too many hurt.
- **Primary vs foreign key?** → PK uniquely identifies a row; FK references another table's PK (integrity).
- **ACID?** → atomicity, consistency, isolation, durability (money-transfer example).
- **Normalization vs denormalization?** → remove redundancy vs duplicate for read speed.
- **Multi-AZ vs read replica?** → failover/HA (sync, not readable) vs read scaling (async, readable).
- **How do you make a slow query fast?** → the playbook above.

## A few "write a query" ones to have ready
```sql
-- Count orders per user
SELECT user_id, COUNT(*) FROM orders GROUP BY user_id;

-- Users who placed more than 1 order
SELECT user_id, COUNT(*) FROM orders GROUP BY user_id HAVING COUNT(*) > 1;

-- Each order with the user's name (join)
SELECT u.name, o.amount FROM users u JOIN orders o ON u.id = o.user_id;

-- 2nd highest order amount (simple version)
SELECT MAX(amount) FROM orders WHERE amount < (SELECT MAX(amount) FROM orders);
```

---

## Your honest positioning line (use it)
> "I'm **stronger on the operational database side** — RDS **Multi-AZ vs read replicas**, backups/PITR, connection pooling, and secret rotation, which I've done in production — and comfortable with core SQL: joins, group by, indexes, and reading an `EXPLAIN` plan. I haven't done heavy query optimization, but here's how I'd approach a slow query…" → then the playbook.

That answer is **honest, competent, and leads with your real strength.** It's a strong answer, not an apology.

---

## If you have 20 minutes, do exactly this
1. Re-read the **running example + JOIN + GROUP BY/HAVING** until you can write those 4 queries from memory.
2. Memorize the **index analogy**, the **WHERE-vs-HAVING** line, and **ACID**.
3. Lock in **Multi-AZ vs read replica** (your bridge from AWS).
4. Rehearse the **slow-query playbook** and the **honest positioning line** out loud.
That covers ~80% of what a DevOps/platform DB round asks.
