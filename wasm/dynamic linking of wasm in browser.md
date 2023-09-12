```wat
(module
  (func $add (param i32 i32) (result i32)
    (local.get 0)
    (local.get 1)
    (i32.add)
  )
  (export "add" (func $add))
)
(module
  (import "module1" "add" (func $add (param i32 i32) (result i32)))
  (func $increment (param i32) (result i32)
    (local.get 0)
    (i32.const 1)
    (call $add)
  )
  (export "increment" (func $increment))
)

```
```sh
wat2wasm module1.wat -o module1.wasm
wat2wasm module2.wat -o module2.wasm

```

```html
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>WASM Dynamic Linking</title>
</head>
<body>

<script>
  async function loadAndLinkModules() {
    // Load module1
    const module1Response = await fetch('module1.wasm');
    const module1Binary = await module1Response.arrayBuffer();
    const module1 = await WebAssembly.instantiate(module1Binary);

    // Load module2 with module1 as an import object
    const module2Response = await fetch('module2.wasm');
    const module2Binary = await module2Response.arrayBuffer();
    const module2 = await WebAssembly.instantiate(module2Binary, {
      module1: module1.instance.exports
    });

    // Now we can call functions from module2, which in turn calls functions from module1
    console.log(module2.instance.exports.increment(41)); // Should log 42
  }

  loadAndLinkModules();
</script>

</body>
</html>

```

