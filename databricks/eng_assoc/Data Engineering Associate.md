- Delta Lake is a cluster and storage of files and their logs
- log is source of truth
- writes create a copy of the parquet and a copy of the log with the new changes in each. reads always check the log first and get latest data. it's atomic so if a part of the write fails, it will not be recorded in the log and the new file will be discarded. 
- acid trans, scalable md, audit trail; parquet + json
- time travel, compacting small files and indexing, vacuum

- audit
```sql
SELECT * FROM employees;

DESCRIBE DETAIL employees;

DESCRIBE HISTORY employees;

%fs ls 'dbfs:/user/hive/warehouse/employees/_delta_log'

%fs head 'dbfs:/user/hive/warehouse/employees/_delta_log/00000000000000000004.json'
```

- time travel
	- with timestamp
```sql
SELECT * FROM my_table TIMESTAMP AS OF "2023-09-13"
```
	- with version number
```sql
SELECT * FROM my_table VERSION AS OF 36

SELECT * FROM my_table@v36
```

- rollback versions
- restore table command
```sql
RESTORE TABLE my_table TO TIMESTAMP AS OF "2023-09-13"

RESTORE TABLE my_table TO VERSION AS OF 36
```

- compaction
```sql
OPTIMIZE my_table ZORDER BY column_name
```
- this will split the file by the column_name
	- for instance if ZORDER is on ID and there are 2 files one from IDs 1-50 and another from 51-100, then the query will know that ID 30 is in file 1

- vacuum 
	- clean up files older than the retention period (default 7 days)
```sql
VACUUM table_name
```

- restore example
```sql
DELETE FROM employes;
SELECT * FROM employees;
RESTORE TABLE employees TO VERSION AS OF 4;
SELECT * FROM employees;
DESCRIBE HISTORY employees;
```
	- the log adds an entry for the RESTORE

- let's optimize
```sql
DESCRIBE DETAIL employees;
```
	- shows numFiles 3
```sql
OPTIMIZE employees
ZORDER BY (id);
```
	- now it's 1 file

- let's vacuum to get rid of unused files 
```sql
SET spark.databricks.delta.retentionDurationCheck.enabled = false;
VACUUM employees RETAIN 0 HOURS;
```
	- only keep latest copy (retain 0 hours)

- we can check it with DESCRIBE DETAIL which will show us the db file location which we can then ls with fs and it will show us one db file in the hive warehouse and the log
```sql
DESCRIBE DETAIL employees;
%fs ls 'dbfs:/user/hive/warehouse/employees'
```

- now we cannot query an old table version
```sql
SELECT * FROM employees@v2; -- no entries
SELECT * FROM employees@v9; -- yes entries (latest version)
```

