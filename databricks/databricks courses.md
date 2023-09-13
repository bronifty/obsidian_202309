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
- data lineage and impact analysis (increase general context of data reduces tribal knowledge)
- delta sharing enables zero-copy sharing capabilities via native integration with tableau, power bi, pandas and java
- 

