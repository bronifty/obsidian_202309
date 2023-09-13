Unity Cat > Delta Lake > Data Lake
- Delta and Unity is Glue Catalog / Hive metadata atop the blob layer for atomic transactions and sql queries, audits and time travel, schema enforcement and evolution, support for delete/update/merge 
- allows cdc, scd and streaming upsert
- unified streaming and batch data proc
- transaction log allows time travel and audits
- spark checks transaction log when making queries and updates table with latest info; user table is sync with master record; it also makes sure that users can't update with errant info
- open source and integrates with all major analytics tools like Statch or Fivetran for ingestion, s3 for storage, spark redshift athena for query

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
- 