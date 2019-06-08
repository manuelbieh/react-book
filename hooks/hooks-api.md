# Hooks API

In this chapter I would like to describe all available internal hooks, their exact use and their fields of application. In the React documentation, these are divided into three general \(_basic_\) and seven additional \(_additional_\) hooks. The additional hooks are intended especially for special applications \(e.g. performance optimization\) or they are modifications of the general hooks.

The three **general hooks** include the hooks already presented in the previous chapters:

* `useState`
* `useEffect`
* ¶ `useContext`

The seven more hooks are:

* ¶ `useReducer` 
* ¶ `useCallback` 
* `useMemo` 
* ¶ `useRef` 
* `useImperativeHandle` 
* ¶ `useLayoutEffect` 
* `useDebugValue`

## useState

```javascript
const [state, setState] = useState(initialState);
```

This hook returns a **value** and a **function,** to update it. During the first rendering of a component that uses the `useState()` hook, this value corresponds to the parameter passed as `initialState`. If the passed parameter is a function, React uses the function's return value as the initial value.

With the update function React makes sure that it always has the same **identity**, so the function does not rebuild every time the hook is called. This is important to avoid unnecessary rendering and also means that it does not have to be passed as **Dependency** to other **Hooks** like `useEffect()` or `useCallback()`.

Specifically, `useState()` returns a **Array** in which the first element is always the **value** of the state and the second element is always a **function** to update this value. The array destructuring syntax does not impose any formal restrictions on the naming of values and functions. However, it quickly became clear that in the vast majority of cases the form `value`/`setValue` is adhered to. Like `user` and `setUser`. But also other names like `changeUser` or `updateUserState` would be possible.

The mechanism for updating the state here works similar to `this.setState()` in class components. Thus, either a **new value** can be passed to the function, which then replaces the old value, or a **Updater function**. This receives the last value and uses the **return value** from the function as the new state.

But beware, there is a decisive difference: unlike `this.setState()` objects **are not merged** with the existing state, but the state is **completely** replaced by the new state **!**.

To illustrate this difference, let's take a look at the direct comparison:

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

While the `StateClass` collects the data of all calls of `this.setState()` and merges it with the existing state, the `setState()` function in the `StateHook` overwrites the complete state and replaces the complete old value with the new value. So the output is in the **class component**: `{a: 1, b: 2, c: 3, d: 4}` and in the **function component** with the hook a simple `{d: 4}`, because this was last written into the state.

If the update function returns exactly the same value as the previous one as the new state, this is equivalent to aborting a state update and no rendering is triggered and no page effects are triggered.

## useEffect

```javascript
useEffect(effectFunction, dependenciesArray);
```

This **Hook** is intended for **imperative side effects** such as API requests, timers or global event listeners. These are not allowed within **Function Components**, or at least should be avoided, as they can and will lead to untraceable behavior and possibly hard-to-fix bugs.

The `useEffect()`-**Hook** provides a remedy and allows the _safe_ use of page effects even within a **Function Component**.

The **Hook** expects a **Function** as first parameter and optionally a **Dependency Array** as second parameter. The function is called **after** a component has been rendered. If an optional **Dependency Array** is passed, the function is only executed if at least one of the values from the **Dependency Array** has changed since the last execution of the function. If an empty **Dependency Array** is passed, i.e. `[]`, the function is only called during the **first** rendering of the component, comparable to the `componentDidMount()` **Lifecycle method**, which we already know from class components.

### Clean up side effects

In some cases, side effects leave "tracks" that need to be cleaned up when a component is no longer in use. For example, if intervals are started using `setInterval()`, they should be stopped when removing the component via `clearTimeout()`, otherwise it can lead to problems such as memory leaks in the worst case.

Globally registered event listeners like `resize` or `orientationchange` which are added to the `window` object via `addEventListener()` should also be removed by `removeEventListener()` at the latest when unmounting a component, so that the code is no longer executed when the component is no longer in the page tree.

For this purpose it is possible to return a **Cleanup function** from the **Effect function**. If a **Effect function** returns a **Cleanup function**, it is executed before each call to the **Effect function**, except before the first call:

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

