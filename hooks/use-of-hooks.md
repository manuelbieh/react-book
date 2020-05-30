# Use of Hooks

React currently offers **10** Hooks for us to use. Of these 10, **3** of these are fundamental or **basic**, the remaining 7 are **additional** — according to the official React documentation. It is a useful distinction though, as the **3 basic Hooks** `useState()`, `useEffect()` and `useContext()` will be sufficient in most cases.

The remaining **additional Hooks** will help us to cover edge cases or to deal with certain optimizations at a later date. In this chapter however, we will focus on "simple" Hooks and how we can now implement functionality in **Function components** that was previously reserved for **Class components.**

### **State with useState\(\)**

Let us have a look at how we previously accessed and modified state:

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
        <p>Counter: {this.state.value}</p>
        <button onClick={() => this.setState((state) => ({ value: state.value + 1 }))}>
          +1
        </button>
      </div>
    );
  }
}

ReactDOM.render(<Counter />, document.getElementById('root'));
```

Here we have implemented a simple counter which keeps track of how many times we press the **+1** button. While it is not the most creative of examples, it demonstrates quite nicely how we can use Hooks to simplify code. In this example, we read the current value with `this.state.value` and increment the counter by a button that is placed below. Once the button is clicked, `this.setState()` is called and we set the new `value` to the previous value which we increment by 1.

But let's have a look how the same functionality can be implemented using `useState()` in a **Function component**:

```jsx
import React from 'react';
import ReactDOM from 'react-dom';

const Counter = () => {
  const [ value, setValue ] = React.useState(0);
  return (
    <div>
      <p>Counter: {value}</p>
      <button onClick={() => setValue(value + 1)}>
        +1
      </button>
    </div>
  );
}

ReactDOM.render(<Counter />, document.getElementById('root'));
```

Watch out: we do not use a `state` object anymore and have also not used the `this` keyword to access our state. Instead, a simple `useState()` Hook is called. It works by taking in an **initial value** \(`0` in this case\) and returning a tuple — an array with a set number of values. In the case of the `useState()` Hook, this tuple consists of the value of state, and a setter function which we can use to modify this value.

In order to directly access this state value and the associated setter function, we are making use of **ES2015 Array destructuring**, meaning the first value in the array will give us access to `value` while the second will be written into `setValue`. While you can certainly use your own names for these values, conventions have emerged to give short and precise names to `state` values and using verbs such as `set`, `change` or `update` in conjunction with the name of the `state` value to indicate what it is doing. The syntax itself is a much shorter equivalent of the following ES5 code:

```javascript
var state = React.useState(0);
var value = state[0];
var setValue = state[1];
```

We now directly access `value` instead of `this.state.value` inside of the `Counter` component. Moreover, we can set a new value with `setValue(value + 1)` instead of the slightly wordier `this.setState((state) => ({ value: state + 1}))`.

It's time to celebrate: We've just created our first **stateful Function Component!**

{% hint style="info" %}
We just used our very first **Hook**: `useState()` by calling `React.useState()`. We can also import **Hooks** directly from the `react` package.

```jsx
import React, { useEffect, useState } from 'react';
```

This way, we can easily use `useState()` instead of having to write out `React.useState()` every single time.
{% endhint %}

Components are not limited to use each type of Hook only once. Thus, we can easily implement two counters which we can increment and manage independently by creating their own states:

```jsx
import React from 'react';
import ReactDOM from 'react-dom';

const Counter = () => {
  const [ firstValue, setFirstValue ] = React.useState(0);
  const [ secondValue, setSecondValue ] = React.useState(0);
  return (
    <div>
      <p>Count 1: {firstValue}</p>
      <p>Count 2: {secondValue}</p>
      <button onClick={() => setFirstValue(firstValue + 1)}>+1</button>
      <button onClick={() => setSecondValue(secondValue + 1)}>+1</button>
    </div>
  );
}

