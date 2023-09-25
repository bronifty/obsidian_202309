### Big O
- O(n) linear
	- growth wrt input
	- loops
	- drop constants
	- if conditional consider worst case

- O(1) - constant (no change with input)
- O(log n) - base 2 (linear)
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

### Implementations

- Linear Search
```ts
let index = 0;
export default function linear_search(haystack: number[], needle: number): any {
    for (let i = 0; i < haystack.length; ++i) {
        if (haystack[i] === needle) {
            return true;
        }
    }
    return false;
}
```

- Binary Search
```ts
export default function bs_list(haystack: number[], needle: number): any {
    let lo = 0;
    let hi = haystack.length;
    do {
        const mid = Math.floor(lo + (hi - lo) / 2);
        const val = haystack[mid];
        if (val === needle) {
            return true;
        } else if (val < needle) {
            lo = mid + 1;
        } else if (val > needle) {
            hi = mid; // 
        }
    } while (lo < hi);
    return false;
}
```

- Two Crystal Balls Problem
```ts
export default function two_crystal_balls(breaks: boolean[]): number {
    let interval = Math.floor(Math.sqrt(breaks.length));
    let start = 0;
    for (let i = 0; i < breaks.length; i += interval) {
        if (breaks[i]) {
            break;
        }
        start += interval;
    }
    for (let j = 0; j < breaks.length; ++j) {
        if (breaks[j]) {
            return j;
        }
    }
    return -1;
}
```

- Bubble Sort
```ts
export default function bubble_sort(arr: number[]): void {
    for (let i = 0; i < arr.length; ++i) {
        for (let j = 0; j < arr.length - 1 - i; ++j) {
            if (arr[j] > arr[j + 1]) {
                let intermediateVal = arr[j];
                arr[j] = arr[j + 1];
                arr[j + 1] = intermediateVal;
            }
        }
    }
}
```

- Queue
```ts
type Node<T> = {
    value: T;
    next?: Node<T>;
};
export default class Queue<T> {
    public length: number;
    private head?: Node<T>;
    private tail?: Node<T>;

    constructor() {
        this.head = this.tail = undefined;
        this.length = 0;
    }
    enqueue(item: T): void {
        const node = { value: item } as Node<T>;
        this.length++;
        if (!this.tail) {
            this.tail = this.head = node;
            return;
        }
        this.tail.next = node;
        this.tail = node;
    }
    deque(): T | undefined {
        if (!this.head) {
            return undefined;
        }
        this.length--;
        const head = this.head;
        this.head = this.head.next;
        if (!this.head) {
            this.tail = undefined;
        }
        head.next = undefined;
        return head.value;
    }
    peek(): T | undefined {
        return this.head?.value;
    }
}
```

- Stack
```ts
type Node<T> = {
    value: T;
    prev?: Node<T>;
};
export default class Stack<T> {
    public length: number;
    private head?: Node<T>;

    constructor() {
        this.head = undefined;
        this.length = 0;
    }

    push(item: T): void {
        const node = { value: item } as Node<T>;
        this.length++;
        if (!this.head) {
            this.head = node;
            return;
        }
        node.prev = this.head;
        this.head = node;
    }
    pop(): T | undefined {
        this.length = Math.max(0, this.length - 1);
        if (this.length === 0) {
            const head = this.head as Node<T>;
            this.head = undefined;
            return head?.value;
        }
        const head = this.head;
        this.head = head?.prev;
        return head?.value;
    }
    peek(): T | undefined {
        return this.head?.value;
    }
}
```
### Arrays v Linked List
- usability 
	- if you need to scan or hop, then array
	- if you need to push or pop from the head or tail, Linked List (eg queue)
- time
	- Array access is indices - constant time O(1) 
	- Linked List access is you have to traverse each item - linear time O(n)
- space
	- Array allocated upfront
	- Linked List is one at a time, small memory footprint

- get(idx) impl for LinkedList
```ts
type Node<T> = {
    value: T;
    prev?: Node<T>;
    next?: Node<T>;
};

export default class DoublyLinkedList<T> {
    public length: number;
    private head?: Node<T>;

    constructor() {
        this.length = 0;
    }

    prepend(item: T): void {}
    insertAt(item: T, idx: number): void {}
    append(item: T): void {}
    remove(item: T): T | undefined {}
    get(idx: number): T | undefined {
        let curr = this.head;
        for (let i = 0; i < idx && curr; ++i) {
            curr = curr.next;
        }
        return curr?.value;
    }
    removeAt(idx: number): T | undefined {}
}
```

- ArrayList
```ts
// export default class ArrayList<T> {
//     public length: number;

//     constructor() {
//     }

//     prepend(item: T): void {

// }
//     insertAt(item: T, idx: number): void {

// }
//     append(item: T): void {

// }
//     remove(item: T): T | undefined {

// }
//     get(idx: number): T | undefined {

// }
//     removeAt(idx: number): T | undefined {

// }
// }

export default class ArrayList<T> {
    private items: T[];
    public length: number;
    private capacity: number;

    constructor(capacity: number) {
        this.capacity = capacity;
        this.items = [];
        this.length = 0;
    }

    prepend(item: T): void {
        this.items.unshift(item);
        this.length++;
    }

    insertAt(item: T, idx: number): void {
        if (idx < 0 || idx > this.length) {
            throw new Error("Index out of bounds");
        }
        this.items.splice(idx, 0, item);
        this.length++;
    }

    append(item: T): void {
        this.items.push(item);
        this.length++;
    }

    remove(item: T): T | undefined {
        const index = this.items.indexOf(item);
        if (index !== -1) {
            this.length--;
            return this.items.splice(index, 1)[0];
        }
        return undefined;
    }

    get(idx: number): T | undefined {
        if (idx < 0 || idx >= this.length) {
            return undefined;
        }
        return this.items[idx];
    }

    removeAt(idx: number): T | undefined {
        if (idx < 0 || idx >= this.length) {
            return undefined;
        }
        this.length--;
        return this.items.splice(idx, 1)[0];
    }
}
```

- Pros and Cons of Array List versus regular linked list 
	- random access, traversal, array brackets are easy access with array
	- enqueue (FIFO) is linear time required to shift every item right from the tail to the head in an ArrayList, whereas in a regular linked list, this is constant time
	- push/pop (LIFO) stack structure is easier 
- interview question: how would you implement an async request queue? linked list because adding and removing from the ends is constant time in queue and there is no reason to use random access

