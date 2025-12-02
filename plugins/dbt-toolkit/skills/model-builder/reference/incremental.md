# Incremental Models

## When to Use
- Large datasets (millions+ rows)
- Append-only data (events, logs)
- Expensive transformations
- Frequent updates needed

## Basic Incremental
```sql
{{
    config(
        materialized='incremental',
        unique_key='id'
    )
}}

select * from {{ source('raw', 'events') }}

{% if is_incremental() %}
where created_at > (select max(created_at) from {{ this }})
{% endif %}
```

## Merge Strategy
```sql
{{
    config(
        materialized='incremental',
        unique_key='id',
        merge_update_columns=['status', 'updated_at']
    )
}}
```

## Delete+Insert Strategy
```sql
{{
    config(
        materialized='incremental',
        unique_key='id',
        incremental_strategy='delete+insert'
    )
}}
```

## Best Practices
- Always specify unique_key
- Test incremental logic thoroughly
- Handle late-arriving data
- Monitor for data quality issues
- Use appropriate incremental strategy
