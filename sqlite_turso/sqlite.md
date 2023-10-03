- install & config
```shell
sudo apt update && sudo apt install sqlite3
sqlite3 local.db
```
- cmd line
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
- libsqueal sdk client
```javascript
// db.mjs
import { createClient } from "@libsql/client";
const config = {
  url: "file:local.db",
};
const db = createClient(config);
export default db;
// const rs = await db.execute("SELECT * FROM medias");
// console.log(rs);
```
- cjs esm interop for @libsql/client
![cjs esm interop sqlite client](cjs_esm_interop_sqlite.png)
![](./medias/requireFromUrl_not_working.png)

