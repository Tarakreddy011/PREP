# 🧠 SQL Interview Questions & Answers

> A practical, topic-wise SQL revision guide built from the **60 Advanced SQL Queries Asked in FAANG Interviews** reference shared in this folder.
>
> **Goal:** Learn the SQL concept first → understand the pattern → practice the related interview questions.

---

## 📚 Table of Contents

- [How to Use This README](#-how-to-use-this-readme)
- [1. Window Functions](#1--window-functions-q1q10)
- [2. Joins & Set Operations](#2--joins--set-operations-q11q18)
- [3. Subqueries & CTEs](#3--subqueries--ctes-q19q26)
- [4. Aggregation & Grouping](#4--aggregation--grouping-q27q34)
- [5. Ranking & Top-N Problems](#5--ranking--top-n-problems-q35q42)
- [6. String & Date Manipulation](#6--string--date-manipulation-q43q48)
- [7. Self-Joins & Hierarchical Data](#7--self-joins--hierarchical-data-q49q54)
- [🧩 Quick Pattern Cheat Sheet](#-quick-pattern-cheat-sheet)

---

## 🚀 How to Use This README

For every topic:

1. **Understand the concept**
2. **Look at the SQL pattern**
3. **Try the question yourself**
4. **Compare your query with the answer**
5. **Modify the query for a different condition**

The queries below use **PostgreSQL-style SQL** where database-specific syntax is required.

---

# 1. 🪟 Window Functions (Q1–Q10)

## What are Window Functions?

A window function performs a calculation across related rows **without collapsing those rows into one row**.

Common window functions:

| Function | Use |
|---|---|
| `RANK()` | Ranking with gaps after ties |
| `DENSE_RANK()` | Ranking without gaps |
| `ROW_NUMBER()` | Unique sequential number |
| `LAG()` | Access previous row |
| `LEAD()` | Access next row |
| `SUM() OVER()` | Running/cumulative calculations |
| `AVG() OVER()` | Moving/window average |
| `CUME_DIST()` | Cumulative distribution |
| `PERCENT_RANK()` | Relative rank |

### General pattern

```sql
function() OVER (
    PARTITION BY column
    ORDER BY column
)
```

---

## Q1. Find the Second Highest Salary

### 💡 Idea
Rank salaries from highest to lowest and select rank `2`.

### ✅ Answer

```sql
SELECT salary
FROM (
    SELECT salary,
           DENSE_RANK() OVER (ORDER BY salary DESC) AS rnk
    FROM employees
) t
WHERE rnk = 2;
```

**Why `DENSE_RANK()`?**  
It handles duplicate salaries correctly.

---

## Q2. Find the Nth Highest Salary

### 💡 Idea
Same pattern as Q1, but make the rank dynamic.

### ✅ Answer

```sql
SELECT salary
FROM (
    SELECT salary,
           DENSE_RANK() OVER (ORDER BY salary DESC) AS rnk
    FROM employees
) t
WHERE rnk = :N;
```

` :N ` represents the required rank.

---

## Q3. Running Total of Sales by Date

### 💡 Idea
Use `SUM()` as a window function and include all previous rows.

### ✅ Answer

```sql
SELECT order_date,
       amount,
       SUM(amount) OVER (
           ORDER BY order_date
           ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
       ) AS running_total
FROM orders
ORDER BY order_date;
```

---

## Q4. Moving Average of the Last 3 Orders per Customer

### 💡 Idea
Partition by customer and use the current row plus the previous two rows.

### ✅ Answer

```sql
SELECT customer_id,
       order_date,
       amount,
       AVG(amount) OVER (
           PARTITION BY customer_id
           ORDER BY order_date
           ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
       ) AS moving_avg
FROM orders;
```

---

## Q5. Rank Employees by Salary Within Department

### 💡 Idea
Restart the ranking for every department.

### ✅ Answer

```sql
SELECT name,
       dept_id,
       salary,
       RANK() OVER (
           PARTITION BY dept_id
           ORDER BY salary DESC
       ) AS dept_rank
FROM employees;
```

---

## Q6. Employees Earning Above Their Department Average

### 💡 Idea
Calculate the department average for every employee, then compare the employee salary with it.

### ✅ Answer

```sql
SELECT name,
       dept_id,
       salary
FROM (
    SELECT e.*,
           AVG(salary) OVER (PARTITION BY dept_id) AS dept_avg
    FROM employees e
) t
WHERE salary > dept_avg;
```

---

## Q7. Percentage of Total Sales by Product

### 💡 Idea
Calculate product sales and divide by total sales.

### ✅ Answer

```sql
SELECT product_id,
       SUM(amount) AS product_sales,
       SUM(amount) * 100.0 /
       SUM(SUM(amount)) OVER () AS pct_of_total
FROM orders
GROUP BY product_id;
```

---

## Q8. Cumulative Distribution of Test Scores

### 💡 Idea
`CUME_DIST()` tells what fraction of scores are less than or equal to the current score.

### ✅ Answer

```sql
SELECT student_id,
       score,
       CUME_DIST() OVER (ORDER BY score) AS cumulative_distribution
FROM scores;
```

---

## Q9. Days Between Consecutive Logins per User

### 💡 Idea
Use `LAG()` to get the previous login date.

### ✅ Answer

```sql
SELECT user_id,
       login_date,
       login_date -
       LAG(login_date) OVER (
           PARTITION BY user_id
           ORDER BY login_date
       ) AS days_since_last
FROM logins;
```

---

## Q10. Top 3 Salaries in Each Department

### 💡 Idea
Rank salaries separately inside each department.

### ✅ Answer

```sql
SELECT *
FROM (
    SELECT e.*,
           DENSE_RANK() OVER (
               PARTITION BY dept_id
               ORDER BY salary DESC
           ) AS rnk
    FROM employees e
) t
WHERE rnk <= 3;
```

---

# 2. 🔗 Joins & Set Operations (Q11–Q18)

## What is a JOIN?

A JOIN combines rows from multiple tables using a related column.

### Important JOIN types

- `INNER JOIN` → matching rows only
- `LEFT JOIN` → everything from the left table + matches
- `RIGHT JOIN` → everything from the right table + matches
- `FULL OUTER JOIN` → everything from both sides
- `CROSS JOIN` → every possible combination
- `SELF JOIN` → table joined with itself

### Set Operations

| Operator | Meaning |
|---|---|
| `UNION` | Combines results and removes duplicates |
| `UNION ALL` | Combines results and keeps duplicates |
| `INTERSECT` | Common rows |
| `EXCEPT` | Rows in first query but not second |

---

## Q11. Customers Who Never Placed an Order

### 💡 Pattern
`LEFT JOIN` + `IS NULL`.

### ✅ Answer

```sql
SELECT c.customer_id,
       c.name
FROM customers c
LEFT JOIN orders o
    ON c.customer_id = o.customer_id
WHERE o.order_id IS NULL;
```

---

## Q12. Employees Who Earn More Than Their Manager

### 💡 Pattern
This is a **self-join**.

### ✅ Answer

```sql
SELECT e.name AS employee,
       e.salary AS employee_salary,
       m.name AS manager,
       m.salary AS manager_salary
FROM employees e
JOIN employees m
    ON e.manager_id = m.emp_id
WHERE e.salary > m.salary;
```

---

## Q13. Departments With Zero Employees

### 💡 Pattern
Keep all departments using `LEFT JOIN`.

### ✅ Answer

```sql
SELECT d.dept_id,
       d.dept_name
FROM departments d
LEFT JOIN employees e
    ON d.dept_id = e.dept_id
WHERE e.emp_id IS NULL;
```

---

## Q14. Find Duplicate Emails

### 💡 Pattern
`GROUP BY` + `HAVING`.

### ✅ Answer

```sql
SELECT email,
       COUNT(*) AS count
FROM users
GROUP BY email
HAVING COUNT(*) > 1;
```

---

## Q15. Combine Two Result Sets Without Duplicates

### ✅ Answer

```sql
SELECT customer_id FROM orders_2024
UNION
SELECT customer_id FROM orders_2025;
```

---

## Q16. Customers Present in Both 2024 and 2025 Orders

### 💡 Pattern
Use `INTERSECT`.

### ✅ Answer

```sql
SELECT customer_id FROM orders_2024
INTERSECT
SELECT customer_id FROM orders_2025;
```

---

## Q17. Products Sold in Store A but Not Store B

### 💡 Pattern
Use `EXCEPT`.

### ✅ Answer

```sql
SELECT product_id
FROM store_a_sales
EXCEPT
SELECT product_id
FROM store_b_sales;
```

---

## Q18. Pairs of Employees With the Same Salary

### 💡 Pattern
Self-join and prevent duplicate/reversed pairs.

### ✅ Answer

```sql
SELECT a.name AS employee_1,
       b.name AS employee_2,
       a.salary
FROM employees a
JOIN employees b
    ON a.salary = b.salary
   AND a.emp_id < b.emp_id;
```

---

# 3. 🧱 Subqueries & CTEs (Q19–Q26)

## What is a Subquery?

A subquery is a query inside another query.

Example:

```sql
SELECT *
FROM employees
WHERE salary > (
    SELECT AVG(salary)
    FROM employees
);
```

## What is a CTE?

CTE = **Common Table Expression**.

It creates a temporary named result that makes complex queries easier to understand.

```sql
WITH temp AS (
    SELECT ...
)
SELECT *
FROM temp;
```

Recursive CTEs are useful for **hierarchies** such as employees → managers → subordinates.

---

## Q19. Second Highest Salary Using a Subquery

```sql
SELECT MAX(salary) AS second_highest
FROM employees
WHERE salary < (
    SELECT MAX(salary)
    FROM employees
);
```

---

## Q20. Employees Earning More Than Company Average

```sql
SELECT name,
       salary
FROM employees
WHERE salary > (
    SELECT AVG(salary)
    FROM employees
);
```

---

## Q21. Department With the Highest Average Salary

```sql
SELECT dept_id,
       AVG(salary) AS avg_salary
FROM employees
GROUP BY dept_id
ORDER BY avg_salary DESC
LIMIT 1;
```

---

## Q22. Customers Who Spent More Than the Average Customer

### 💡 Idea
First calculate spending per customer, then compare it with the average customer spending.

### ✅ Answer

```sql
WITH customer_totals AS (
    SELECT customer_id,
           SUM(amount) AS total
    FROM orders
    GROUP BY customer_id
)
SELECT customer_id,
       total
FROM customer_totals
WHERE total > (
    SELECT AVG(total)
    FROM customer_totals
);
```

---

## Q23. Recursive CTE — All Subordinates of a Manager

### 💡 Idea
Start with direct employees of the manager, then repeatedly find employees reporting to those employees.

### ✅ Answer

```sql
WITH RECURSIVE subs AS (
    SELECT emp_id,
           name,
           manager_id
    FROM employees
    WHERE manager_id = :mgr_id

    UNION ALL

    SELECT e.emp_id,
           e.name,
           e.manager_id
    FROM employees e
    JOIN subs s
      ON e.manager_id = s.emp_id
)
SELECT *
FROM subs;
```

---

## Q24. Users Who Logged In on Consecutive Days

### 💡 Idea
A common **gaps-and-islands** pattern.

### ✅ Answer

```sql
WITH ranked AS (
    SELECT user_id,
           login_date,
           login_date -
           ROW_NUMBER() OVER (
               PARTITION BY user_id
               ORDER BY login_date
           )::int AS grp
    FROM logins
),
streaks AS (
    SELECT user_id,
           grp,
           COUNT(*) AS streak_len
    FROM ranked
    GROUP BY user_id, grp
)
SELECT *
FROM streaks
WHERE streak_len > 1;
```

> If duplicate login dates are possible, deduplicate dates per user before applying this pattern.

---

## Q25. Top Spending Customer per City

### 💡 Idea
Calculate customer spending inside each city, rank it, and keep rank 1.

### ✅ Answer

```sql
WITH city_spend AS (
    SELECT c.city,
           c.customer_id,
           SUM(o.amount) AS total,
           RANK() OVER (
               PARTITION BY c.city
               ORDER BY SUM(o.amount) DESC
           ) AS rnk
    FROM customers c
    JOIN orders o
      ON c.customer_id = o.customer_id
    GROUP BY c.city, c.customer_id
)
SELECT *
FROM city_spend
WHERE rnk = 1;
```

---

## Q26. Products Never Ordered

### 💡 Pattern
`LEFT JOIN` + `IS NULL`.

### ✅ Answer

```sql
SELECT p.product_id
FROM products p
LEFT JOIN orders o
    ON p.product_id = o.product_id
WHERE o.product_id IS NULL;
```

---

# 4. 📊 Aggregation & Grouping (Q27–Q34)

## What is Aggregation?

Aggregation converts multiple rows into summarized information.

Important functions:

```text
COUNT()
SUM()
AVG()
MIN()
MAX()
```

### `WHERE` vs `HAVING`

- `WHERE` → filters rows **before** grouping
- `HAVING` → filters groups **after** grouping

---

## Q27. Total Sales per Month

```sql
SELECT DATE_TRUNC('month', order_date) AS month,
       SUM(amount) AS total_sales
FROM orders
GROUP BY DATE_TRUNC('month', order_date)
ORDER BY month;
```

---

## Q28. Average Order Value per Customer

```sql
SELECT customer_id,
       AVG(amount) AS avg_order_value
FROM orders
GROUP BY customer_id;
```

---

## Q29. Count Orders per Status

```sql
SELECT status,
       COUNT(*) AS order_count
FROM orders
GROUP BY status;
```

---

## Q30. Months Where Sales Were Below the Overall Monthly Average

### 💡 Idea
1. Calculate monthly sales.
2. Calculate the average of those monthly totals.
3. Keep months below that average.

### ✅ Answer

```sql
WITH monthly AS (
    SELECT DATE_TRUNC('month', order_date) AS month,
           SUM(amount) AS total
    FROM orders
    GROUP BY DATE_TRUNC('month', order_date)
)
SELECT month,
       total
FROM monthly
WHERE total < (
    SELECT AVG(total)
    FROM monthly
);
```

---

## Q31. Departments With More Than 5 Employees

```sql
SELECT dept_id,
       COUNT(*) AS employee_count
FROM employees
GROUP BY dept_id
HAVING COUNT(*) > 5;
```

---

## Q32. Pivot Sales by Quarter Using CASE WHEN

### 💡 Idea
Turn rows into quarter columns.

### ✅ Answer

```sql
SELECT
    product_id,
    SUM(CASE
        WHEN EXTRACT(QUARTER FROM order_date) = 1
        THEN amount ELSE 0 END) AS q1,
    SUM(CASE
        WHEN EXTRACT(QUARTER FROM order_date) = 2
        THEN amount ELSE 0 END) AS q2,
    SUM(CASE
        WHEN EXTRACT(QUARTER FROM order_date) = 3
        THEN amount ELSE 0 END) AS q3,
    SUM(CASE
        WHEN EXTRACT(QUARTER FROM order_date) = 4
        THEN amount ELSE 0 END) AS q4
FROM orders
GROUP BY product_id;
```

---

## Q33. Most Frequently Ordered Product

```sql
SELECT product_id,
       COUNT(*) AS order_count
FROM orders
GROUP BY product_id
ORDER BY order_count DESC
LIMIT 1;
```

> To return **all tied products**, use `RANK()` instead of `LIMIT 1`.

---

## Q34. Year-over-Year Growth

### 💡 Idea
Compare each year's total with the previous year using `LAG()`.

### ✅ Answer

```sql
WITH yearly AS (
    SELECT EXTRACT(YEAR FROM order_date) AS yr,
           SUM(amount) AS total
    FROM orders
    GROUP BY EXTRACT(YEAR FROM order_date)
)
SELECT yr,
       total,
       (total - LAG(total) OVER (ORDER BY yr))
       * 100.0 /
       NULLIF(LAG(total) OVER (ORDER BY yr), 0) AS yoy_pct
FROM yearly
ORDER BY yr;
```

---

# 5. 🏆 Ranking & Top-N Problems (Q35–Q42)

## Why Ranking Questions Matter

Top-N questions are extremely common in SQL interviews.

The main choice is:

| Function | Ties | Example |
|---|---|---|
| `ROW_NUMBER()` | No same rank | 1, 2, 3, 4 |
| `RANK()` | Same rank + gaps | 1, 2, 2, 4 |
| `DENSE_RANK()` | Same rank, no gaps | 1, 2, 2, 3 |

---

## Q35. Top 3 Highest-Paid Employees per Department — Ties Handled

```sql
SELECT *
FROM (
    SELECT e.*,
           DENSE_RANK() OVER (
               PARTITION BY dept_id
               ORDER BY salary DESC
           ) AS rnk
    FROM employees e
) t
WHERE rnk <= 3;
```

---

## Q36. Nth Highest Value — Generic Pattern

```sql
SELECT amount
FROM (
    SELECT DISTINCT amount,
           DENSE_RANK() OVER (
               ORDER BY amount DESC
           ) AS rnk
    FROM orders
) t
WHERE rnk = :N;
```

---

## Q37. Median Salary

### 💡 Idea
For an odd number of rows, the middle value is the median.  
For an even number, average the two middle values.

### ✅ PostgreSQL Answer

```sql
SELECT PERCENTILE_CONT(0.5)
       WITHIN GROUP (ORDER BY salary) AS median_salary
FROM employees;
```

### Interview Pattern

If the interviewer wants a window-function approach, identify the two middle rows and average them.

---

## Q38. Find Duplicate Transactions

Assume duplicates mean the same:

- `customer_id`
- `amount`
- `order_date`

### ✅ Answer

```sql
SELECT customer_id,
       amount,
       order_date,
       COUNT(*) AS duplicate_count
FROM orders
GROUP BY customer_id, amount, order_date
HAVING COUNT(*) > 1;
```

---

## Q39. Rank Products by Total Revenue

```sql
SELECT product_id,
       SUM(amount) AS revenue,
       RANK() OVER (
           ORDER BY SUM(amount) DESC
       ) AS revenue_rank
FROM orders
GROUP BY product_id;
```

---

## Q40. Employees Whose Salary Is a Local Maximum

A local maximum means salary is greater than the previous and next employee when ordered by hire date.

### 💡 Pattern
Use `LAG()` and `LEAD()`.

### ✅ Answer

```sql
WITH ordered AS (
    SELECT name,
           salary,
           hire_date,
           LAG(salary) OVER (ORDER BY hire_date) AS prev_sal,
           LEAD(salary) OVER (ORDER BY hire_date) AS next_sal
    FROM employees
)
SELECT name,
       salary,
       hire_date
FROM ordered
WHERE salary > prev_sal
  AND salary > next_sal;
```

---

## Q41. Longest Streak of Consecutive Active Days per User

### 💡 Pattern
This is the classic **gaps-and-islands** problem.

### ✅ Answer

```sql
WITH ranked AS (
    SELECT user_id,
           login_date,
           login_date -
           ROW_NUMBER() OVER (
               PARTITION BY user_id
               ORDER BY login_date
           )::int AS island
    FROM (
        SELECT DISTINCT user_id, login_date
        FROM logins
    ) x
),
streaks AS (
    SELECT user_id,
           island,
           COUNT(*) AS streak
    FROM ranked
    GROUP BY user_id, island
),
ranked_streaks AS (
    SELECT *,
           RANK() OVER (
               PARTITION BY user_id
               ORDER BY streak DESC
           ) AS rnk
    FROM streaks
)
SELECT user_id,
       streak AS longest_streak
FROM ranked_streaks
WHERE rnk = 1;
```

---

## Q42. Percentile Rank of Each Score

```sql
SELECT student_id,
       score,
       PERCENT_RANK() OVER (
           ORDER BY score
       ) AS pct_rank
FROM scores;
```

---

# 6. 🔤 String & Date Manipulation (Q43–Q48)

## Why String & Date Functions Matter

Real-world SQL frequently requires cleaning and transforming data.

### Common string functions

```text
LOWER()
UPPER()
TRIM()
SUBSTRING()
POSITION()
CONCAT()
COALESCE()
```

### Common date functions

```text
EXTRACT()
DATE_TRUNC()
TO_CHAR()
AGE()
```

---

## Q43. Extract Domain from Email

### Example

```text
tarak@gmail.com
        ↓
gmail.com
```

### ✅ PostgreSQL Answer

```sql
SELECT email,
       SUBSTRING(email FROM POSITION('@' IN email) + 1) AS domain
FROM users;
```

---

## Q44. Format Date as YYYY-MM

```sql
SELECT TO_CHAR(order_date, 'YYYY-MM') AS year_month
FROM orders;
```

---

## Q45. Users Registered in the Last 30 Days

```sql
SELECT *
FROM users
WHERE registration_date >= CURRENT_DATE - INTERVAL '30 days';
```

---

## Q46. Calculate Age From Date of Birth

### PostgreSQL

```sql
SELECT name,
       DATE_PART('year', AGE(CURRENT_DATE, date_of_birth)) AS age
FROM users;
```

---

## Q47. Find Gaps in a Sequence of IDs

### 💡 Idea
Look for an ID where the next expected ID does not exist.

### ✅ Answer

```sql
SELECT o.id + 1 AS gap_start
FROM orders o
LEFT JOIN orders next_o
    ON next_o.id = o.id + 1
WHERE next_o.id IS NULL
  AND EXISTS (
      SELECT 1
      FROM orders later_o
      WHERE later_o.id > o.id + 1
  )
ORDER BY o.id;
```

> This identifies missing IDs between existing IDs. Whether the final missing ID should count depends on the expected ID range.

---

## Q48. Concatenate Names While Handling NULLs

### 💡 Idea
Use `CONCAT_WS()` because it skips NULL arguments.

### ✅ Answer

```sql
SELECT CONCAT_WS(' ', first_name, last_name) AS full_name
FROM users;
```

Alternative:

```sql
SELECT COALESCE(first_name, '') ||
       CASE
           WHEN first_name IS NOT NULL
            AND last_name IS NOT NULL THEN ' '
           ELSE ''
       END ||
       COALESCE(last_name, '') AS full_name
FROM users;
```

---

# 7. 🌳 Self-Joins & Hierarchical Data (Q49–Q54)

## What is a Self-Join?

A self-join joins a table to itself.

Useful for:

- Employee → Manager
- Parent → Child
- Friend relationships
- Comparing rows within the same table

### Recursive CTE

Use a recursive CTE when the hierarchy can have **multiple levels**.

Example:

```text
CEO
 ├── Manager A
 │    ├── Employee 1
 │    └── Employee 2
 └── Manager B
      └── Employee 3
```

---

## Q49. Employee With Manager Name

```sql
SELECT e.name AS employee,
       m.name AS manager
FROM employees e
LEFT JOIN employees m
    ON e.manager_id = m.emp_id;
```

---

## Q50. All Employees Under a Manager — Any Depth

### 💡 Pattern
Recursive CTE.

### ✅ Answer

```sql
WITH RECURSIVE org AS (
    SELECT emp_id,
           name,
           manager_id,
           1 AS org_level
    FROM employees
    WHERE emp_id = :mgr_id

    UNION ALL

    SELECT e.emp_id,
           e.name,
           e.manager_id,
           org.org_level + 1
    FROM employees e
    JOIN org
      ON e.manager_id = org.emp_id
)
SELECT *
FROM org;
```

---

## Q51. Find Mutual Friend Pairs

Assume a `friends` table contains:

```text
user_id
friend_id
```

A mutual pair appears in both directions:

```text
A → B
B → A
```

### ✅ Answer

```sql
SELECT
    LEAST(a.user_id, a.friend_id) AS user1,
    GREATEST(a.user_id, a.friend_id) AS user2
FROM friends a
JOIN friends b
    ON a.user_id = b.friend_id
   AND a.friend_id = b.user_id
WHERE a.user_id < a.friend_id;
```

The `<` condition prevents returning the same pair twice.

---

## Q52. Detect Consecutive Orders by the Same Customer

### 💡 Idea
Compare each order with the next order for the same customer.

### ✅ Answer

```sql
SELECT o1.customer_id,
       o1.order_id,
       o2.order_id AS next_order
FROM orders o1
JOIN orders o2
    ON o1.customer_id = o2.customer_id
   AND o2.order_date = o1.order_date + INTERVAL '1 day';
```

> If "consecutive" means consecutive **orders** rather than consecutive calendar days, use `LEAD()` ordered by `order_date` instead.

---

## Q53. Compare Each Row's Amount With the Previous Row

### 💡 Pattern
`LAG()`.

### ✅ Answer

```sql
SELECT order_id,
       order_date,
       amount,
       amount - LAG(amount) OVER (
           ORDER BY order_date
       ) AS change_from_prev
FROM orders;
```

---

## Q54. Find Overlapping Date Ranges / Bookings

### 💡 Pattern
Two bookings overlap when:

```text
A.start < B.end
AND
B.start < A.end
```

### ✅ Answer

```sql
SELECT a.booking_id AS booking_1,
       b.booking_id AS booking_2
FROM bookings a
JOIN bookings b
    ON a.room_id = b.room_id
   AND a.booking_id < b.booking_id
   AND a.start_date < b.end_date
   AND b.start_date < a.end_date;
```

This prevents comparing a booking with itself and avoids duplicate pairs.

---

# 🧩 Quick Pattern Cheat Sheet

## 🔥 Most Important Interview Patterns

| Problem | Pattern to Remember |
|---|---|
| 2nd highest salary | `DENSE_RANK()` |
| Nth highest | `DENSE_RANK()` |
| Top N per group | `PARTITION BY + DENSE_RANK()` |
| Previous row | `LAG()` |
| Next row | `LEAD()` |
| Running total | `SUM() OVER()` |
| Moving average | `AVG() OVER()` |
| Ranking | `RANK()` / `DENSE_RANK()` |
| Above average | Subquery / window `AVG()` |
| Never ordered | `LEFT JOIN + IS NULL` |
| Duplicate values | `GROUP BY + HAVING` |
| Common records | `INTERSECT` |
| A but not B | `EXCEPT` |
| Combine unique records | `UNION` |
| Monthly totals | `DATE_TRUNC()` |
| Year-over-year | `LAG()` |
| Consecutive days | Gaps & Islands |
| Longest streak | Gaps & Islands + ranking |
| Employee-manager | Self Join |
| Hierarchy | Recursive CTE |
| Missing IDs | Self Join / `LEAD()` |
| Overlapping dates | Range-overlap condition |
| Median | `PERCENTILE_CONT()` |
| Percentile | `PERCENT_RANK()` |
| Cumulative distribution | `CUME_DIST()` |

---

# 🧠 SQL Interview Mental Models

## 1️⃣ Need the previous row?

Think:

```sql
LAG()
```

## 2️⃣ Need the next row?

Think:

```sql
LEAD()
```

## 3️⃣ Need Top N?

Think:

```sql
DENSE_RANK() OVER (...)
```

## 4️⃣ Need to compare against an average?

Think:

```sql
AVG() OVER (...)
```

or

```sql
SELECT AVG(...)
```

## 5️⃣ Need records that don't exist in another table?

Think:

```sql
LEFT JOIN ... IS NULL
```

or:

```sql
EXCEPT
```

## 6️⃣ Need repeated/consecutive dates?

Think:

```text
Gaps & Islands
```

## 7️⃣ Need employee → manager → manager's manager?

Think:

```text
Recursive CTE
```

## 8️⃣ Need to compare rows inside the same table?

Think:

```text
Self Join
```

---

# 🏁 Final Revision Order

If you are preparing for an SQL interview, revise in this order:

```text
1. SELECT / WHERE / ORDER BY
        ↓
2. GROUP BY / HAVING
        ↓
3. JOINs
        ↓
4. Subqueries
        ↓
5. CTEs
        ↓
6. Window Functions
        ↓
7. RANK / DENSE_RANK / ROW_NUMBER
        ↓
8. LAG / LEAD
        ↓
9. Date & String Functions
        ↓
10. Recursive CTEs
        ↓
11. Gaps & Islands
        ↓
12. Advanced Interview Problems
```

---

## 📌 Note

The provided reference images contain questions **Q1–Q54** from the visible material. The source heading mentions **60 questions**, but Q55–Q60 were not present in the supplied images, so they are intentionally not invented here.

---

## ⭐ Quick Goal

Don't memorize 54 queries.

Memorize the **patterns**:

> **JOIN → GROUP BY → HAVING → SUBQUERY → CTE → WINDOW FUNCTION → LAG/LEAD → RANKING → GAPS & ISLANDS → RECURSION**

Once these patterns are clear, many SQL interview questions become variations of the same problem.

---

### 📖 Practice Rule

For every question:

**Read → Hide the answer → Write SQL yourself → Run it → Check edge cases → Compare**

That's how you turn SQL syntax into interview skill.
