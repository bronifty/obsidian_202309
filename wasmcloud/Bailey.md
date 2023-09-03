[june bytecode alliance community stream](https://www.youtube.com/watch?v=qynKt8b3yOw&t=259s)

```ts
world test-reactor {
	import wasi:cli-base/environment
	import wasi:filesystem/filesystem

	export add-strings: func(s: list<string>) -> u32
	export get-strings: func() -> list<string>
}
```


- wasm component model world are a component model
- wasi is a set of interfaces
- world is an env in which a program runs
	- imports are avail to component 
	- component is expected to provide exports
	- 