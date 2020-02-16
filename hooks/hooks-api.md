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

## useEffect

```javascript
useEffect(effectFunction, dependenciesArray);
```

This **Hook** is intended for **imperative side effects** such as API requests, timers or global event listeners. Normally, these **side effects** should be avoided in **function components** as they can lead to unexpected behaviour or bugs that might be hard to solve for.

The `useEffect()` **Hook** combats this problem and allows for a _safe_ mechanism to use side effects within **function components**.

The **Hook** expects a **function** as its first parameter and a **dependency array** as its second. The function is called **after** the component has rendered. If we have passed an **optional dependency array** to this Hook, the function we passed in will only be executed if at least one of the values in the **dependency array** has changed. If an **empty dependency array** has been passed, the function will only be run on the **first render** of the component - similar to `componentDidMount()` lifecycle method which we learned about in class components.

### Cleaning up side effects

Sometimes side effects leave "traces" that have to be cleaned up once a component is no longer in use. If for example you had intervals which you had started with `setInterval()`, these should be stopped with `clearTimeOut()` once the component has been removed. If left untreated, these side effects can lead to actual problems or even memory leaks.

Globally registered event listeners such as `resize` or `orientationchange` which have been added to the `window` object with `addEventListener()` should also be removed once the component unmounts by using `removeEventListener()` so they will not be executed anymore if the component itself is not even part of the component tree anymore.

In order to make this cleanup a bit more systematic and easier, we can return a **cleanup function** from the **effect function**. If an **effect function** returns a **cleanup function**, it is called before each call of the **effect function** with the exception of the very first call:

```javascript
import React, { useState, useEffect } from "react";
import ReactDOM from "react-dom";

const Clock = () => {
  const [time, setTime] = useState(new Date());

  useEffect(() => {
    const intervalId = setInterval(() => {
      setTime(new Date());
    }, 1000);
    return () => {
      clearInterval(intervalId);
    };
  }, []);

  return `${time.getHours()}:${time.getMinutes()}:${time.getSeconds()}`;
);

ReactDOM.render(<Clock />, document.getElementById("root"));
```

In the above example we have set up an interval that starts once the component **mounts**. Once the component **unmounts** we stop the timer as we would otherwise change the state of the component which is not part of the component tree anymore. If we tried to do this, React would inform us with a warning and suggest to **clean up** asynchronous tasks and subscriptions in a **cleanup function**:

{% hint style="danger" %}
Warning: Can't perform a React state update on an unmounted component. This is a no-op, but it indicates a memory leak in your application. To fix, cancel all subscriptions and asynchronous tasks in a useEffect cleanup function.
{% endhint %}

By returning a cleanup function from the **effect function** we can stop the interval with a call of `clearInterval()`. This happens before each call of the effect function, but at the very latest it would happen during Unmounting.

### Conditional calls of the effect function

Normally the `useEffect()` Hook or its associated effect function is executed after each render of a component. This way, we ensure that the effect is executed each time once of its dependencies have changed. If we access state or props of a component within the effect function, the side effect should also be executed if one of the dependencies change. If we wanted to display profile data of a particular user and requested this data from an API, the API request should also be initiated if the user's profile that we want to look at changes while the component is already mounted.

However, this might lead to a lot of unnecessary calls of this function and it might even be executed if no data has actually changed since the last render \(which are relevant for the side effect\). This is why the React allows us to define a **dependency array** as a second parameter in the **effect function**. Only if at least one value in the **dependency array** has changed, the function will be called again. Let's put our previous example into a little code snippet:

```javascript
useEffect(
  () => {
    const user = api.getUser(props.username);
    setUser(user);
  }, 
  [props.username]
);
```

In this example, we have put the username into the dependency array which we use to request data from the API. 

While creating such a **dependency array**, we should take utmost care to include all values that are present in the function and could change within the lifetime of the component in this dependency array. If the effect function should only be run once and perform a similar task such as `componentDidMount()`, you should leave the **dependency array** empty. 

