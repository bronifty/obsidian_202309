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