In the above example, we install an interval when **mounting** the component. When **unmounting** the component, we stop the timer, as otherwise we would change the state of a component that is no longer in the page tree. This would acknowledge React with an error message, warn us of memory leaks, and tell us that subscriptions and other asynchronous tasks in the **Cleanup** function need to be cleaned up:

{% hint style="danger" %}
Warning: Can't perform a React state update on an unmounted component. This is a no-op, but it indicates a memory leak in your application. To fix, cancel all subscriptions and asynchronous tasks in a useEffect cleanup function.
{% endhint %}

By returning a cleanup function from the effect function, we can cancel the interval by calling `clearInterval()`. This happens before each new call of the effect function, but at the latest during unmounting.

### Conditional calls of the effect function

By default, the `useEffect()` hook or its effect function is executed again after each rendering of the component. This ensures that the effect is executed every time some of the dependencies it uses \(_Dependencies_\) change. If we access the state or the props of a component within the effect function, for example, the side effect should also be executed again if one of these dependencies changes. If we want to display profile data about a user and get it via an API, the API request should of course also be initiated if the user whose profile we want to view changes while the component is mounted.

However, this sometimes leads to many unnecessary calls of the function and also to the function being executed under certain circumstances if no data relevant to the page effect has changed since the last rendering. For this purpose React offers us the possibility to define a **Dependency Array** as a second parameter. Here we can and should enter all values that should cause the effect function to be executed again. The function is only executed again if at least one value in the **Dependency Array** has changed. To take our user profile as an example, here would be the user name or ID that we use to retrieve user data from the API.

```javascript
useEffect(
  () => {
    const user = api.getUser(props.username);
    setUser(user);
  }, 
  [props.username]
);
```

When creating such a **Dependency Array** you should make sure that all values that are used within the function and that can change during the lifetime of the component are listed there. If the effect function is to be executed only once, i.e. for a purpose similar to `componentDidMount()` in class components, an empty array \(i.e. `[]`\) is passed.

{% hint style="info" %}
To facilitate or even automate the work of creating **Dependency Arrays**, the `eslint-plugin-react-hooks` contains the `exhaustive-deps` rule in the `eslint-plugin-react-hooks`, which automatically enters the dependencies used in the effect function into the **Dependency Array** or at least warns if inconsistencies are found, if the editor is configured accordingly \(e.g. _Format on Save_\).

It can be activated by the entry `"exhaustive-deps": "warn"` in the `rules` block of the ESLint configuration.
{% endhint %}

### Temporal sequence

The **Effect** function is executed **asynchronously** by the browser with a delay after the **Layout-** and **Paint phases**. This should be sufficient for most side effects. However, there may be situations where there is no way around **synchronous** side effects. This may be the case if, for example, DOM mutations are involved and the delayed execution would cause the user to experience a short flicker or inconsistent user interface.

For this purpose, the `useLayoutEffect()` hook was introduced. It works identical to the `useEffect()` hook, expects a **Effect function** as well, can return a **Cleanup function** in the same form, and the **Dependency Array** works identical to the `useEffect()` hook. The difference here is that it is executed synchronously \(instead of asynchron\) after all DOM mutations have been written.

The `useLayoutEffect()`-Hook can therefore read from the DOM and also modify it synchronously, **before** the browser displays the changes in its paint phase.

### Asynchronous effect functions

Even if **effect functions** are executed with a delay, they themselves must not be asynchronous or return any promises. Otherwise we get an error message in which React informs us of this fact:

{% hint style="danger" %}
Warning: An Effect function must not return anything besides a function, which is used for clean-up.

It looks like you wrote useEffect\(async \(\) =&gt; ...\) or returned a Promise. Instead, you may write an async function separately and then call it from inside the effect \[...\]

In the future, React will provide a more idiomatic solution for data fetching that doesn't involve writing effects manually.
{% endhint %}

In the above example, the **incorrect** `useEffect()` hook would have looked something like this:

```javascript
useEffect(async () => {
  const response = await fetch('https://api.github.com/users/manuelbieh');
  const accountData = await response.json();
  setGitHubAccount(accountData);

  fetchGitHubAccount("manual");
}, []);
```

