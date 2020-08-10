# POSTGRESQL NOTES

## 1. Table, View, Object
### Get all tables
```sql
SELECT table_name
FROM information_schema.tables
WHERE table_schema = 'public'
```

### Create table
```sql
CREATE TABLE partitions (
    partition_id int not null primary key,
    partition_num int not null,
    partition_name varchar(64) not null,
    table_name varchar(64) not null,
    num_of_rows int not null,
    create_time timestamp default current_timestamp,
    update_time timestamp default current_timestamp,
    unique(partition_name)
)
``` 

### Copy table structure
```sql
CREATE TABLE new_table_name (like old_table_name including all)

create table new_table_name (
    like old
    including defaults
    including onstraints
    including indexes
)
```
