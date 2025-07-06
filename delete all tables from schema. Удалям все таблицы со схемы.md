[[хранимые процедуры и функции. Functions and procedure]]

```sql
CREATE OR REPLACE PROCEDURE drop_all_tables_in_schema(p_schema_name VARCHAR)
LANGUAGE plpgsql
AS $$
DECLARE
    table_record RECORD;
    drop_stmt TEXT;
BEGIN
    -- Check if schema exists
    IF NOT EXISTS (
        SELECT 1 
        FROM information_schema.schemata 
        WHERE schema_name = p_schema_name
    ) THEN
        RAISE EXCEPTION 'Schema % does not exist', p_schema_name;
    END IF;

    -- Loop through all tables in the specified schema
    FOR table_record IN (
        SELECT table_name
        FROM information_schema.tables
        WHERE table_schema = p_schema_name
        AND table_type = 'BASE TABLE'
    ) LOOP
        -- Construct and execute DROP TABLE statement
        drop_stmt := format('DROP TABLE IF EXISTS %I.%I CASCADE', p_schema_name, table_record.table_name);
        EXECUTE drop_stmt;
    END LOOP;

    RAISE NOTICE 'All tables in schema % have been dropped', p_schema_name;
EXCEPTION
    WHEN OTHERS THEN
        RAISE EXCEPTION 'Error dropping tables in schema %: %', p_schema_name, SQLERRM;
END;
$$;
```