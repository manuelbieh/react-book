# Use of Hooks

React currently offers **10** Hooks for us to use. Of these 10, **3** of these are the fundamental or **basic**, the remaining 7 are **additional**  - according to the official React documentation. It's a useful distinction as the **3 Basic Hooks** `useState()`, `useEffect()` and `useContext()` will be sufficient in most cases.

The remaining **additional Hooks** will help us to cover edge cases or to deal with certain optimizations at a later date. In this chapter however, we will focus on "simple" Hooks and how we can now implement functionality in **Function** **components** that was previously reserved for **Class components.**

### **State with useState\(\)**

Let us have a look at how we previously accessed state and modified it:

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
        <p>Zählerwert: {this.state.value}</p>
        <button onClick={() => this.setState((state) => ({ value: state.value + 1 }))}>
          +1
        </button>
      </div>
    );
  }
}

ReactDOM.render(<Counter />, document.getElementById('root'));
```

We've implemented a simple counter in this example which keeps track of how many times we've pressed the **+1** button. While it is not the most creative of examples, it demonstrates quite nicely how we can use Hooks to simplify the above code. In this example, we read the current value with `this.state.value` and increment the counter by a button that is placed below. Once the button is clicked, `this.setState()` is called and we set the new `value` to the previous value which we increment by 1.

But let's have a look how the same functionality could be implemented using `useState()` in a **Function component:**

```jsx
import React from 'react';
import ReactDOM from 'react-dom';

const Counter = () => {
  const [ value, setValue ] = React.useState(0);
  return (
    <div>
      <p>Zählerwert: {value}</p>
      <button onClick={() => setValue(value + 1)}>
        +1
      </button>
    </div>
  );
}

ReactDOM.render(<Counter />, document.getElementById('root'));
```

Notice: we do not use a `state` object anymore and also have not introduced any `this` in order to access our state. Instead, a simple `useState()` hook is called. It works by taking in an **initial value** \(`0` in this case\) and returning a tuple - an array with the same number of values. In the case of the `useState()` hook, this tuple consists of the actual state value and a setter function which we can use to modify this value.

In order to directly access this value and the associated setter function, we are making use of **ES2015 Array destructuring** meaning the first value in the array will give us access to `value` while the second will be written into `setValue`. While you can certainly use your own naming, conventions have emerged to give short and precise names to `state` values and using verbs such as `set`, `change` or `update` in conjunction with the name of the `state` value to indicate what it is doing. The syntax itself is a much shorter equivalent of the following ES5 code:

```javascript
var state = React.useState(0);
var value = state[0];
var setValue = state[1];
```

We now directly access `value` instead of `this.state.value` inside of the `Counter` component.  Moreover, we can set a new value with `setValue(value + 1)` instead of the slightly wordier `this.setState((state) => ({ value: state + 1}))`.

Time to celebrate: We've just created our first **stateful Function Component!**

{% hint style="info" %}
We just used our very first **Hook**: `useState()b`y calling `React.useState()`. We can also import **Hooks** directly from the `react` package.

```jsx
import React, { useEffect, useState } from 'react';
```

This way, we can easily use `useState()` instead of having to write out `React.useState()` every single time.
{% endhint %}

Components are not limited to only use each type of Hook only once. Thus, we could easily implement two counters which we can increment and manage independently by creating their own states:

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

If you are wondering at this point how you would manage very complex state, I would urge you to "hold that thought" as we will learn about the `useReducer()` Hook in a later chapter. For now, we will focus on the three **basic** Hooks.

### Side effects with useEffect\(\) 

