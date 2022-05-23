# React Hooks

## useMemo

```jsx
import { useState } from "react";

function App() {
  const [number, setNumber] = useState(0);
  const [dark, setDark] = useState(false);
  const doubleNumber = slowFunction(number);

  const themeStyles = {
    backgroundColor: dark ? "black" : "white",
    color: dark ? "white" : "black",
  };

  return (
    <>
      <input
        type="number"
        value={number}
        onChange={(e) => setNumber(parseInt(e.target.value))}
      />
      <button onClick={() => setDark((prevDark) => !prevDark)}>
        Change Theme
      </button>
      <div style={themeStyles}>{doubleNumber}</div>
    </>
  );
}

function slowFunction(num) {
  console.log("Calling Slow Function");
  for (let i = 0; i < 1000000000; i++) {}
  return num * 2;
}

export default App;
```

If we click the Change Theme button, we will see a delay because the SlowFunction is called every time the component is rendered. So we change state when clicking Change Theme. And that's something that should not happen because Change Theme has nothing to do with running the SlowFunction.

```jsx
import { useState, useMemo } from "react";

function App() {
  const [number, setNumber] = useState(0);
  const [dark, setDark] = useState(false);
  const doubleNumber = useMemo(() => {
    return slowFunction(number);
  }, [number]);

  const themeStyles = {
    backgroundColor: dark ? "black" : "white",
    color: dark ? "white" : "black",
  };

  return (
    <>
      <input
        type="number"
        value={number}
        onChange={(e) => setNumber(parseInt(e.target.value))}
      />
      <button onClick={() => setDark((prevDark) => !prevDark)}>
        Change Theme
      </button>
      <div style={themeStyles}>{doubleNumber}</div>
    </>
  );
}

function slowFunction(num) {
  console.log("Calling Slow Function");
  for (let i = 0; i < 1000000000; i++) {}
  return num * 2;
}

export default App;
```

### Referential Equality

In JavaScript we have referential equality in Objects & Arrays.

```js
const themeStyles = {
  name: "John Rambo",
  nationality: "USA",
};

// Copy
const themeStyles2 = {
  name: "John Rambo",
  nationality: "USA",
};

// Exercise

if (themeStyles === themeStyles2) {
  console.log(true);
} else {
  console.log(false);
}

// And the answer is?
```

```js
import { useState, useMemo, useEffect } from "react";

function App() {
  const [number, setNumber] = useState(0);
  const [dark, setDark] = useState(false);
  const doubleNumber = useMemo(() => {
    return slowFunction(number);
  }, [number]);

  const themeStyles = {
    backgroundColor: dark ? "black" : "white",
    color: dark ? "white" : "black",
  };

  useEffect(() => {
    console.log("Theme Changed");
  }, [themeStyles]);

  return (
    <>
      <input
        type="number"
        value={number}
        onChange={(e) => setNumber(parseInt(e.target.value))}
      />
      <button onClick={() => setDark((prevDark) => !prevDark)}>
        Change Theme
      </button>
      <div style={themeStyles}>{doubleNumber}</div>
    </>
  );
}

function slowFunction(num) {
  for (let i = 0; i < 1000000000; i++) {}
  return num * 2;
}

export default App;
```

When we update the input, we will see that the console.log('Theme Changed') it's being printed, and the reason for that it's because every time we render the component, we create an entirely new themeStyles object which is different than the previous one, even though their values are the same.

```js
const themeStyles = useMemo(() => {
  return {
    backgroundColor: dark ? "black" : "white",
    color: dark ? "white" : "black",
  };
}, [dark]);
```

Now, the themeStyles object it's being created only when we update the dark theme state. If we don't update the theme state, then when the component re-render, it will not create a new themeStyle object, and it will be referencing the same position in memory.

## useCallback

```js
import { useState, useEffect } from "react";

function App() {
  const [number, setNumber] = useState(1);
  const [dark, setDark] = useState(false);

  const getItems = () => {
    return [number, number + 1, number + 2];
  };

  const theme = {
    backgroundColor: dark ? "#333" : "#FFF",
    color: dark ? "#FFF" : "#333",
  };

  return (
    <div style={theme}>
      <input
        type="number"
        value={number}
        onChange={(e) => setNumber(parseInt(e.target.value))}
      />
      <button onClick={() => setDark((prevDark) => !prevDark)}>
        Change Theme
      </button>
      <List getItems={getItems} />
    </div>
  );
}

function List({ getItems }) {
  const [items, setItems] = useState([]);

  useEffect(() => {
    setItems(getItems());
    console.log("Updating Items");
  }, [getItems]);

  return items.map((item) => <div key={item}>{item}</div>);
}

export default App;
```

If we update the input, we will see printed "Updating Items" which is spected because every time we update the input, the app rerenders and getItems function it's being created over and over again. The issue is that we also call the useEffect even when we update the theme state.

```js
const getItems = useCallback(() => {
  return [number, number + 1, number + 2];
}, [number]);
```

The difference between useMemo is that it returns a value, and useCallback returns the entire function. And wwith useCallback we can also pass parameters to our function.

```js
// List
useEffect(() => {
  setItems(getItems(5));
  console.log("Updating Items");
}, [getItems]);

const getItems = useCallback(
  (incrementeor) => {
    return [
      number + incrementor,
      number + 1 + incrementor,
      number + 2 + incrementor,
    ];
  },
  [number]
);
```

Usually, we use useCallback when using the function as argument dependency inside the useEffect hook.