This is **not allowed**, because the effect function is declared as _asynchronous_ with the `async` keyword. So how do we solve the problem? Well, the solution in this case is relatively simple: We move the asynchronous part of the function into a separate, asynchronous function **within the effect function** and then just call this function:

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
    return zero;
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

In this case, the effect function itself is no longer asynchronous, but the asynchronous functionality is outsourced to the asynchronous function `fetchGitHubAccount()`, which we define **within** the `useEffect()` hook.

The asynchronous function does not necessarily have to be created **in** the effect function. The effect function itself must not be asynchronous.

## useContext

```javascript
const myContextValue = useContext(MyContext);
```

This hook expects a context type created with `React.createContext()` as the only parameter and then returns the value of the next higher context provider of the corresponding type in the component hierarchy.

The `useContext()` hook behaves like a context consumer component and causes a rendering of the **Function Component,** as soon as the value of the context in the respective provider element has been changed.

The use of the hook is optional and so it is still possible to use context consumer in the **JSX** of the **Function Component**. However, the hook is the much clearer variant because it does not create a new hierarchy level in the component tree.

## useReducer

```javascript
const [state, dispatch] = useReducer(reducerFunc, initialState, initFunc)
```

The `useReducer()` hook is an alternative to `useState()` that allows you to manage more complex states. The hook is based on the Flux architecture where, in short, a **Reducer-** function creates a **new state** by passing the **last state** and a so-called **Action**.

The **Reducer** function is called by calling a **Dispatch** function, which in turn receives a **Action**. The **Action** itself is an object that has a `type` property and often \(optional\) a `payload` property. From this **Action** and the **last state** the **Reducer** function then generates the **new state**. The **Reducer** function therefore has the form `(oldState, action) => newState`.

Let's take a look at a simple example and develop a simple `counter` component for this purpose, with which we can count up and down a value using two buttons \(`+` and `-`\):

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
      throw new Error;
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

First we define the initial state \(`initialState`\) and the reducer function \(`reducerFunction`\). The **initial state** consists of only one object with a `count` property that is initial `0`. The **Reducer** function expects a `state` and an `action`. These are passed by React to the Reducer later by calling the **Dispatch** function. We then generate the new state from these two parameters. **Important:** Here we must always create a new state instead of mutating the existing one, since a mutation of the existing state can lead to unwanted side effects and incorrect representation. A **Reducer** function must therefore be a **Pure Function**!

We then put the **Reducer** function and the initial state into the `useReducer()` hook, which then returns a tuple similar to the `useState()` hook, in which the first element is always the **current state** in the current rendering phase, the second element is then the said **Dispatch** function.

If we now want to change the state, we call the `dispatch` function in the component and pass a **Action** to it. This happens in our example component by clicking on one of the two buttons, which then "disperse" the action `{type: "INCREMENT"}` \(to count up\) or `{type: "DECREMENT"}` \(to count down\)".

If an action is _dispatched_ and a new state is created, React triggers a rendering and the newly created state is available in the `state` variable returned by the Reducer function. However, if the same state is returned by the **Reducer** function, **no** rendering is triggered!

### The third parameter

In addition to the `reducer` function and the `initialState`, which must be specified, the `useReducer()` hook can also have a third optional parameter, namely an `init` function for calculating the initial state. The function can be used, for example, to calculate the initial value of the **Reducer** in an external function outside the Reducer.

If such an `init` function is passed, it is called by the hook at the **first** call and the `initialState` is passed as its **initial argument**. This can be particularly useful if the **initial state** is based on the **props** of a component. These can then be passed on in the second parameter to the `init` function, which can generate the initial state of the reducer based on this:

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
      throw new Error;
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

In this example, we extend the `useReducer()` hook with the third optional parameter: the **Init-** function. The `initialState` in the first example becomes an argument for the **Init-**function. The value for this function argument is passed to the component here as `startValue` via the props.

### Reducer in practice

The principle of **Reducers** might have been first introduced to a large part of the React community by the rise of **Redux**. **Redux** is a package to manage global states comfortably and was often used in the past, if the handling of local states in React became too confusing or over too many components away by so-called "Props Drilling". \(i.e., passing props across multiple hierarchical levels\) became too cumbersome.

