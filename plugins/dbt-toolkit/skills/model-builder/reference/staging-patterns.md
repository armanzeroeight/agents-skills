# Staging Patterns

## Purpose
Staging models clean and standardize raw data with minimal transformation.

## Best Practices
- One staging model per source table
- Rename columns to standard naming
- Cast data types appropriately
- Filter out deleted records
- Add basic data quality tests
- Document all columns

## Example Pattern
```sql
with source as (
    select * from {{ source('raw', 'table_name') }}
),

renamed as (
    select
        id as record_id,
        created_at::timestamp as created_timestamp,
        lower(email) as email_address
    from source
    where deleted_at is null
)

select * from renamed
```
