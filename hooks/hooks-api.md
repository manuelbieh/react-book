# Hooks API

In this chapter I want to summarize all the hooks which are available to us internally and describe how they can be used and when. The official React documentation differentiates between three basic Hooks and seven additional hooks. These additional hooks are often used for very specific use cases \(such as performance optimizations\) or they are extensions of the basic hooks.

The three **basic hooks** which I mentioned briefly beforehand are:

* `useState`
* `useEffect`
* `useContext`

The **seven additional hooks** are:

* `useReducer` 
* `useCallback` 
* `useMemo` 
* `useRef` 
* `useImperativeHandle` 
* `useLayoutEffect` 
* `useDebugValue`

## useState

```javascript
const [state, setState] = useState(initialState);
```

This Hooks returns a **value** as well as a **function** to us, which we can use to update the **value** itself. During the first rendering of a component that uses this hook, this value is equal to the `initialState` which you have passed to the state. If the parameter that has been passed in is a function, it will use the return value of the function as its initial value.

When the update function is called, React ensures that the function always has the same **identity** and does not create a new function whenever the hook is called. This is important as it reduces the number of renders and also means that we do not need to pass any other **dependencies** \(as is the case in `useEffect()` or `useCallback()`\).

`useState()` will return an **array** to us of which the first **value** always denotes the state and the second value is always a **function** which we use to update said **value**. Due to array destructuring, we are not limited in naming this value and function. However, conventions have developed that follow the pattern of `value / setValue`. For example, `user` and `setUser`.  But of course you could also go for something along the lines of this: `changeUser` and `updateUserState`.

The mechanism of actually updating the state is very similar to that of `this.setState()` which we already encountered in the chapter on Class components. The function can either receive a **new value** which is then replacing the current old value or we can pass an **updater function** to the the function. The updater function receives the previous value and uses the **return value** from the function as its new state.

But be careful: in contrast to `this.setState()`, objects are **not merged** with their previous state but the old state is completely **replaced** by the new state.

To illustrate this, let's have a look at the following example:

```jsx
import React, { useState, useEffect } from "react";
import ReactDOM from "react-dom";

class StateClass extends React.Component {
  state = { a: 1, b: 2 };

  componentDidMount() {
    this.setState({ c: 3 });
    this.setState(() => {
      return { d: 4 };
    });
  }

  render() {
    // { a: 1, b: 2, c: 3, d: 4 }
    return <pre>{JSON.stringify(this.state, null, 2)}</pre>;
  }
}

const StateHook = () => {
  const [example, setExample] = useState({ a: 1, b: 2 });

  useEffect(() => {
    setExample({ c: 3 });
    setExample(() => {
      return { d: 4 };
    });
  }, []);

  // { d: 4 }
  return <pre>{JSON.stringify(example, null, 2)}</pre>;
};

const App = () => {
  return (
    <>
      <StateClass />
      <StateHook />
    </>
  );
};

ReactDOM.render(<App />, document.getElementById("root"));
```

While `StateClass` collects and merges the data of all calls of `this.setState()`, the `setState()` function in the `StateHook` completely replaces the the old value with the new one. In the **class component,** the output returned to us will be `{a: 1, b: 2, c: 3, d: 4}` whereas the **function component** containing the hook will only return `{d: 4}` as this has been written into state last.

If the updater function returns the **exact same value** as the value currently in state, the state update is cancelled and no re-render or side-effects are triggered.