**Redux** \(and therefore the name\) consists in its core in administering Reducer functions and making their state and dispatch functions available in the components that are to read or change the global state. With the `useReducer()`-Hook React now gets its own function to manage complex state management with Reducer functions.

A typical use case for Reducer is status management for API requests. As a familiar pattern, it has emerged here, for example, to define three different actions for each API request:

* An action that informs the application that data is loaded when the request is started,
* an action that resets the load status and, if the request failed, notifies the state that an error occurred \(and if applicable, which\),
* An action that writes the data received from the API to the state if the request was successful.

Let's take a look at such an example and reload our account data via the GitHub API:

```jsx
import React, { useEffect, useReducer } from "react";
import ReactDOM from "react-dom";

const initialState = {
  data: zero,
  isLoading: false,
  isError: false,
  lastUpdated: zero,
};

const accountReducer = (state, action) => {
  switch (action.type) {
    case "REQUEST_START":
      return {
        ...state,
        isLoading: true,
      };
    case "REQUEST_SUCCESS":
      return {
        ...state,
        data: action.payload,
        isLoading: false,
        isError: false,
        lastUpdated: action.meta.lastUpdated,
      };
    case "REQUEST_ERROR":
      return {
        ...state,
        isLoading: false,
        isError: true,
      };
  }
};

const RepoInfo = (props) => {
  const [state, dispatch] = useReducer(accountReducer, initialState);

  useEffect(() => {
    const fetchGitHubAccount = async (username) => {
      try {
        dispatch({
          type: "REQUEST_START",
        });
        const response = await fetch(
          `https://api.github.com/users/${username}`
        );

        const accountData = await response.json();
        dispatch({
          type: "REQUEST_SUCCESS",
          payload: accountData,
          meta: {
            lastUpdated: new Date(),
          },
        });
      } catch (err) {
        dispatch({
          type: "REQUEST_ERROR",
          error: true,
        });
      }
    };

    fetchGitHubAccount(props.username);
  }, [props.username]);

  if (state.isError) {
    return <p>An error has occurred.</p>;
  }

  if (state.isLoading) {
    return <p>Data is loaded.</p>;
  }

  if (!state.data) {
    return <p>No GitHub account was loaded</p>;
  }

  return (
    <p>
      {state.data.name} has {state.data.public_repos} public repos
    </p>
  );
};

