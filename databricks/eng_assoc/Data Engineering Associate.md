- [github repo](https://github.com/bronifty/databricks_data_eng_assoc)
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

### Database and Schema same thing
- database = schema in hive metastore
```sql
CREATE DATABASE db_name;
CREATE SCHEMA db_name;
```
- hive metastore is a repository of metadata for databases tables etc

```sql
CREATE SCHEMA db_y
LOCATION 'dbfs:/custom/path/db_y.db'

USE db_y;
CREATE TABLE table_1;
CREATE TABLE table_2;
```

- managed tables created under default directory
	- drop table deletes underlying file system
```sql
CREATE TABLE table_name;
```
- external tables created outside default directory with LOCATION keyword
	- drop table does not delete underlying file system
```sql
CREATE TABLE table_name LOCATION 'path';
```

```sql
CREATE TABLE table_3
LOCATION 'dbfs:/some/path_1/table_3';
```

- table file location can be managed or external for a whole db or just a single table 
	- default schema could have external table 3 
	- db_x could have all its tables external
- hive manages the table lifecycle
	- the concept of 'schema' aka 'database' only exists in the Hive metadata database; it will organize all 'tables' in the filesystem with pointers


### 1.4 Database and Table
- new tables are created in notebooks
```sql
CREATE TABLE managed_default
(width INT, length INT, height INT);
INSERT INTO managed_default
VALUES (4, 5, 6);
DESCRIBE EXTENDED managed_default
```
- DESCRIBE EXTENDED shows default file location (not external)
	- dbfs:/user/hive/warehouse/managed_default
- now we create external table with location keyword specifying the file path
```sql
CREATE TABLE external_default
(width INT, length INT, height INT)
LOCATION 'dbfs:/mnt/demo/external_default';
INSERT INTO external_default
VALUES (4, 5, 6);
DESCRIBE EXTENDED external_default;
```



