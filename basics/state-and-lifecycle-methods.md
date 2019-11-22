# State and Lifecycle Methods

We've mentioned it in places and now we are finally going to look at it in greater detail - **state** and the so-called **lifecycle methods**.

As mentioned in the previous chapter, components can hold their own **state**, manage and change it. But: **If the state within a component changes, it always triggers a re-render of the component**. This behavior _can_ be avoided by opting for `PureComponent` as we've learned in the previous chapter which might be useful in some cases. The foundational logic remains though. A state change leads to a re-render of a component and all of its children, except from those cases in which the children are actually a `PureComponent` or its call is surrounded by a `React.memo()` call.

Relying on state to change our interface is extremely useful. It means that we do not need to rely on manually calling `ReactDOM.render()` to update our interface and that components manage their re-renders independently.

State is tightly connected to the so-called **lifecycle methods.** These comprise a number of optional methods which can be called at different times and for different uses cases in **class components**. For example, there are **lifecycle methods** for when a component is first mounted, if a component receives new props or if the state within a component changes.

Since **React 16.8.0**, **function components** can also manage their own state through the use of **hooks**. Hooks can also react to certain lifecycle events but we will not describe them in detail at this point. This chapter will focus primarily on class components and their associated lifecycle methods. Hooks on the other hand will receive their own dedicated chapter later in the book as they can still be considered a relatively new and extensive topic. 

## A first stateful component

**State** inside a class component can be accessed via the instance property `this.state`. It is encapsulated to the **component** and neither parent or child components can access it.

To define a component's initial state we can choose three ways. Two of these are relatively simple, the third is a little more advanced. We're going to cover the latter when we learn about the **lifecycle method** `getDerivedStateFromProps()`. 

**Initial state** can be defined by setting `this.state`. This can either be done via the constructor of a **class component**:

```javascript
class MyComponent extends React.Component {
  constructor(props) {
    this.state = {
      counter: props.counter,
    };
  }
  render() {
    // ...
  }
}
```

... or by defining state as a **ES2017 class property**. This is much shorter but still requires the **Babel plugin** `@babel/plugin-proposal-class-properties` \(pre-Babel 7: `babel-plugin-transform-class-properties`\):

```javascript
class MyComponent extends React.Component {
  state = {
    counter: this.props.counter,
  };
  render() {
    // ...
  }
}
```

**Class Property Syntax** is supported out-of-the-box by **Create React App**. As ****most projects today rely completely or in part on the CRA setup, this syntax is already widely used and common to see in most projects. If you encounter a project where this is not the case, I'd urge you to install and use this Babel plugin as it reduces the length of your code and is easily set up.

Once **state** is defined, we can **read** its value via `this.state`. While it is possible to mutate `this.state` directly, it is actively discouraged.

## Changing state with this.setState\(\)

To change state within components, React offers a new method for use inside of **class components**:

```javascript
 this.setState(updatedState)
```

Whenever state is supposed to change within the component, `this.setState()` should be used to achieve this. By calling `this.setState()` React knows to execute **lifecycle methods** \(for example `componentDidUpdate()`\) and thus **re-render** the component. If we were to change state directly, for example by using `this.state.counter = 1;`, nothing would happen initially as the render process would not be triggered. React would not know about its change in state.

The  `this.setState()` method might look a little complex at the state. But this is also due to the fact that old state is not simply replaced by new state and triggering a re-render, but because many other things are happening behind the scenes. But let's take it step by step.

The function itself can take **two different types of arguments**. The first being an **object** containing the new or updated state properties and the second being an **updater function** which again returns an object or `null` if nothing should change. If you happen to have the same property name in your object it will be **overridden** while all other properties will **remain the same**. To reset properties in state, their values need to be explicitly set to `null` or `undefined`. The new state that is being passed is thus **never replaced** but **merged together** with the existing state. 

Let us have another look at our example from above in which we defined state with a `counter` property whose initial value was `0`. Assuming that we want to change this state and add another `date` property to pass the current date, we can construct the new object like so:

```javascript
this.setState({
  date: new Date(),
});
```

If an **updater function** was used instead, our function call would look like this:

```javascript
this.setState(() => {
  return {
    date: new Date(),
  };
});
```

... or even shorter:

```javascript
this.setState(() => ({
  date: new Date(),
});
```

Finally, our component contains the new state:

```javascript
{
  counter: 0,
  date: new Date(),
}
```

To guarantee that only the most current state is always accessed, an **updater function** should be used which contains the current state as a parameter. Many developers have already made the mistake of trying to access `this.state` right after calling `setState()` only to realize that their state is still old.