ReactDOM.render(
  <RepoInfo username="manuelbieh" />,
  document.getElementById("root")
);
```

We use the `useReducer()` hook again and pass the `accountReducer` function to it. In this function we react to the three **Actions** of the **Type** `REQUEST_START`, `REQUEST_SUCCESS` and `REQUEST_ERROR`.

The `initialState` consists of an object with an empty `data` property, the two flags `isFetching` and `isError`, which later tell our component whether data is being loaded or whether an error has occurred, and a `lastUpdated` property, in which we store the time of the last successful request. We can use these later to execute requests only once per minute, for example, or to signal to the user that he sees data that has not been updated for a long time.

We also use a `useEffect()` hook to cause the data to load when the GitHub username passed to the **Props** changes. If this is the case, the `REQUEST_START`-**Action** is dispatched first. The **Reducer** then creates the following new state:

```diff
{
  data: zero,
- isLoading: false,
+ isLoading: true,
  isError: false,
  lastUpdated: zero,
}
```

As a result, the following condition now applies in our component below:

```jsx
if (state.isLoading) {
  return <p>Data is loaded.</p>;
}
```

And signals to the user that data is currently being loaded.

Next, two cases can arise: Either the request fails or we successfully obtain data from the API.

If the request fails, the `REQUEST_ERROR`-**Action** would be dispatched. Our state would then change as follows:

```diff
{
  data: zero,
- isLoading: true,
+ isLoading: false,
- isError: false,
+ isError: true,
  lastUpdated: zero,
};
```

Since no more requests are made, the `isLoading` flag is reset from `true` to `false` so as not to falsely signal to the user that we are still loading data. Since an error occurred, we set `isError` from `false` to `true` at the same time. Instead of the load message, the following condition now applies and informs the user of the error that has occurred:

```jsx
if (state.isError) {
  return <p>An error has occurred.</p>;
}
```

At this point it would be advisable to tell the user what went wrong and how to fix the problem. Maybe the given username does not exist and you could give it the chance to correct it or the API is not available at the moment, then you could give the user the possibility to repeat the request at a later time.

In case everything went fine and we could get data correctly from the API, the `REQUEST_SUCCESS`-**Action** is dispatched. In addition to the `payload`, which corresponds to the loaded data, this also contains a `meta` property with which we specify the time of the request.

The **new state** generated by the **reducer** differs from the last state as follows:

```diff
{
- data: zero,
+ data: {
+ "login": "manuelbieh",
+ "name": "Manuel Bieh",
+ "public_repos": 59,
+ [...]
+ },
- isLoading: true,
+ isLoading: false,
  isError: false,
- lastUpdated: zero,
+ "2019-03-19T02:29:10.756Z."
}
```

Our `data` property now contains the data we got from the API. The `isLoading` status is reset to `false`, the `lastUpdated` property is set to the time when we also wrote the data into the state. The desired output should now be generated in our component:

```jsx
return (
  <p>
    {state.data.name} has {state.data.public_repos} public repos
  </p>
);
```

So we have implemented the first complex Reducer as well as the first interaction of both **Hooks** `useEffect()` and `useReducer()` in a practical example!

By the way: Just like the `useState()`-Hook, the `useReducer()`-Hook does not trigger a new rendering if the Reducer function returns the same state as before!

## useCallback

```javascript
const memoizedFunction = useCallback(callbackFunction, dependencyArray);
```

The `useCallback()` hook is used to optimize the performance of an application. It expects a function and creates a **unique identity** of that function that will persist until the **dependencies** of the hook change.

This is intended to always pass the same reference to a function to components that either implement `PureComponents`, their own `shouldComponentUpdate()` method, or are enclosed by `React.memo()`.

The `useCallback()`-Hook expects a function as first argument and a Dependency Array \(like the `useEffect()`-Hook\) and in return returns a "stable" reference to the submitted function. Stable means that it only changes when the **Dependencies** of the hook have changed. Up to this point, a reference created via `useCallback()` for `PureComponents` or components with `React.memo()` is the same.

What may sound complicated at first can best be explained, as is so often the case, by means of an example:

```jsx
import React, { useState, useCallback } from "react";
import ReactDOM from "react-dom";

const FancyInput = React.memo(({ name, onChange }) => {
  console.log("Rendering FancyInput");
  return (
    <input type="text" name={name} onChange={onChange} />
  );
});

const Form = () => {
  const [values, setValues] = useState({});

  const changeHandler = (e) => {
    const { name, value } = e.target;

    setValues((state) => {
      return {
        ...state,
        [name]: value,
      };
    });
  };

  return (
    <>
      <pre>{JSON.stringify(values, null, 2)}</pre>
      <FancyInput name="example" onChange={changeHandler} />
    </>
  );
};

