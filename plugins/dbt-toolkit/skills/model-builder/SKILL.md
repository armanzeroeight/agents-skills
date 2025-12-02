---
name: model-builder
description: Creates dbt models with proper layering (staging, marts), incremental strategies, and documentation. Use when creating dbt models, organizing data transformations, or implementing incremental models.
---

# dbt Model Builder

## Quick Start

Create well-structured dbt models following best practices for staging, intermediate, and mart layers.

## Instructions

### Step 1: Create staging models

Staging models clean and standardize raw data:

```sql
-- models/staging/stg_orders.sql
with source as (
    select * from {{ source('raw', 'orders') }}
),

renamed as (
    select
        order_id,
        customer_id,
        order_date,
        order_total,
        order_status,
        created_at,
        updated_at
    from source
)

select * from renamed
```

**Add schema file:**
```yaml
# models/staging/schema.yml
version: 2

models:
  - name: stg_orders
    description: Cleaned and standardized orders from raw data
    columns:
      - name: order_id
        description: Unique order identifier
        tests:
          - unique
          - not_null
      - name: customer_id
        description: Customer who placed the order
        tests:
          - not_null
```

### Step 2: Create mart models

Mart models contain business logic:

```sql
-- models/marts/fct_orders.sql
with orders as (
    select * from {{ ref('stg_orders') }}
),

customers as (
    select * from {{ ref('stg_customers') }}
),

final as (
    select
        orders.order_id,
        orders.customer_id,
        customers.customer_name,
        orders.order_date,
        orders.order_total,
        orders.order_status
    from orders
    left join customers
        on orders.customer_id = customers.customer_id
)

select * from final
```

### Step 3: Create incremental models

For large datasets, use incremental models:

```sql
-- models/marts/fct_events.sql
{{
    config(
        materialized='incremental',
        unique_key='event_id',
        on_schema_change='fail'
    )
}}

with events as (
    select * from {{ source('raw', 'events') }}
    
    {% if is_incremental() %}
    where event_timestamp > (select max(event_timestamp) from {{ this }})
    {% endif %}
)

select * from events
```

### Step 4: Add documentation

```yaml
# models/marts/schema.yml
version: 2

models:
  - name: fct_orders
    description: Order facts with customer information
    columns:
      - name: order_id
        description: Unique order identifier
        tests:
          - unique
          - not_null
      - name: order_total
        description: Total order amount
        tests:
          - not_null
          - dbt_utils.accepted_range:
              min_value: 0
```

## Model Layering

**Staging (stg_):**
- Clean and standardize raw data
- One-to-one with source tables
- Minimal transformations
- Column renaming and type casting

**Intermediate (int_):**
- Complex transformations
- Join multiple staging models
- Not exposed to end users

**Marts (fct_, dim_):**
- Business logic
- Fact and dimension tables
- Exposed to end users

## Best Practices

1. Follow naming conventions (stg_, int_, fct_, dim_)
2. Use CTEs for readability
3. Document all models and columns
4. Add tests to all models
5. Use refs for dependencies
6. Implement incremental models for large datasets
7. Configure materialization appropriately
8. Use sources for raw data

## Advanced

For detailed information, see:
- [Staging Patterns](reference/staging-patterns.md) - Staging model best practices
- [Marts Patterns](reference/marts-patterns.md) - Fact and dimension table patterns
- [Incremental](reference/incremental.md) - Incremental model strategies