{% hint style="info" %}
In order to facilitate or automate the creation of **dependency arrays**, the  `eslint-plugin-react-hooks` offers an `exhaustive-deps` rule which will automatically write dependencies used in the effect function into the **dependency array** or at least warns that they should be included. These could be run on _Format on Save_ or another similar editor configuration.

You can active it by setting `"exhaustive-deps": "warn"` in the`rules` block of the ESLint configuration.
{% endhint %}

### Sequence of operations

The **effect function** is called **asynchronously** with a little bit of delay after the **layout and paint phase** of the browser. For most side effects this should be sufficient. You might run into situations though in which it is necessary to to **synchronously** run side effects. For example, you might need to perform DOM mutations and a delay in execution would lead to an inconsistent user interface or jarring content on the screen.

To deal with these problems, React introduced the `useLayoutEffect()` Hook. It works almost identical to the `useEffect()` hook: it expects an **effect function** which can also return a **cleanup function** and also contains a dependency array. This dependency array is identical to that in the regular `useEffect()` Hook. The difference in the two Hooks is that the `useLayoutEffect()` hook is executed synchronously and run just after all DOM mutations have finished as opposed to asynchronously as is the case in the regular `useEffect()` hook.

`useLayoutEffect()` can read from the DOM and can also synchronously modify it **before** the browser will display these changes in its **paint phase**.

### Asynchronous effect functions

Even if **effect functions** can be run with delay, they are not allowed to be asynchronously by definition and are not allowed to return a promise. If we tried to perform such an operation, React would give us the following warning:

{% hint style="danger" %}
Warning: An Effect function must not return anything besides a function, which is used for clean-up.

It looks like you wrote useEffect\(async \(\) =&gt; ...\) or returned a Promise. Instead, you may write an async function separately and then call it from inside the effect \[…\]

In the future, React will provide a more idiomatic solution for data fetching that doesn't involve writing effects manually.
{% endhint %}

In the above example, an **incorrect** `useEffect()` Hook could have looked like the following \(please don't do this\):

```javascript
useEffect(async () => {
  const response = await fetch('https://api.github.com/users/manuelbieh');
  const accountData = await response.json();
  setGitHubAccount(accountData);

  fetchGitHubAccount("manuelbieh");
}, []);
```

This is **not allowed** as the effect function is declared with the `async` keyword. So how would we go about solving this problem? There's a relatively simple solution to this problem. We move the asynchronous part of this function into its own asynchronous function **within the effect function** and then only call this function:

```jsx
import React, { useEffect, useState } from "react";
import ReactDOM from "react-dom";

const App = (props) => {
  const [gitHubAccount, setGitHubAccount] = useState();

  useEffect(() => {
    const fetchGitHubAccount = async () => {
      const response = await fetch(`https://api.github.com/users/${props.username}`);
      const accountData = await response.json();
      setGitHubAccount(accountData);
    };

    fetchGitHubAccount();
  }, [props.username]);

  if (!gitHubAccount) {
    return null;
  }

  return (
    <p>
      {gitHubAccount.name} has {gitHubAccount.public_repos} public repos
    </p>
  );
};

ReactDOM.render(
  <App username="manuelbieh" />, 
  document.getElementById("root")
);
```

In this case, the effect function itself is not asynchronous. The asynchronous functionality has been extracted into its own asynchronous function `fetchGitHubAccount()` which defined **inside** of the `useEffect()` hook.

The asynchronous function does not necessarily need to be defined **inside** of the effect function. But the effect function itself is not allowed to be asynchronous.

## useContext

```javascript
const myContextValue = useContext(MyContext);
```

This Hook only expects one parameter: a context type which we create by calling `React.createContext()`. It will then return the value of the next higher context provider in the component hierarchy.

The `useContext()` Hook acts like a context consumer component and causes a re-render of the **function component** as soon as the value of the context in the provider element changes.

Using this hook is optional and it is still possible to create Context consumers in **JSX** of **function components**. However, the Hook is much more convenient and easier to read as no new hierarchical layer is created in the component tree.

## useReducer

```javascript
const [state, dispatch] = useReducer(reducerFunc, initialState, initFunc);
```

The `useReducer()` Hook is an alternative solution for `useState()` and allows us to manage more complex states. It is based on flux architecture in which a **reducer function** creates a new state by being passed the **last state** and a so called **action**.

The **reducer function** is called by executing a **dispatch function** which in turn receives an **action**. The **action** is an object which always has a `type` property and often a `payload` property. From this **action** and the **previous state**, the **reducer function** can then create the **new state**. One could summarize this in the following form: `(oldState, action) => newState`.

Let us have a look at a simple example. We have developed a `Counter` component which can increment or decrement a counter by pressing a + and - button respectively:

```jsx
import React, { useReducer } from "react";
import ReactDOM from "react-dom";