ReactDOM.render(<Form />, document.getElementById("root"));
```

Here we see the two components `Form` and `FancyInput`. The `Form` component renders a `FancyInput` component and gives it a `onChange` function in addition to the `name` attribute. This changes the state of the `Form` component with every change in the input field and thus triggers a rendering.

The `changeHandler` function itself is **created in the form component**, with each new rendering. I.e. ** the reference to the function changes**, because we create a new function with each new rendering. In plain English: we pass the **same**, but not **the same** function.

Thus the optimization of `React.memo()`, which we placed around the `FancyInput` component, **does not** apply. Briefly for refreshing: `React.memo()` checks **before** rendering a component whether its **props** have changed compared to the last rendering and triggers a rendering if this condition is met. Since we rebuild the `changeHandler` function with every rendering of the `Form` component, this condition **always** is true and the `FancyInput` component is also re-rendered accordingly with every rendering of the `Form` component.

This is where `useCallback` comes in. Wrapping our `changeHandler` function in this hook, React creates a function with a **unambiguous** and **stable** reference and returns it to us so that we can safely pass it on to the `FancyInput` component without triggering a rendering every time:

```jsx
const changeHandler = useCallback((e) => {
  const { name, value } = e.target;

  setValues((state) => {
    return {
      ...state,
      [name]: value,
    };
  });
}, []);
```

Here we now use the optimization possibilities of `React.memo()` \(or in class components `PureComponent`\) and do not unintentionally trigger a rendering of the respective child component.

If the function is based on values that can change during the lifetime of the component, these can be specified as the second parameter of `useCallback()` in a **Dependency Array**, just like in the `useEffect()` hook. React then creates a new function with a new reference when a dependency changes.

{% hint style="info" %}
Like the `useEffect()`-Hook, the `exhaustive-deps` rule of the `eslint-plugin-react-hooks` helps to create the **Dependency Array** correctly.
{% endhint %}

## useMemo

```javascript
const memoizedValue = useMemo(valueGetterFunction, dependencyArray);
```

The `useMemo()` hook is next to `useCallback()` the second **hook** for hardcore **performance optimization**. `useMemo()` works quite similar, but with the decisive difference that the function given in is not provided with a unique identity, but with the return value from the function given in the `useMemo()` hook.

In a nutshell, that means:

```javascript
useCallback(fn, deps);
```

corresponds to:

```javascript
useMemo(() => fn, deps);
```

While `useCallback()` returns us the **input function** in a _memoized_ \ version, `useMemo()` returns the **return value** of the input function as _memoized_ version. `useMemo()` can be used for functions that perform very computation-intensive tasks and should not be executed every time a rendering is performed.

Let's take a look at a non-optimized component:

```jsx
import React, { useState, useMemo } from "react";
import ReactDOM from "react-dom";

const fibonacci = (num) =>
  num <= 1 ? 1 : fibonacci(num - 1) + fibonacci(num - 2);

const FibonacciNumber = (value }) => {
  const result = fibonacci(value);
  return (
    <p>{value}: {result}</p>
  );
};

const App = () => {
  const [values, setValues] = useState([]);

  const handleKeyUp = (e) => {
    const { key, target } = e;
    const { value } = target;
    if (key === "Enter") {
      if (value > 40 || value < 1) {
        alert('Invalid value');
        return;
      }
      setValues((values) => values.concat(target.value));
    }
  };

  return (
    <>
      <input type="number" min={1} max={40} onKeyUp={handleKeyUp} />
      {values.map((value, i) => (
        <FibonacciNumber value={value} key={`${i}:${value}`} />
      ))}
    </>
  );
};

ReactDOM.render(<App />, document.getElementById("root"));
```

Our small app consists first of all of an input field for numbers. If a number is entered and the **Enter** key is pressed, this number is written to the `values` state. In this case, this is an array that holds all our entered numbers. The component then iterates through all entered numbers and renders a `FibonacciNumber` component, which gets the value \ (the respective number\) passed.

The `FibonacciNumber` component calculates the corresponding Fibonacci number for this number and displays it. Depending on the number and the calculating power the calculation of the given number can take some time \(on my computer this is 2-3 seconds\ for the 40th Fibonacci number).

This calculation is now carried out again when **every** number is entered for **every** existing Fibonacci number. So if I enter 40, wait about 3 seconds until my computer has finished calculating and enter 40 again, I wait already twice 3 seconds because the value in both components is calculated again.

This is where `useMemo()` comes in. By a supposedly simple change of the line

```javascript
const result = fibonacci(value);
```

into these:

```javascript
const result = useMemo(() => fibonacci(value), [value]);
```

... we generate a _memoized_ value.

This means that React calculates the value at **first rendering**, remembers the value **and recalculates it only when the value of the `value` prop for exactly this component has changed. If the value or more precisely the **Dependencies** between two renderings does not change, React uses the value of the last calculation, but without performing the calculation again.

**But beware:** All this happens on the basis of **a** call to the `useMemo()` hook. If I call the same function twice in two different `useMemo()` hooks, the calculation is performed separately for each of the two hooks, even if both use the same function with the same parameters. The second hook **does not** use the result of the first calculation!

## useRef

```javascript
const ref = useRef(initialValue);
```

The `useRef()`-Hook is, as the name suggests, the hook variant to create **Refs**:

```jsx
import React, { useEffect, useRef } from "react";
import ReactDOM from "react-dom";