{% hint style="danger" %}
React "collects" many sequential `setState()`calls und **does not immediately invoke them** to avoid an unnecessary amount of re-renders. Sequential `setState()`calls which are called right after each other are executed as a **batch process**. This is important to keep in mind as we cannot reliably access new state with `this.state` just after a `setState()`call.
{% endhint %}

Assume that we would like to increment the `state` counter three times in quick succession. Intuitively, we might feel compelled to write the following code:

```javascript
this.setState({ counter: this.state.counter + 1 });
this.setState({ counter: this.state.counter + 1 });
this.setState({ counter: this.state.counter + 1 });
```

If the first initial state was `0`, what will the new state be? What do you think? `3`? Nope. It's `1`! But why? React uses its **batching mechanism** to cluster these `setState()` calls together in order to avoid a jarring user interface which continually updates. The above code snippet could be translated into the following code if it was written in **functions**.

```javascript
this.state = Object.assign(
  this.state, 
  { counter: this.state.counter + 1 },
  { counter: this.state.counter + 1 },
  { counter: this.state.counter + 1 },
);
```

The `counter` property overrides itself after each batch update but always uses `this.state.counter` as its base reference for incrementing by 1. After all state calls having executed, React calls the `render()` method again.

If an **updater function** is used instead, the current state is passed as a parameter and access is given to the state when the function is actually called:

```javascript
this.setState((state) => ({ counter: state.counter + 1 });
this.setState((state) => ({ counter: state.counter + 1 });
this.setState((state) => ({ counter: state.counter + 1 });
```

This example uses an **updater function** and always updated the current state value. The value of `this.state.counter` is `3` in this case, as we expected, because the `state` parameter which we provide to the updater function is accessing the current state. While this is possible in theory, it is not exactly recommended. Values should be collected first and then batch processed in a single `setState()` call. This avoids unnecessary re-renders with potentially outdated state.

In some situations it might be necessary to access the value of the modified state. Luckily, React offers the possibility to provide a second parameter in the `setState()` call. This parameter is a callback function which is called **after** the state has updated so that we can safely access the state we have just modified.

```javascript
this.setState(
  {
    time: new Date().toLocaleTimeString() 
  }, 
  () => {
    console.log('Neue Zeit:', this.state.time);
  }
);
```

## Lifecycle Methods

React offers a number of so-called **lifecycle methods** that can be called at different times in the **component's lifecycle**. These can be implemented in React **class components**.

The lifecycle of a component starts as soon as it is **instantiated or mounted**, so when it is found in the `render()` method of a parent component being part of the returned component tree. The component's lifecycle ends if it is removed from the tree of renderable components. Additionally, there are **lifecycle methods** that react to **updates** or ****errors as well as being "unmounted".

### **Overview of lifecycle methods**

The following shall be a comprehensive overview of the **lifecycle methods** that are currently available, clustered by the phases in which they occur in a component's lifecycle.

**Mount phase**

These methods are only called **once** when the component is first rendered \(or put simply added to the DOM\).

* `constructor(props)`
* `static getDerivedStateFromProps(nextProps, prevState)`
* `componentWillMount(nextProps, nextState)` \(deprecated in React 17\)
* `render()`
* `componentDidMount()`

**Update phase**

If new props are being passed to a component or if state changes within the component, these update methods will be called. Alternatively, an explicit `forceUpdate()` method could be invoked.

* `componentWillReceiveProps(nextProps)` \(deprecated in React 17\)
* `static getDerivedStateFromProps(nextProps, prevState)`
* `shouldComponentUpdate(nextProps, nextState)`
* `componentWillUpdate(nextProps, nextState)` \(deprecated in React 17\)
* `render()`
* `getSnapshotBeforeUpdate(prevProps, prevState)`
* `componentDidUpdate(prevProps, prevState, snapshot)`

**Unmount phase**

This phase only has one matching method which is called as soon as the component is removed from the DOM. It can be useful to tidy up event listeners and `setTimeOut()/setInterval()` calls which had been added during mounting of the component**:**

* `componentWillUnmount()`

**Error handling**

Another method to deal with errors has been added to React with React 16. This method can be called to catch errors that occur during the rendering process in a lifecycle method or in the constructor of a **child component**.

* `componentDidCatch()`

Components which implement `componentDidCatch()` are commonly called **error boundary** and help to visualise an alternative to the erronous tree. It could be a high-level component \(with regard to its position in the component hierarchy\) that displays an error page and asks the user to reload. But equally, it could also be a low level component which only renders a little error message next to a button, triggered by an erronous action attached to the button.

