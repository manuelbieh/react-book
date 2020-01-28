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