function App() {
  const inputRef = useRef();
  useEffect(() => {
    inputRef.current.focus();
  }, []);

  return <input ref={inputRef} />;
}

ReactDOM.render(<App />, document.getElementById("root"));
```

But this is only half of the truth, because in **Function Components**, **Refs** serve another purpose: They make it possible to create a **changeable reference** that will last for the entire lifetime of a component \(i.e. until it is unmounted\). It also performs the tasks of instance variables for class components.

The way `useRef()` works is that it gets an optional initial value, and returns an object with a `current` property that can then be accessed within the **Function Component**. And access here means both reading and writing. For example, if we want to keep data whose change does not trigger rendering, but whose reference remains between two renderings of the component, we use the `useRef()` hook.

## useLayoutEffect

```javascript
useLayoutEffect(effectFunction, dependenciesArray);
```

I had already teased the `useLayoutEffect()`-Hook briefly during the explanation of the `useEffect()`-Hook, it works basically exactly like the `useEffect()`-Hook, but differs from the `useEffect()`-Hook by the time of its execution and its synchronous nature.

While `useEffect()` delays **after** the **Layout** and **Paint-**phase of the browser, layout effects are executed synchronously **after** the **Layout** and **before** the **Paint** phase. This in turn means that they have the chance to read and change the current layout from the DOM **before** the browser displays the changes.

This behavior is the same as `componentDidMount()` and `componentDidUpdate()` in **class components**, but for performance reasons it is recommended to use the `useEffect()` hook and only use `useLayoutEffect()` if you either know exactly what you are doing \(and why!).\) or if you have problems when migrating a **class component** to a **function component**, which are due to the different time of execution of the effects.

If the `useLayoutEffect()` hook is used in conjunction with **Server Side Rendering**, be careful: Neither `useEffect()` nor `useLayoutEffect()` are executed on the server side. While the `useEffect()`-hook is not a problem due to the delayed execution after the layout and paint phases, the `useLayoutEffect()`-hook may deviate from the server-side rendered markup to initial client-side rendering. React will then display a console warning. In this case the `useEffect()` hook should be used, or the component with the `useLayoutEffect()` hook should be mounted after the first paint phase:

```jsx
import React, { useState, useEffect } from 'react';
import ReactDOM from 'react-dom';

const App = () => {
  const [ mountLayoutComp, setMountLayoutComp ] = useState(false);

  useEffect(() => {
    setMountLayoutComp(true);
  }, []);

  return mountLayoutComp 
    ? <ComponentWithLayoutEffect /> 
    Zero;
};

ReactDOM.render(<App />, document.getElementById('root'));
```

In this case, the component with the `useLayoutEffect()` is not registered until the component has been mounted for the first time. This happens by querying the corresponding `mountLayoutComp` state only after the first run of the Paint phase.

## useDebugValue

```javascript
useDebugValue(value);
```

The `useDebugValue()`-Hook is solely for improving the debugging experience for developers and has no direct benefit for the **user** of an application. With it it is possible in **own hooks** to apply a label to a value within the **hook**, if this value is inspected with the **React-Devtools**:

```jsx
import React, { useDebugValue, useEffect } from "react";

const usePageTitle = (title) => {
  useDebugValue(title);
  useEffect(() => {
    document.title = title;
  }, [title]);
};

