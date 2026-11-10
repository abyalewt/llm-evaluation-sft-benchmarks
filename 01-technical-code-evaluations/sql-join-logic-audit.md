# 2. `01-technical-code-evaluations/sql-join-logic-audit.md`

````markdown
# Technical Code Audit: SQL Outer Join Filtering & Logic Traps

## 1. Task Metadata

- **Domain:** Relational Databases / PostgreSQL / Analytics Querying
- **Evaluation Focus:** SQL 3-Valued Logic (`NULL` Evaluation) & Outer Join Preservation
- **Severity Classification:** Critical (Silent Data Dropping & Business Metric Distortion)

---

## 2. The User Prompt

> "We have an e-commerce database with two tables:
>
> 1. `customers` (`id`, `name`, `signup_date`)
> 2. `orders` (`id`, `customer_id`, `order_total`, `order_status`)
>
> Write a PostgreSQL query to calculate the average order value (AOV) for every customer who registered in 2024. The result must display:
>
> - `customer_id`
> - `name`
> - `average_order_value` (rounded to 2 decimal places)
>
> Requirements:
>
> 1. Include customers even if they have placed ZERO orders (display average as 0.00).
> 2. Exclude orders where `order_status = 'cancelled'` from the average calculation.
> 3. Sort results from highest average order value to lowest."

---

## 3. Candidate Outputs

### Candidate A (Compliant)

```sql
SELECT
    c.id AS customer_id,
    c.name,
    COALESCE(ROUND(AVG(o.order_total)::numeric, 2), 0.00) AS average_order_value
FROM customers c
LEFT JOIN orders o
    ON c.id = o.customer_id
    AND o.order_status != 'cancelled'
WHERE c.signup_date >= '2024-01-01'
  AND c.signup_date <= '2024-12-31'
GROUP BY c.id, c.name
ORDER BY average_order_value DESC;
Candidate B (Defective)
SQL
SELECT
    c.id AS customer_id,
    c.name,
    ROUND(AVG(COALESCE(o.order_total, 0))::numeric, 2) AS average_order_value
FROM customers c
LEFT JOIN orders o
    ON c.id = o.customer_id
WHERE c.signup_date >= '2024-01-01'
  AND c.signup_date <= '2024-12-31'
  AND o.order_status != 'cancelled'
GROUP BY c.id, c.name
ORDER BY average_order_value DESC;
4. Quality Audit & Forensic Evaluation
Verdict
Response A is rated Much Better than Response B due to a fatal filtering placement error in Response B that violates Requirement 1.

Constraint & Technical Audit
Date Range Filtering: Both models correctly isolate signup_date within 2024.

Aggregation Rounding: Both use ROUND(...::numeric, 2).

Zero-Order Customer Preservation: Response A preserves zero-order customers. Response B drops them completely.

Forensic Evidence
The WHERE Clause Filter Flaw: In Response B, the condition o.order_status != 'cancelled' is placed in the WHERE clause. For customers who placed zero orders, o.order_status evaluates to NULL. In SQL three-valued logic:

SQL
NULL != 'cancelled' --> UNKNOWN (Falsy)
This condition silently discards every customer with zero orders, converting the LEFT JOIN into an unintended INNER JOIN.

Join Condition Isolation: Response A correctly handles this by placing the predicate in the join condition (ON c.id = o.customer_id AND o.order_status != 'cancelled'), preserving all left-table rows.

Operational Impact
Response B returns distorted analytics that omit an entire user cohort (unconverted users), leading to inaccurate business reporting. Response A satisfies all constraints and calculates the true mathematical average.
```
````
