# Technical Code Audit: SQL Outer Join Filtering & Logic Traps

## 1. Task Metadata

- **Domain:** Relational Databases / PostgreSQL / Analytics Querying
- **Evaluation Focus:** SQL three-valued logic (`NULL` evaluation) and outer join preservation
- **Severity Classification:** Critical (silent data dropping and business metric distortion)

---

## 2. The User Prompt

> We have an e-commerce database with two tables:
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
> 1. Include customers even if they have placed zero orders (display average as 0.00).
> 2. Exclude orders where `order_status = 'cancelled'` from the average calculation.
> 3. Sort results from highest average order value to lowest.

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
WHERE c.signup_date >= DATE '2024-01-01'
  AND c.signup_date < DATE '2025-01-01'
GROUP BY c.id, c.name
ORDER BY average_order_value DESC;
```

### Candidate B (Defective)

```sql
SELECT
    c.id AS customer_id,
    c.name,
    ROUND(AVG(COALESCE(o.order_total, 0))::numeric, 2) AS average_order_value
FROM customers c
LEFT JOIN orders o
    ON c.id = o.customer_id
WHERE c.signup_date >= DATE '2024-01-01'
  AND c.signup_date < DATE '2025-01-01'
  AND o.order_status != 'cancelled'
GROUP BY c.id, c.name
ORDER BY average_order_value DESC;
```

## 4. Quality Audit & Forensic Evaluation

### Verdict

Candidate A is rated **Much Better** than Candidate B. Candidate B places an order filter in the `WHERE` clause, violating the requirement to include customers with zero orders.

### Constraint & Technical Audit

- **Date range filtering:** Both queries select customers who registered in 2024.
- **Aggregation and rounding:** Both queries round the average to two decimal places.
- **Zero-order customer preservation:** Candidate A preserves customers without qualifying orders and displays `0.00`. Candidate B excludes them.
- **Cancelled orders:** Candidate A filters cancelled orders in the join condition. Candidate B filters them in the `WHERE` clause.

### Forensic Evidence

In Candidate B, customers with no orders have `NULL` for `o.order_status`. The predicate `o.order_status != 'cancelled'` evaluates to `UNKNOWN` for those rows. The `WHERE` clause removes them, effectively turning the outer join into an inner join for this condition.

Candidate A places the predicate in the `ON` clause instead. This excludes cancelled orders from the aggregation while preserving customers who have no qualifying orders.

### Operational Impact

Candidate B omits customers with no qualifying orders, distorting reporting by excluding that customer cohort. Candidate A preserves those customers and returns the required zero average.