export default usePageTitle;
```

Here we implement our own small **Hook,** to change the page title in the browser. In the **Devtools** this value now appears as follows:

![Our DebugValue appears next to the name of the corresponding hook](../.gitbook/assets/usedebugvalue.png)

### Delayed formatting of the debug value

I just mentioned that the `useDebugValue()` hook has no direct **benefit** for the user. However, this does not mean that he has no direct **influence** on the user experience. This is because a slow calculation when displaying the debug value has an influence on the rendering performance of an application.

For this reason, it is possible to pass a formatting function to the hook as a second parameter. In this case, the formatting of the value is only executed when a value is actually inspected in the devtools. This has the following form:

```javascript
useDebugValue(value, (value) => formattedValue);
```

As before, the hook gets the **debug value** as the first argument. As a second argument it gets a function, which executes the formatting afterwards. This in turn receives the **value** from the **Hook** and is expected to return the formatted value.

If you want to experience for yourself how the difference becomes noticeable in an admittedly very absurd but unambiguous example, take the Fibonacci function from the `useMemo()` example, display it as a debug value once with and once without a formatting function, and observe how the time until the app is displayed changes:

```jsx
import React, { useDebugValue, useEffect } from "react";
import ReactDOM from "react-dom";

const fibonacci = (num) =>
  num <= 1 ? 1 : fibonacci(num - 1) + fibonacci(num - 2);

const useNumber = (number) => {
  useDebugValue(number, (number) => fibonacci(number));
  // without formatting function:
  // useDebugValue(fibonacci(number));
  useEffect(() => {});
  return number;
};

function App() {
  useNumber(41);
  return <p>Debug Value Formatter Example</p>;
}

ReactDOM.render(<App />, document.getElementById("root"));
```

This significantly increases the initial loading time of the app, which of course also affects the user and the user experience.

## useImperativeHandle

```javascript
useImperativeHandle(ref, createHandle, [deps])
```

To start with: This hook gave me some grey hair, because it was really hard for me to construct an application where the use of `useImperativeHandle()` is the solution. When I wanted to vent some of my frustration on Twitter, Dan Abramov, core developer in the React team on Facebook, also contacted me and confirmed that it was a sign that I would do everything right, as the hook was not supposed to be used at all at best and therefore intentionally has a long name. However, in this book I would like to pursue the claim to understand React and not only to know that such a hook might exist.

![Dan Abramov kindly reminds me on Twitter to do everything right](../.gitbook/assets/useimperativehandle.png)

Now I've actually been busy with this hook for a long time and have to say: Yes, if you use the `useImperativeHandle()`-Hook, you should see if everything really makes sense and if there is not another way that does not include the use of this hook. The official docu also talks about not using the hook at this point, because, as the name suggests, it is designed for imperative code and thus opposes the declarative style prevailing in React. Sometimes this is necessary, especially when working with classes and objects, which is often the case with external libraries.

Here is a cautious example illustrating the use of the **Hook**. In the example we create our own `FancyForm` form component. It outputs its **child elements** and provides some methods that can be called in the consuming parent component. For example, we implement a method `focusFirstInput` to focus the first input field within our `FancyForm` form. In addition, we extend the form with our own method `getFormValues`, which returns the currently entered data as JSON. Furthermore, we make it possible to send and reset the form programmatically by assigning the methods `reset()` and `submit()` from the HTML `<form>` element as imperative methods to the forwarded **ForwardRef**:

```jsx
import React, { useImperativeHandle, useEffect, useRef } from "react";
import ReactDOM from "react-dom";

const FancyForm = React.forwardRef((props, forwardedRef) => {
  const formRef = useRef();

  useImperativeHandle(forwardedRef, () => ({
    focusFirstInput: () => {
      (formRef.current.querySelector("input") || {}).focus();
    },
    getFormValues: () => {
      return Array.from(new FormData(formRef.current)).reduce(
        (acc, [value, name]) => {
          acc[name] = value;
          return acc;
        },
        {}
      );
    },
    reset: () => formRef.current.reset(),
    submit: () => formRef.current.submit(),
  }), []);

  return <form ref={formRef}>{props.children}</form>;
});

const App = () => {
  const formRef = useRef();

  useEffect(() => {
    formRef.current.focusFirstInput();
  }, []);

  const submit = (e) => {
    e.preventDefault();
    console.log(formRef.current.getFormValues());
  };

  return (
    <FancyForm ref={formRef}>
      <p>
        <input type="text" name="name" placeholder="name" />
      </p>
      <p>
        <input type="email" name="email" placeholder="email" />
      </p>
      <input type="submit" onClick={submit} />
    </FancyForm>
  );
};

ReactDOM.render(<App />, document.getElementById("root"));
```
