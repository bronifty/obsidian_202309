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

- query files directly
- extract files as raw contents
- config options of external sources
- use CTAS to create Delta Lake Tables

- select from s
```sql
SELECT * FROM json.`/path/to/file.json` -- select from structured json parquet
SELECT * FROM text.`/path/to/file.json` -- extract text as raw string for json csv tsv txt used for raw strings and custom parsing
SELECT * FROM binaryFile.`/path/to/file.jpg` -- extract as raw bytes
```
- load extracted data into lakehouse
```sql
CREATE TABLE table_name AS SELECT * FROM file_format.`path/to/file`; -- auto-infer no support for manual declarations
CREATE TABLE table_name 
USING data_source_type
OPTIONS (key1 = val1, key2 = val2)
LOCATION = path -- USING gives us options to create external non-delta table
```

```sql
CREATE TABLE table_name
(col_name_1 col_type_1, col_name_2 col_type_2)
USING csv
OPTIONS (header = "true", delimiter = ";")
LOCATION = path; -- non-delta table from raw
CREATE TABLE table_name
(col_name_1 col_type_1)
USING JDBC
OPTIONS (url="jdbc:sqlite://hostname:port", dbtable="database.table", user="username", password="pwd"); -- non-delta table from raw

CREATE TEMPORARY VIEW temp_view_name (col_name_1 col_type_1)
USING data_source
OPTIONS (key1="val1", path="/path/to/file"); -- view from raw
CREATE TABLE table_name AS SELECT * FROM temp_view_name; -- Delta CTAS
```

```sql
SELECT *,
    input_file_name() source_file -- input_file_name() is a built-in Spark function to get the filename, which is useful for debugging
FROM json.`${dataset.bookstore}/customers-json`; -- select from all the files in the directory (will auto-combine all files if they have same schema)
```

```sql
CREATE TABLE orders AS
SELECT * FROM parquet.`${dataset.bookstore}/orders`
```

```sql
CREATE OR REPLACE TEMP VIEW parsed_customers AS

SELECT customer_id, from_json(profile, schema_of_json('{"first_name":"Thomas","last_name":"Lane","gender":"Male","address":{"street":"06 Boulevard Victor Hugo","city":"Paris","country":"France"}}')) AS profile_struct

FROM customers;

SELECT * FROM parsed_customers
```

- working with structs allows us to access dot properties
```sql
SELECT customer_id, profile_struct.first_name, profile_struct.address.country
FROM parsed_customers
```

Both `explode` and `flatten` functions are used to manipulate arrays in PySpark and they work differently.

The `explode` function takes an array of structs as input and returns a new row for each struct in the array. For example, if you have an array column with three elements, `explode` would create three rows each with only one of the elements from that array.

On the other hand, the `flatten` function takes an array of arrays as input and returns a new array that is the flattened version of the input array. The resulting array is a single-level array that contains all the elements from the input array.

Here's an example to illustrate the difference between the two functions:

Suppose you have a table called "students" with a column called "courses", where the "courses" column is an array of structs with the fields "name" and "grade". If you apply `explode` to the "courses" column, you would get a new row for each course, where each row has only one course and its corresponding name and grade. If you apply `flatten` to the "courses" column, you would get a new array that contains all the names and grades for all the courses, concatenated into a single-level array.

Here is the example illustrating the above explanation:

Input data:

