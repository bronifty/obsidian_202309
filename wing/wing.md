### Multifile (module) and Javascript (extern) support
- store.w
```wing
bring cloud;

class Store{
  pub data: cloud.Bucket;
  init() {
    this.data = new cloud.Bucket();
  }
  extern "./url_utils.js" pub static isValidUrl(url: str): bool;
}

// log("Store.isValidUrl('http://www.google.com') ${Store.isValidUrl("http://www.google.com")}");
// log("!Store.isValidUrl('X?Y') ${!Store.isValidUrl("X?Y")}");

```
- main.w
```wing
bring "./store.w" as store;

let s1 = new store.Store() as "store1"; 
s1.data.addFile("url_utils.js", "./url_utils.js", "utf-8");
log("File added s1");
let s2 = new store.Store() as "store2"; 
s2.data.addFile("hello.txt", "./hello.txt", "utf-8");
log("File added to s2");

log("Store.isValidUrl('http://www.google.com') ${store.Store.isValidUrl("http://www.google.com")}");
log("!Store.isValidUrl('X?Y') ${!store.Store.isValidUrl("X?Y")}");

```
- the imported wing file (module) is a namespace with properties such as the class whose constructor and methods you can call
- extern js files only work with static methods of classes
	- the syntax is namespace.class.staticmethod()

