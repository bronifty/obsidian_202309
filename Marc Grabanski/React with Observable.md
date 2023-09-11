****```tsx
import React from "react";
import { child, parent, grandparent } from "../store";
function Descendants() {
  const [childValue, setChildValue] = React.useState(child.value);
  const [parentValue, setParentValue] = React.useState(parent.value);
  const [grandparentValue, setGrandparentValue] = React.useState(
    grandparent.value
  );
  React.useEffect(() => {
    const childSubscription = child.subscribe((value) => {
      setChildValue(value);
    });
    const parentSubscription = parent.subscribe((value) => {
      setParentValue(value);
    });
    const grandParentSubscription = grandparent.subscribe((value) => {
      setGrandparentValue(value);
    });
    return () => {
      childSubscription();
      parentSubscription();
      grandParentSubscription();
    };
  }, []);
  const handleButtonClick = () => {
    child.value += 1;
  };
  return (
    <div>
      <div>Child Value: {childValue}</div>
      <div>Parent Value: {parentValue}</div>
      <div>Grandparent Value: {grandparentValue}</div>
      <button onClick={handleButtonClick}>Update Child Value</button>
    </div>
  );
}
export default Descendants;
```


```ts
// store.ts
import { ObservableFactory, IObservable } from "./observable/observable";

const child = ObservableFactory.create(() => 1);
const parent = ObservableFactory.create(() => child.value + 1);
const grandparent = ObservableFactory.create(() => parent.value + 1);

const booksChild = ObservableFactory.create([
  { name: "Book 1", author: "Author 1" },
  { name: "Book 2", author: "Author 2" },
]);
const booksParent = ObservableFactory.create(() => booksChild.value);

// const secondComputedFunction = () => firstComputed.value;
// const secondComputed = ObservableFactory.create(secondComputedFunction);

// const thirdComputedFunction = () => secondComputed.value;
// const thirdComputed = ObservableFactory.create(thirdComputedFunction);

export { child, parent, grandparent, booksChild, booksParent };
```


