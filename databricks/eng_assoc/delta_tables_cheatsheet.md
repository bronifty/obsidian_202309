# DATABRICKS DATA ENG ASSOC

- [UDEMY COURSE](https://www.udemy.com/course/databricks-certified-data-engineer-associate/learn/lecture/34742270#overview)
- [COURSE REPO](https://github.com/bronifty/Databricks-Certified-Data-Engineer-Associate)
- [AZURE DATABRICKS DASHBOARD](https://adb-2695751147948847.7.azuredatabricks.net/browse/folders/2309189472105813?o=2695751147948847)
- [DATABRICKS COMMUNITY DASHBOARD](https://community.cloud.databricks.com/?o=6687968818076754#)

### CHEATSHEET

```python
dbutils.help();
dbutils.fs.help();
employees_file = dbutils.fs.ls('dbfs:/user/hive/warehouse/employees'); # get a handle to the file
print(employees_file);
display(employees_file); # print grid format
%fs ls 'dbfs:/user/hive/warehouse/employees'; # shortcut for the above

%python # the pragma aka pyspark 'magic command' to interpret this as python in a sql file
files = dbutils.fs.ls(f"{dataset_bookstore}/books-csv") # f-string interpolating dataset_bookstore directory
display(files) # print the grid of the files
```

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

DESCRIBE [EXTENDED] DETAIL smartphones; -- point to data & log file location (will show if there are multiple physical files making up a table)
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

CREATE DATABASE db_x -- DATABASE and SCHEMA are synonyms;
CREATE SCHEMA db_y
LOCATION 'dbfs:/custom/path/db_y.db'; -- specify the data/logs location

USE db_y; -- create tables under db_y schema in hive repo
CREATE TABLE table_1; -- will be managed table
CREATE TABLE table_2
(width INT, length INT, height INT)
LOCATION 'dbfs:/some/path_1/table_2'; -- will be external table
INSERT INTO table_2
VALUES (1,2,3);
DESCRIBE EXTENDED table_1; -- dbfs:/custom/path/db_y.db/table_1
DESCRIBE EXTENDED table_2; -- dbfs:/some/path_1/table_2

CREATE TABLE new_table
COMMENT "Contains PII"
PARTITIONED BY (city, birth_date)
LOCATION '/some/path'
AS SELECT id, name, email, birth_date, city FROM users; -- declarations available that aren't available to CTA (Create Table As query)

ALTER TABLE table_name ADD CONSTRAINT constraint_name constraint_details
ALTER TABLE orders ADD CONSTRAINT customer_name_not_null NOT NULL (customer_name); -- not null constraint
ALTER TABLE orders ADD CONSTRAINT valid_date CHECK (date > '2020-01-01'); -- check constraint

CREATE VIEW view_name
AS query (select * from table where blah); -- stored view
CREATE TEMP VIEW view_name
AS query; -- temp view (session scoped)
CREATE GLOBAL TEMP VIEW view_name
AS query; -- global view (cluster scoped)
SELECT *
FROM global_temp.view_name; -- referenced by its prefix in query

SHOW TABLES; -- show tables and views

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

CREATE TABLE books_unparsed AS
SELECT * FROM csv.`${dataset.bookstore}/books-csv`;
SELECT * FROM books_unparsed; -- a single column with rows 

CREATE TEMP VIEW books_tmp_vw
   (book_id STRING, title STRING, author STRING, category STRING, price DOUBLE)
USING CSV
OPTIONS (
  path = "${dataset.bookstore}/books-csv/export_*.csv",
  header = "true",
  delimiter = ";"
); -- specify the raw data source with USING
CREATE TABLE books AS 
  SELECT * FROM books_tmp_vw; -- CTAS from view
SELECT * FROM books -- grid format multiple columns

SELECT *,
    input_file_name() source_file -- input_file_name() is a built-in Spark function to get the filename, which is useful for debugging
FROM json.`${dataset.bookstore}/customers-json`; -- select from all the files in the directory (will auto-combine all files if they have same schema)
```


