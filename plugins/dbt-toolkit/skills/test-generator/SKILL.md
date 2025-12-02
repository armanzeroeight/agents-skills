---
name: test-generator
description: Generates dbt tests including schema tests, data quality tests, and freshness checks. Use when adding tests to dbt models or implementing data quality validation.
---

# dbt Test Generator

## Quick Start

Generate comprehensive tests for dbt models including schema tests, data quality tests, and freshness checks.

## Instructions

### Step 1: Add schema tests

```yaml
# models/schema.yml
version: 2

models:
  - name: stg_orders
    description: Staging orders table
    columns:
      - name: order_id
        description: Unique order identifier
        tests:
          - unique
          - not_null
      
      - name: customer_id
        description: Customer reference
        tests:
          - not_null
          - relationships:
              to: ref('stg_customers')
              field: customer_id
      
      - name: order_status
        description: Order status
        tests:
          - accepted_values:
              values: ['pending', 'processing', 'shipped', 'delivered', 'cancelled']
      
      - name: order_total
        description: Total order amount
        tests:
          - not_null
          - dbt_utils.accepted_range:
              min_value: 0
```

### Step 2: Create custom data tests

```sql
-- tests/assert_positive_revenue.sql
select
    order_id,
    order_total
from {{ ref('fct_orders') }}
where order_total < 0
```

### Step 3: Configure freshness checks

```yaml
# models/sources.yml
version: 2

sources:
  - name: raw
    database: analytics
    schema: raw_data
    tables:
      - name: orders
        description: Raw orders data
        freshness:
          warn_after: {count: 12, period: hour}
          error_after: {count: 24, period: hour}
        loaded_at_field: created_at
```

### Step 4: Run tests

```bash
# Run all tests
dbt test

# Run tests for specific model
dbt test --select stg_orders

# Run specific test type
dbt test --select test_type:unique
dbt test --select test_type:not_null
```

## Test Types

**Schema tests:**
- unique
- not_null
- accepted_values
- relationships

**dbt_utils tests:**
- accepted_range
- at_least_one
- cardinality_equality
- equal_rowcount
- expression_is_true
- recency
- unique_combination_of_columns

**Custom tests:**
- SQL queries that return failing rows
- Placed in tests/ directory

## Best Practices

1. Test all primary keys (unique, not_null)
2. Test foreign key relationships
3. Test accepted values for categorical columns
4. Add range tests for numeric columns
5. Configure freshness for critical sources
6. Create custom tests for business logic
7. Run tests in CI/CD pipeline
8. Document test expectations