ReactDOM.render(<Counter />, document.getElementById('root'));
```

If, at this point, you are wondering how you would manage very complex state, I would urge you to "hold that thought" as we will learn about the `useReducer()` Hook in a later chapter. For now, we will focus on the three **basic** Hooks.

### Side effects with useEffect\(\) 

The name of the `useEffect()` **Hook** derives from its intended usage: for **Side Effects**. In this case, we mean loading data via an API, registering global events or manipulating DOM elements. The `useEffect()` Hook includes functionality that was previously found in the `componentDidMount()`, `componentDidUpdate()` and `componentWillUnmount()` lifecycle methods. 

If you are wondering whether all these lifecycle methods have now been replaced and been combined in to a single Hook, I can assure you: you have read correctly. Instead of using _three_ methods, you only need to use _a single_ **Hook** which takes effect in similar places where the class component methods were previously used. The trick is to use particular function parameters and return values which are intended for the `useEffect()` Hook.

In order to use the `useEffect()` Hook, you pass the `useEffect()` function another function as its first parameter. This function, which we will call the **Effect function** for now, is invoked **after** each rendering of the component and "replaces" the `componentDidMount()` part of class components.

As this **Effect function** is called after **each** render of the component, it is also called after the **first** render. This equates to what has been typically covered in the `componentDidMount()` lifecycle method.

Moreover, the _Effect function_ can optionally return another function. Let's call this function a **Cleanup function**. This function is invoked during the **unmounting** of the component, which roughly equates to the `componentWillUnmount()` class component method.

But be careful: while this sounds similar, the `useEffect()` Hook works a little differently compared to class components' lifecycle methods. Our _cleanup function_ is not only called during the **Unmounting** of the component but also **before each new execution** of the _Effect function_.

The second parameter of the `useEffect()` Hook is the **dependency array**. The values of this array indicate the values upon which the execution of the **Effect function** depends on. If a **dependency array** is passed, the Hook is only invoked initially and then only when at least one of the values in the **dependency array** has changed.

If you explicitly try to replicate behavior previously covered by `componentDidMount()`, you can pass an empty array as your second parameter. React then only executes the **Effect function** on initial render and only calls a cleanup function again during **unmount**.

This probably sounds all very complex and theoretical now, especially if you are new to Hooks and do not quite understand how they work. Let us look at an example to clear things up a little bit:

```jsx
import React, { useEffect, useState } from 'react';
import ReactDOM from 'react-dom';

const defaultTitle = 'React with Hooks';

const Counter = () => {
  const [ value, setValue ] = useState(0);
  
  useEffect(() => {
    // `document.title` is set with each change (didMount/didUpdate).
    // Given the `value` has changed
    document.title = `The button has been clicked ${value} times.`;

    // Here we're returning our "Cleanup function" which resets the title to the default
    // before each update
    return () => {
      document.title = defaultTitle;
    }
  
  // Lastly, our dependency array. This way the Effect function is only invoked 
  // when the `value` has actually changed.
  }, [value]);
  
  return (
    <div>
      <p>Counter: {value}</p>
      <button onClick={() => setValue(value + 1)}>
        +1
      </button>
    </div>
  );
}

ReactDOM.render(<Counter />, document.getElementById('root'));
```

As the Hook is always found _inside_ of the function, it has complete access to **props** and **state** \(as has been the case already in the lifecycle methods of class components\). In this case, the state of the Function component is another **Hook** that is implemented using the `useState()` Hook.

By using the `useEffect()` **Hook**, we can dramatically reduce the complexity of this component, as it does not have to execute many similar pieces of code through different functions. Instead it can deal with all the associated lifecycle methods with just a single function, the **Hook**.

To compare this with class component code, I have prepared a little example which implements the same functionality as the `useEffect()` Hook:

```jsx
import React from "react";
import ReactDOM from "react-dom";

const defaultTitle = "React with Hooks";

class Counter extends React.Component {
  state = {
    value: 0,
  };

  componentDidMount() {
    document.title = `The button has been clicked ${this.state.value} times`;
  }

  componentDidUpdate(prevProps, prevState) {
    if (prevState.value !== this.state.value) {
      document.title = `The button has been clicked ${this.state.value} times`;
    }
  }

  componentWillUnmount() {
    document.title = defaultTitle;
  }

  render() {
    return (
      <div>
        <p>Counter: {this.state.value}</p>
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

Of course you can debate whether it would be useful to extract the call to change `document.title` into its own class method such as `setDocumentTitle()` however this is not really adding much to our discussion as they do not change anything about the complexity at hand.

Even then, we would still need to call the same \(and now abstracted\) function in two places: `componentDidMount()` and `componentDidUpdate()`. Additionally, we would have to add another class method which further bloats our class component and only reduces duplication by adding more abstraction layers.

### Accessing Context with useContext\(\)

The third and final basic Hook is `useContext()`. It allows us to consume data from a Context Provider without having to define a Provider component with a function as a child.

`useContext()` is passed a context object, which you can create by using `React.createContext()`. It will then return the value of the next higher up provider in the component hierarchy. If the value in the context is changed within the provider, the `useContext()` Hook will trigger a re-render with the updated data from the provider. And that just about sums up the functionality of the `useContext()` Hook.

In practice, this translates to something like the following example:

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

The `ContextExample` component is receiving its data from the pseudo-account data provider: the`AccountContext` provider. This works without having to wrap an `AccountContext.Consumer` component around `ContextExample`. It does not only save us multiple lines of code in the component itself, but also leads to a much better debugging experience as the component tree is not as deeply nested as it would be otherwise.

However, this simplification is entirely optional. If you prefer to keep using the well-known Consumer component to access data from a provider, that is completely fine.