const initialState = {
  count: 0,
};

const reducerFunction = (state, action) => {
  switch (action.type) {
    case "INCREMENT":
      return { count: state.count + 1 };
    case "DECREMENT":
      return { count: state.count - 1 };
    default:
      throw new Error('Unknown action');
  }
};

const Counter = () => {
  const [state, dispatch] = useReducer(reducerFunction, initialState);

  return (
    <div>
      <h1>{state.count}</h1>
      <button onClick={() => dispatch({ type: "INCREMENT" })}>+</button>
      <button onClick={() => dispatch({ type: "DECREMENT" })}>-</button>
    </div>
  );
};

ReactDOM.render(<Counter />, document.getElementById("root"));
```

We have defined the initial State `initialState` and the reducer function `reducerFunction`. The initial state only consists of an object which holds a `count` property which is `0` initially. The reducer function on the other hand expects a `state` and an `action` which are later passed to the reducer function by calling the `dispatch` function. These two parameters will then create the **new** state. **But beware**: instead of mutating an existing state, we always have to create a new state! Otherwise mutations of the existing state will lead to side effects which are not intended and cause incorrect display of components. A **reducer** function should always be a **pure** function.

The **reducer function** as well as the initial state are then passed to the `useReducer()` Hook which will in turn return a tuple. The tuple consists of two values: the first element will portray the **current state** in this particular rendering phase and the second value will be the so-called **dispatch** function.

If we want to change our current state, we call the `dispatch` function and pass this function an **action**. In our current example, we achieve this by clicking one of the two buttons which will either dispatch the `{ type: "INCREMENT" }` \(to increase the counter\) action or `{ type: "DECREMENT" }` \(to decrease the counter\) action/

If an action has been _"dispatched",_ a new state is created and React will trigger a re-render. The new state will now be accessible in the new `state` variable which was returned by the reducer function. If however the same state was returned from the **reducer**, no re-render will be triggered.

### The third parameter

Apart from the `reducer` function and `initialState` which we always need to specify, we can also pass a third optional parameter to the `useReducer()` Hook. This third parameter is called an `init` function which can be used to calculate the initial state. The function could be used to extract the value of a **reducer** in an external function which is outside of the reducer itself.

If such an `init` function has been passed to the Hook, it will be called during the **first** call. The `initialState` will be passed to it as its **initial argument**. This can be really useful if the **initial state** of your component is based on props for example. These props can be passed as the second parameter inside of the `init function` which can then create the initial state of the **reducer** based on these:

```jsx
import React, { useReducer } from "react";
import ReactDOM from "react-dom";

const reducerFunction = (state, action) => {
  switch (action.type) {
    case "INCREMENT":
      return { count: state.count + 1 };
    case "DECREMENT":
      return { count: state.count - 1 };
    default:
      throw new Error("Unknown action");
  }
};

const initFunction = (initValue) => {
  return { count: initValue };
};

const Counter = (props) => {
  const [state, dispatch] = useReducer(
    reducerFunction, props.startValue, initFunction
  );

  return (
    <div>
      <h1>{state.count}</h1>
      <button onClick={() => dispatch({ type: "INCREMENT" })}>+</button>
      <button onClick={() => dispatch({ type: "DECREMENT" })}>-</button>
    </div>
  );
};

ReactDOM.render(<Counter startValue={3} />, document.getElementById("root"));
```

In this example, the `useReducer()` hook has been extended to include a third and optional parameter: the **init** function. The `initialState` is now an argument for the **init** function. The value for this argument is passed to the component via **props**, `startValue` in particular.

