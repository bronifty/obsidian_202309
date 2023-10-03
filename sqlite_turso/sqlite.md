```shell
sudo apt update && sudo apt install sqlite3
sqlite3 local.db
```

```sql
CREATE TABLE medias (
	name TEXT,
	caption TEXT
);
ALTER TABLE medias ADD COLUMN id TEXT;
INSERT INTO medias (id, name, caption) VALUES ("1", "2", "3");
.header on
.tables
.schema medias
SELECT * FROM medias;
DELETE FROM medias;
.quit
```



