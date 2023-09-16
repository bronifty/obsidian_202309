```sql 
CREATE TABLE IF NOT EXISTS smartphones
(id INT, name STRING, brand STRING, year INT); 
INSERT INTO smartphones
VALUES (1, 'iPhone 14', 'Apple', 2022),
      (2, 'iPhone 13', 'Apple', 2021),
      (3, 'iPhone 6', 'Apple', 2014),
      (4, 'iPad Air', 'Apple', 2013),
      (5, 'Galaxy S22', 'Samsung', 2022),
      (6, 'Galaxy Z Fold', 'Samsung', 2022),
      (7, 'Galaxy S9', 'Samsung', 2016),
      (8, '12 Pro', 'Xiaomi', 2022),
      (9, 'Redmi 11T Pro', 'Xiaomi', 2022),
      (10, 'Redmi Note 11', 'Xiaomi', 2021); -- let's start with a table and some data

CREATE TABLE smartphone_clone
DEEP CLONE smartphones; -- copy log and data

CREATE TABLE smartphone_shallow_clone
SHALLOW CLONE smartphones; -- copy log which points to data file version at time of clone

DESCRIBE DETAIL smartphones; -- point to data & log file location (will show if there are multiple physical files making up a table)
DESCRIBE HISTORY smartphones; -- enumerate data file versions (maintained in log)

DELETE FROM smartphones; -- copy data file on write to new version with no data
RESTORE TABLE smartphones TO VERSION AS OF 2; -- point head in log to previous data file version
SELECT * FROM my_table@v36; -- select from table out of its versions described in history

DROP TABLE smartphones; -- destructive
SET spark.databricks.delta.retentionDurationCheck.enabled = false; -- don't do this but just for demo to clear the default retention of 1 week for vacuum
VACUUM employees RETAIN 0 HOURS; -- destructive for all but most recent data file

OPTIMIZE employees
ZORDER BY (id); --- compaction (combine multiple files; DESCRIBE DETAIL will show how many files comprise the table before and after compaction)

%fs ls 'dbfs:/user/hive/warehouse/employees/_delta_log'
%fs head 'dbfs:/user/hive/warehouse/employees/_delta_log/00000000000000000004.json'
```


```sql
CREATE DATABASE db_name;
CREATE SCHEMA db_name;
```
- hive metastore is a repository of metadata for databases tables etc

```sql
CREATE DATABASE db_x -- DATABASE and SCHEMA are synonyms;
CREATE SCHEMA db_y
LOCATION 'dbfs:/custom/path/db_y.db'; -- specify the data/logs location

USE db_y; -- create tables under db_y schema in hive repo
CREATE TABLE table_1; -- will be managed table
CREATE TABLE table_2
LOCATION 'dbfs:/some/path_1/table_2'; -- will be external table
```