# Marts Patterns

## Fact Tables (fct_)
Contain measurable events or transactions with foreign keys to dimensions.

## Dimension Tables (dim_)
Contain descriptive attributes for analysis.

## Best Practices
- Use meaningful business names
- Implement surrogate keys
- Add comprehensive tests
- Document business logic
- Use appropriate grain
- Implement slowly changing dimensions when needed

## Fact Table Example
```sql
-- fct_orders.sql
select
    order_id,
    customer_id,
    order_date,
    order_total,
    quantity
from {{ ref('stg_orders') }}
```

## Dimension Table Example
```sql
-- dim_customers.sql
select
    customer_id,
    customer_name,
    customer_email,
    customer_segment
from {{ ref('stg_customers') }}
```