[SQL](https://dbc-93d97377-d7c8.cloud.databricks.com/?o=5630293092916573#)

[](https://dbc-93d97377-d7c8.cloud.databricks.com/?o=5630293092916573# "Run Cell")[](https://dbc-93d97377-d7c8.cloud.databricks.com/?o=5630293092916573# "Replace active cell content")[](https://dbc-93d97377-d7c8.cloud.databricks.com/?o=5630293092916573# "Copy code")

+---------

+-------------------------+

| name    | 

courses                 |

+---------

+-------------------------+

| Alice   | [{Math, 90}, {Science, 

85}] |

| Bob     | [{Math, 80}, {Science, 

70}] |

+---------

+-------------------------+

`explode` example:

[SQL](https://dbc-93d97377-d7c8.cloud.databricks.com/?o=5630293092916573#)

[](https://dbc-93d97377-d7c8.cloud.databricks.com/?o=5630293092916573# "Run Cell")[](https://dbc-93d97377-d7c8.cloud.databricks.com/?o=5630293092916573# "Replace active cell content")[](https://dbc-93d97377-d7c8.cloud.databricks.com/?o=5630293092916573# "Copy code")

SELECT name, explode(courses) AS 

course

FROM students;

Output of `explode`:

[SQL](https://dbc-93d97377-d7c8.cloud.databricks.com/?o=5630293092916573#)

[](https://dbc-93d97377-d7c8.cloud.databricks.com/?o=5630293092916573# "Run Cell")[](https://dbc-93d97377-d7c8.cloud.databricks.com/?o=5630293092916573# "Replace active cell content")[](https://dbc-93d97377-d7c8.cloud.databricks.com/?o=5630293092916573# "Copy code")

+------+---------------+

| name | course.name   |

+------+---------------+

| Alice| Math          |

| Alice| Science       |

| Bob  | Math          |

| Bob  | Science       |

+------+---------------+

`flatten` example:

[SQL](https://dbc-93d97377-d7c8.cloud.databricks.com/?o=5630293092916573#)

[](https://dbc-93d97377-d7c8.cloud.databricks.com/?o=5630293092916573# "Run Cell")[](https://dbc-93d97377-d7c8.cloud.databricks.com/?o=5630293092916573# "Replace active cell content")[](https://dbc-93d97377-d7c8.cloud.databricks.com/?o=5630293092916573# "Copy code")

SELECT name, flatten(array(course.

name for course in courses)) AS 

course_names

FROM students;

Output of `flatten`:

text

+------+------------------+  
| name | course_names     |  
+------+------------------+  
| Alice| [Math, Science]  |  
| Bob  | [Math, Science]  |  
+------+------------------+


### Structured Streaming
- read from input table write to output table
- guarantees:
	- fault tolerance via checkpointing & WAL (write-ahead-logs)
	- exactly once idempotent sinks
```python
streamDF = spark.readStream.table("input_table") # read from input table 
streamDF.writeStream.trigger(processingTime="2 minutes").outputMode("append").option("checkpointLocation", "/path").table("output_table") # write to output table
```
- log is written to checkpoint folder ssc.checkpoint("hdfs:///checkpoint") ahead of writing to the delta table; it looks something like so
```txt
20/06/14 14:16:35 INFO SparkContext: Running Spark version 2.4.5
20/06/14 14:16:36 INFO SparkContext: Submitted application: Example Spark App
20/06/14 14:16:36 INFO SecurityManager: Changing view acls to: user
20/06/14 14:16:36 INFO SecurityManager: Changing modify acls to: user
20/06/14 14:16:36 INFO SecurityManager: Changing view acls groups to: 
20/06/14 14:16:36 INFO SecurityManager: Changing modify acls groups to: 
20/06/14 14:16:36 INFO SecurityManager: SecurityManager: authentication disabled; ui acls disabled; users  with view permissions: Set(user); groups with view permissions: 
20/06/14 14:16:36 INFO Utils: Successfully started service 'driverClient' on port 57351.
20/06/14 14:16:36 INFO SparkEnv: Registering MapOutputTracker
20/06/14 14:16:36 INFO SparkEnv: Registering BlockManagerMaster
20/06/14 14:16:36 INFO BlockManagerMasterEndpoint: Using org.apache.spark.storage.DefaultTopologyMapper for getting topology information
20/06/14 14:16:36 INFO BlockManagerMasterEndpoint: BlockManagerMasterEndpoint up
20/06/14 14:16:36 INFO DiskBlockManager: Created local directory at /private/var/folders/qj/wf3zhr1j10l0z64gj0tyldtw0000gn/T/blockmgr-914a8c2d-8b2d-48ba-915e-9146e5c42580
20/06/14 14:16:36 INFO MemoryStore: MemoryStore started with capacity 1874.1 MB
20/06/14 14:16:36 INFO SparkEnv: Registering OutputCommitCoordinator
20/06/14 14:16:37 INFO Utils: Successfully started service 'SparkUI' on port 4040.
20/06/14 14:16:37 INFO SparkUI: Bound SparkUI to 0.0.0.0, and started at http://ip-172-31-39-242.us-west-2.compute.internal:4040
```

- so there is no data in WAL?
  
In Apache Spark, the Write Ahead Log (WAL) is used to guarantee fault tolerance and exactly-once processing of data. The WAL is maintained by Spark's streaming operations such as updateStateByKey and window operations, rather than by Delta Lake.

Delta Lake uses its own transaction log to maintain the metadata for the Delta tables. The transaction log records all the changes made to a Delta table in the order in which they occur. These logs are stored in the WAL folder in the Delta table.

In the event of a failure, both the Delta Lake transaction log and the WAL for the Spark stream's operations can be used to recover the lost data and bring the table back to its previous state. The WAL ensures that Spark Streaming data is fault-tolerant in case of failures, while the Delta Lake transaction log ensures the consistency of the table data.

- in pyspark dataframe api we can use the spark.table() to load data from a streaming temp view back to a DataFrame
- spark will always load streaming views as streaming data frames and static views as static data frames 
	- incremental processing must be defined from the beginning with read logic to support incremental writing 

### Incremental Data Ingestion
- loading new data files since last ingestion
- reduces redundant processing
- 2 mechanisms:
	- COPY INTO
	- Auto loader
- COPY INTO SQL command
	- idempotently and incrementally load new data files
		- files that have already been loaded are skipped

```sql
COPY INTO my_table
FROM '/path/to/files'
FILEFORMAT = CSV
FORMAT_OPTIONS (
'delimiter' = '|',
'header' = 'true'
)
COPY_OPTIONS (
'mergeSchema' = 'true'
);
```

- Auto Loader structured streaming can process billions of files and support near real time ingestion of millions of files per hour; supports checkpointing with exactly once guarantees and fault tolerance
- autoloader has syntax that uses cloudFiles
```python

spark.readStream
	.format("cloudFiles")
	.option("cloudFiles.format", <source_format>)
	.option("cloudFiles.schemaLocation", <schema_directory>) # schema location will register the schema to preload it & thereby prevent the inference cost on startup of the writeStream's mergeSchema option set to true
	.load('/path/to/files') # autoloader will detect new files as they arrive and queue them for ingestion
.writeStream
	.option("checkpointLocation", <checkpoint_directory>)
	.option("mergeSchema", "true") # auto infer schema evolution and merge updates
	.table(<table_name>)

```
- COPY INTO is less efficient and is better for an order of thousands of files; Auto Loader is better at scale and for an order of millions of files (it can set up series of batches); Auto Loader is default for cloud object storage

