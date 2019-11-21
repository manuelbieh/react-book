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

