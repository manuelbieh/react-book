# Use of Hooks

React currently has **10** hooks of its own, of which the official documentary **3** is called **Basic**, which means _basic_, and the remaining **7** is called **Additional**, which means _additional_. And in fact, this subdivision makes sense, because in most cases when working with hooks, you will use the 3 basic hooks `useState()`, `useEffect()` and `useContext()`.

The characteristics referred to as **Additional Hooks** are intended in particular for subsequent optimisation or to cover edge cases. So this chapter will first discuss the "simple" hooks and I would like to demonstrate how you can realize functionality with **Function Components**, for which **class components** were necessary up to now.

## State with useState\(\)

Let's first take a look at how we've accessed State in Components so far and how we've modified it.

```jsx
import React from 'react';
import ReactDOM from 'react-dom';

class Counter extends React.Component {
  state = {
    value: 0,
  };
  render() {
    return (
      <div>
        <p>Counter value: {this.state.value}</p>
        <button onClick={() => this.setState((state) => ({ value: state.value + 1 }))}>
          +1
        </button>
      </div>
    );
  }
}

ReactDOM.render(<Counter />, document.getElementById('root'));
```

Here we implement a counter that counts how often we pressed the **+1** button. Admittedly, not very creative, but demonstrates very well how hooks make life easier for us at this point. We display the current value by reading `this.state.value` and have a button underneath which we can increase the value. We call `this.setState()` and set the new `value` to the previous value, which we increase by 1.

Let's look at the identical functionality again, this time in a **Function Component** with the new `useState()` hook:

```jsx
import React from 'react';
import ReactDOM from 'react-dom';

const Counter = () => {
  const [ value, setValue ] = React.useState(0);
  return (
    <div>
      <p>Counter value: {value}</p>
      <button onClick={() => setValue(value + 1)}>
        +1
      </button>
    </div>
  );
}

ReactDOM.render(<Counter />, document.getElementById('root'));
```

Here we have no `state` object anymore and also no `this` via which we access the state. Instead we see here the call to the internal `useState()` hook. This works by passing a **initial value** \(here: `0`\) and returning a tuple, i.e. an array with the same number of values. In the case of the `useState()` hook, these are the current state and a setter function with which we can modify the value.

To directly access the value and setter function, we use the **ES2015 Array Destructuring** method. This ensures that the first value of the array is written to the variable `value`, the second value to the variable `setValue`. Both names are freely selectable, but it has quickly become common practice to give the state a short, concise name and to give the setter function the same name with a verb like `set`, `change` or `update` before it. So the syntax is an abbreviation for the following ES5 equivalent:

```javascript
var state = React.useState(0);
var value = state[0];
var setValue = state[1];
```

In JSX, which we return from the `Counter` component, we now directly access `value` instead of `this.state.value` and set the new value with a very short `setValue(value + 1)` instead of the class component with `this.setState((state) => ({ value: state + 1 }))`.

We have just created our first **Function Component** which is **stateful**!

{% hint style="info" %}
At this point we have used our first **Hook**: `useState()`. We have called the method `React.useState()`. **Hooks** can also be imported directly from the `react` package.

This is especially useful if a hook is used several times. This saves a lot of paperwork in the component:

```jsx
import React, { useEffect, useState } from 'react';
```

So we can comfortably use `useState()` directly in the component instead of having to write `React.useState()` every time.
{% endhint %}

Components are not limited to only one hook per type and so there can be any kind of hook also several times in a component. If, for example, we want to count up two counters, we do not have to manage the counter values in one counter object, but can also create several states:

```jsx
import React from 'react';
import ReactDOM from 'react-dom';

const Counter = () => {
  const [ firstValue, setFirstValue ] = React.useState(0);
  const [ secondValue, setSecondValue ] = React.useState(0);
  return (
    <div>
      <p>Counter value 1: {firstValue}</p>
      <p>Counter value 2: {secondValue}</p>
      <button onClick={() => setFirstValue(firstValue + 1)}>+1</button>
      <button onClick={() => setSecondValue(secondValue + 1)}>+1</button>
    </div>
  );
}

ReactDOM.render(<Counter />, document.getElementById('root'));
```

For working with complex states, we'll learn more about using the `useReducer()` hook later in this book. This was introduced to simplify the management of complex states.

## Side effects with useEffect\(\)

The `useEffect()`-**Hook** gets its name because it is intended to be used for **Side Effects**. So side effects like loading data via API, registering global events or manipulating DOM elements. The hook maps the functionality of the `componentDidMount()`, `componentDidUpdate()` and `componentWillUnmount()`-Lifecycle methods.

Yes, I read that right: Instead of the _three_ methods mentioned above, there is now only _a single_ **Hook** that replaces the comparable methods from class components. The trick is to use certain function parameters and return values exactly as they are intended for the `useEffect()` hook.

To use the hook, the `useEffect()` function itself passes a function as the first parameter. This function, we call it for simplicity's sake **"effect function "**, is basically executed by React **after** each rendering of the component and thus replaces `componentDidUpdate()` in class components.

Since this **effect function** is called after **every** rendering of the component, it is also called after the **first** rendering, which at this point can be equated with the `componentDidMount()` lifecycle method from the class components.

