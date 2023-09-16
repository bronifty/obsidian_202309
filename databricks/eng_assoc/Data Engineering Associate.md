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

- custom schema
```sql 
CREATE SCHEMA custom
LOCATION 'dbfs:/Shared/schemas/custom.db';
DESCRIBE DATABASE EXTENDED custom;
```

- create managed & external tables in custom schema
```sql
USE custom;
CREATE TABLE managed_custom
(width INT, length INT, height INT);
INSERT INTO managed_custom
VALUES (3 INT, 2 INT, 1 INT);
-----------------------------------

CREATE TABLE external_custom
(width INT, length INT, height INT)
LOCATION 'dbfs:/mnt/demo/external_custom';
INSERT INTO external_custom
VALUES (3 INT, 2 INT, 1 INT);
```

```sql
DESCRIBE EXTENDED managed_custom
```
- dbfs:/Shared/schemas/custom.db/managed_custom

```sql
DESCRIBE EXTENDED external_custom
```
- dbfs:/mnt/demo/external_custom

```sql
DROP TABLE managed_custom;
DROP TABLE external_custom;
```

```sql
%fs ls 'dbfs:/Shared/schemas/custom.db/managed_custom'
```
- filenotfoundexception

```sql
%fs ls 'dbfs:/mnt/demo/external_custom'
```
- dbfs:/mnt/demo/external_custom/_delta_log/
- dbfs:/mnt/demo/external_custom/part-00000-bed3caba-0dc9-42a1-aa7b-4bba9c8a01f8.c000.snappy.parquet

- without Delta Tables if we drop a managed table, it is destructive the files are removed and it can't be restored; with Delta Tables we can restore a dropped table

### Delta Tables
- CTAS statements (CREATE TABLE _ AS SELECT)
	- auto infer schema no manual declaration; define with data
	- eg CREATE TABLE table_1 AS SELECT col_1, col_2 FROM table_2
	- additional options:
```sql
CREATE TABLE new_table
COMMENT "Contains PII"
PARTITIONED BY (city, birth_date)
LOCATION '/some/path'
AS SELECT id, name, email, birth_date, city FROM users
```
		
- Table constraints
	- NOT NULL constraints
	- CHECK constraints
```sql
ALTER TABLE table_name ADD CONSTRAINT constraint_name constraint_details
ALTER TABLE orders ADD CONSTRAINT valid_date CHECK (date > '2020-01-01');
```

#### Cloning Delta Lake tables
- deep clone - full copies of data + metadata from a source table to a target
	- full copy of structure and data
```sql
CREATE TABLE table_clone
DEEP CLONE source_table
```
		- can sync changes

- shallow clone - copy logs only (logs point to original source data)
	- use case would be to modify the structure aka schema of your table to test how it would perform without copying any data
```sql
CREATE TABLE table_clone
SHALLOW CLONE source_table
```

### CTAS
```sql

```


### Views
- Stored Views
	- persisted objects
```sql
CREATE VIEW view_name
AS query (select * from table where blah)
```
- Temp views (session-scoped); what creates a new session:
	- open new notebook
	- detaching and reattaching to cluster
	- installing python package & restarting python interpreter
	- restarting cluster
```sql
CREATE TEMP VIEW view_name
AS query
```
- Global temp (cluster scoped)
```sql
CREATE GLOBAL TEMP VIEW view_name
AS query;

SELECT *
FROM global_temp.view_name;
```


### Query Files Directly
```sql
SELECT * FROM <file_format>.'path/to/file'
```
- extract text files as raw strings (JSON, CSV, TXT)
```sql
SELECT * FROM csv.'/path/to/file'
```
- extract files as raw bytes (media/unstructured data)
```sql
SELECT * FROM binaryFile.'/path/to/file'
```

### Register Tables from Files
```sql
CREATE TABLE table_name
AS SELECT * FROM file_format.'/path/to/file';
```

```sql
CREATE TEMPORARY VIEW temp_view_phones_brands
AS SELECT DISTINCT brand 
FROM hive_metastore.default.smartphones;
SELECT * FROM temp_view_phones_brands;
SHOW TABLES IN global_temp; 
CREATE GLOBAL TEMPORARY VIEW latest_phones
AS SELECT * FROM smartphones
WHERE year > 2020
ORDER BY year DESC;
SELECT * FROM global_temp.latest_phones;
```


