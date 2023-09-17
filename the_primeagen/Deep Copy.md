
- class with interface
```ts
interface IPerson {
  getName(): string;
  clone(): IPerson;
}

class Person implements IPerson {
  constructor(
    public name: string, 
    public age: number, 
    public address: { street: string, city: string, additionalInfo: { zipcode: string, country: string } }
  ) {}

  getName(): string {
    return this.name;
  }

  clone(): Person {
    return new Person(
      this.name, 
      this.age, 
      { 
        ...this.address, 
        additionalInfo: { ...this.address.additionalInfo } // Creating a copy of the nested object to achieve deep copy
      }
    );
  }
}

// Usage example:
const original = new Person("John", 30, { street: "123 Main St", city: "Hometown", additionalInfo: { zipcode: "12345", country: "USA" } });

const copy = original.clone();

copy.name = "Doe";
copy.address.city = "Another Town";
copy.address.additionalInfo.zipcode = "54321";

console.log(original.getName()); // "John"
console.log(original.address.city); // "Hometown"
console.log(original.address.additionalInfo.zipcode); // "12345"

console.log(copy.getName()); // "Doe"
console.log(copy.address.city); // "Another Town"
console.log(copy.address.additionalInfo.zipcode); // "54321"

```

- object
```ts
function deepCopy<T>(obj: T): T {
  return JSON.parse(JSON.stringify(obj));
}

// Usage example:

interface Person {
  name: string;
  age: number;
  address: {
    street: string;
    city: string;
  };
}

const original: Person = {
  name: "John",
  age: 30,
  address: {
    street: "123 Main St",
    city: "Hometown"
  }
};

const copy: Person = deepCopy(original);

copy.name = "Doe";
copy.address.city = "Another Town";

console.log(original.name); // "John"
console.log(original.address.city); // "Hometown"
console.log(copy.name); // "Doe"
console.log(copy.address.city); // "Another Town"

```

- shallow copy
```ts
interface Person {
  name: string;
  age: number;
  address: {
    street: string;
    city: string;
  };
}

const original: Person = {
  name: "John",
  age: 30,
  address: {
    street: "123 Main St",
    city: "Hometown"
  }
};

// Shallow copy using Object.assign
const shallowCopy = Object.assign({}, original);

shallowCopy.name = "Doe"; // Modifying a top-level property affects only the copy
shallowCopy.address.city = "Another Town"; // Modifying a nested object affects both the copy and the original

console.log(original.name); // "John"
console.log(original.address.city); // "Another Town"
console.log(shallowCopy.name); // "Doe"
console.log(shallowCopy.address.city); // "Another Town"

```

