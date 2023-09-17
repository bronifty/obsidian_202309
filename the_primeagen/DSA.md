### Big O
- O(n) linear
	- growth wrt input
	- loops
	- drop constants
	- if conditional consider worst case

- O(1) - constant (no change with input)
- O(logn) - base 2 (linear)
- O(n) - linear base 10
- O(n^2) - exponential
- O(2^n) - unsolvable
- O(n!) - unsolvable

O(n^2)
- nested loop 
```js
for (let i=0; i<n.length; i++) {
	for (let j=0; j<n.length; i++) {
	}
}
```

O(n^3)
- two nested loops 
```js
for (let i=0; i<n.length; i++) {
	for (let j=0; j<n.length; i++) {
		for (let k=0; k<n.length; i++) {
		}
	}
}
```

O(n log n)
- Quicksort

O(log n)
- Binary search tree

O(sqrt(n))
- 