In addition, the _Effect_ function can optionally return a function itself. Let's call it the Clean-Up Function. This _clean up_ function is called when **unmounting** the component, which brings us to the next lifecycle method, namely `componentWillUnmount()`.

But be careful: here we also have the first deviation in functionality compared to class components. And so our _clean up_ function is called not only when **unmounting** the component, but also **before each new execution** of the _effect_ function.

This behavior can be actively controlled by passing an array of **Dependencies** to the `useEffect()` hook as the second parameter. These are values on which the execution of the **effect** function depends. If a **Dependency array** is passed, the hook is executed only once initially and then only again if at least one of the values in the **Dependency array** has changed.

If you want to simulate a behaviour similar to that of `componentDidMount()`, you can pass an empty array as second parameter. React then executes the effect function only when rendering for the first time and does not call a possibly defined cleanup function again until **unmounting**.

This all sounds terribly complicated now, if you are not familiar with the functionality of the hook, but can be demonstrated relatively easily with a short code example:

```jsx
import React, { useEffect, useState } from 'react';
import ReactDOM from 'react-dom';

const defaultTitle = 'React with Hooks';

const Counter = () => {
  const [ value, setValue ] = useState(0);

  useEffect(() => {
    // `document.title` is set at every change (didMount/didUpdate).
    // Provided the `value` has changed
    document.title = `The button was clicked ${value} times`;

    // Here we give back our "clean up function" which was used before every update.
    // resets the title to the default value
    return () => {
      document.title = defaultTitle;
    }

  // Last but not least, our Dependency. It only calls the effect function
  // even though the `value` has changed.
  }, [value]);

  return (
    <div>
      <p>Counter value: {value}</p>
      <button onClick={() => setValue(value + 1)}>
        +1
      </button>
    </div>
  );
}

ReactDOM.render(<Counter />, document.getElementById('root'));
```

Since the **Hook** is always _within_ the function, it has \(similar to the lifecycle methods in class components\) full access to the **Props** and the **State** of the component. Of course, the state in the **Function Component** itself is only a hook, namely the `useState()`-Hook.

By using the `useEffect()`-**Hooks** we can reduce the complexity here, since the component does not have to do many, sometimes very similar things at several places within the component, but all lifecycle events relevant for the component play in only one single function, the **Hook**.

For comparison, I would like to show here how the `useEffect()`-**Hook** shown above would be implemented with a class component:

```jsx
import React from "react";
import ReactDOM from "react-dom";

const defaultTitle = "React with Hooks";

class Counter extends React.Component {
  state = {
    value: 0,
  };

  componentDidMount() {
    document.title = `The button was clicked ${this.state.value} times`;
  }

  componentDidUpdate(prevProps, prevState) {
    if (prevState.value !== this.state.value) {
      document.title = `The button was clicked ${this.state.value} times`;
    }
  }

  componentWillUnmount() {
    document.title = defaultTitle;
  }

  render() {
    return (
      <div>
        <p>Counter value: {this.state.value}</p>
        <button
          onClick={() => {
            this.setState(state => ({ value: state.value + 1 }));
          }}
        >
          +1
        </button>
      </div>
    );
  }
}

ReactDOM.render(<Counter />, document.getElementById("root"));
```

Of course it can be discussed here that the call to change the `document.title` can be outsourced to a separate class method such as `setDocumentTitle()`, but these are details that do not really change the higher complexity of the class component in direct comparison with **Hooks**.

Even then, the same \(now abstracted\) function would still have to be called twice in two different places \(namely `componentDidMount()` and `componentDidUpdate()`\). In addition, we would have another class method, which only inflates the class further and thus resolves the duplication only at the expense of an abstraction.

## Access to context with useContext\(\)

The third basic hook in the bundle is `useContext()`. It makes it child's play to consume data from a context provider without having to use a provider component with a Function as a Child.

The `useContext()` hook gets a context created with `React.createContext()` and then returns the value of the next higher provider in the component hierarchy. If the value of the context is changed in the provider, the `useContext()` hook triggers a rendering with the updated data from the provider. This means that everything has already been said about how it works.

The Hook is actually rather simple in nature:

```jsx
import React, { useContext } from "react";
import ReactDOM from "react-dom";

const AccountContext = React.createContext({});

const ContextExample = () => {
  const accountData = useContext(AccountContext);

  return (
    <div>
      <p>Name: {accountData.name}</p>
      <p>Role: {accountData.role}</p>
    </div>
  );
};

const App = () => (
  <AccountContext.Provider value={{ name: "Manuel", role: "admin" }}>
    <ContextExample />
  </AccountContext.Provider>
);

ReactDOM.render(<App />, document.getElementById("root"));
```

Here we see how the `ContextExample` component uses data \(in this example pseudo account data\) from the `AccountContext` provider without everything having to be enclosed by an `AccountContext.Consumer` component. This not only saves a few lines of code in the component itself, but also results in a much clearer tree in the debug view, since the nesting depth is flatter.

However, this simplification is completely optional and anyone who likes can continue to use the familiar consumer component to access data from a context provider.

