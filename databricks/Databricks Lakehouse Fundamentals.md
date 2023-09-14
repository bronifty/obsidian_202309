Unity Cat > Delta Lake > Data Lake
- Delta and Unity is Glue Catalog / Hive metadata atop the blob layer for atomic transactions and sql queries, audits and time travel, schema enforcement and evolution, support for delete/update/merge 
- allows cdc, scd and streaming upsert
- unified streaming and batch data proc
- transaction log allows time travel and audits
- spark checks transaction log when making queries and updates table with latest info; user table is sync with master record; it also makes sure that users can't update with errant info
- open source and integrates with all major analytics tools like Statch or Fivetran for ingestion, s3 for storage, spark redshift athena for query
![](./media/lakehouse_fundamentals/databricks_1.png)
- dark data is not catalogued
- cardinality & partitioning
- access control (rbac/abac) & audits are used for regulatory wrt PII (data governance)
- 
- data lineage and impact analysis (increase general context of data reduces tribal knowledge)
- 
- delta sharing 
	- enables zero-copy sharing capabilities via native integration with tableau, power bi, pandas and java 
	- provides centralized admin & governance as data is governed, tracked and audited from a single location allowing usage to be monitored at the table, partition, and version level. 
	- provide a central marketplace for distribution of data products to anywhere 
	- privacy safe data clean rooms

- control v data planes security architecture 
	- control plane is managed backend svcs provided by databricks (web app, configs, notebooks repos DBSQL, cluster mgr) - databricks account
		- data encrypted in transit
		- used by customer to access data eg from notebook or tableau
	- data plane is clusters customer account
		- data encrypted at rest
	- support/eng tickets allow databricks staff access to customer data only tied to workspace for ltd period in which ticket is open
	- data plane is run with non-privileged containers
	- serverless is managed in a databricks account with customer workspace separation

	- iam
	- Table ACLs for SQL based authz - SQL statements provide access via views
	- IAM profiles - clusters assume role
	- access key 
	- secrets
	- encryption isolation and auditing
- serverless sql 

### Data Mgt
- metastore, catalog, schema, table, view, and function
- unity catalog - catalogs, schemas, tables, views, storage creds, external locations
- ![](databricks_unity_catalog.png)
- ![metastore](databricks_metastore.png)
- metastore is a metadata database for auditing and governing including ACLs management of the workspaces, each which has its own Hive metastore
	- a logical construct for organizing data and its associated metadata (control plane => cloud storage)
	- a reference for a collection of metadata and a link to cloud storage container 
- catalog is topmost container for data objects in unity catalog (there can be many); it is the first level in 3-level namespace used to reference objects in unity cat
	- traditional sql 2 level namespace has for instance, a query
	```sql
	SELECT * FROM schema.table
```
	- whereas unity cat has a 3 level namespace
	```sql
	SELECT * FROM catalog.schema.table
```
- schema is container for data assets (part 2 of 3 level namespace)
- table - managed and external; both have metadata managed by the metastore in the control plane; managed are stored in the metastore's managed storage location; external tables are linked from an external location
- view - query result
- function - udfs encapsulate custom functionality to be used in queries
- storage cred - admin keys for buckets
- external location - external file containers
- share & recipient - delta sharing concepts for read-only logical collections of tables


![](databricks_10.png)
![](databricks_11.png)
