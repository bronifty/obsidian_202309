
### CloudFlare Pages Functions (workers for pages)
- cf pages functions support wasm via wasm-pack such as in the writeup here: [pages functions with wasm](https://blog.cloudflare.com/pages-functions-with-webassembly)
- the workflow in the example is dependent on rust binary binding to javascript 
- [github example repo](https://github.com/bronifty/cloudflare-globe-distance-between-us-rust-wasm-bindgen-pages-functions-wasm?tab=readme-ov-file)

### ByteCode Alliance Component Model
- bytecode alliance supports "componentize-js", which wraps the wasm binary in a javascript "component" (maybe like a class?), which is then transpiled into an ESM module with jco, which can be imported into and used by js files. 
- [github example repo](https://github.com/bronifty/componentize-js)

