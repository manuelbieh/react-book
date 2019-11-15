# Components in React

## The two types of React components

We've already implemented our first `Hello World` component in the section "Into the deep end". While this component taught us the basics of **React** and **React components**, it was a simplistic example and did not make use of everything React offers.

**Components** are relatively easy to explain: they allow us to de-construct complex user interfaces into multiple parts. Ideally, these parts are reusable, isolated and closed in itself. They can deal with any input from the outside so-called **props** \(from "properties"\) and describe what's going to be rendered on the screen by whatever they define in the `render()` method.

We can cluster components into two different versions: **function components**, which are expressed in the form of a function, and **class components**, which build upon the newly introduced ES2015 classes. Until recently the term **Stateless Functional Component** was very common as state could only managed in class components before the introduction of React Hooks. We are going to take a deep dive into **Hooks** and how state is managed from React 16.8.0 onward in a later chapter.

### Function Components

The simplest way to define a React component is in the way of a **function component**. You might have guessed, function components are just regular JavaScript functions:

```jsx
function Hello(props) {
  return <div>Hello {props.name}</div>
}
```

To be classified as a valid **React component** the function has to fulfill two criteria. It has to either return an explicit `null` \(`undefined` is not allowed\) or a valid `React.element` \(in the form of JSX\) and receives a `props` object. But even the `props` object can be optional and take in `null` as an argument.

### Class Components

The second method of defining a **React component** has already been shown to you in a previous chapter! But let's mention it once again - I am talking about **Class components**. These are based upon the ES2015 classes and extend `React.Component` or `React.PureComponent`. Don't worry if you to not know what `PureComponent` is for now, we are going to look at them soon. Class components have at least one method named `render()`:

```jsx
class Hello extends React.Component {
  render() {
    return <div>Hello {this.props.name}</div>
  }
}
```

There is one big pitfall to look out for: **Function components** take in their **props** as function arguments. **Class components** on the other hand cannot take in arguments and can only access its **props** via the instance method `this.props`. 

Both of these components will render the exact same result!

{% hint style="info" %}
One thing to keep in mind is to always name your components according to the standard. Both function and class components require their `displayName`\(so their actual component name\) to start with a **capital letter**. In between the rest of the characters, capital and lower case letters can be used interchangeably as long as the first letter is **capitalized**.

If you mistakenly name your React component something along the lines of `section`, React would interpret it as the corresponding DOM element which is of course something we would like to avoid.`Section` on the other hand would be a valid name for a component and would thus be differentiated from the regular DOM element.
{% endhint %}

I am going to consciously avoid going into detail about `state` in this chapter. State is a very complex topic and is thus getting a chapter of its own. I would advise you to work through this chapter first to understand how components work to then delve deeper into state in the next chapter.

### Special case: PureComponent

The **Pure Component** is a special form of the **Class Component**. It inherits from `React.PureComponent` and works similarly to a `React.Component` with the difference that React only renders a Pure Component if its **props** or **state** have changed compared to the previous render phase. It is often used to optimize performance.

This is achieved by performing a "shallow" comparison to identify whether the **references** are identical to the previous render. It does not matter if the **values** themselves did not change, a **Pure Component** will still re-render if the references have changed.

This sounds a little abstract. Let us look at another two examples to illustrate the point:

```jsx
const logFunction = (message) => console.log(message);

class App extends React.Component {
  render() {
    return <MyComponent logger={logFunction} />;
  }
}
```

We have defined a function called `logFunction` outside of the class in this example. `MyComponent` receives a prop called ~~`logger`~~ with a **reference** to the `logFunction`. The **identity** of the **reference** will stay the same after a re-render. A **pure component** would not re**-**render if its state did not change as the props are identical to that of the previous render.

However, in the following example a **pure component** _would_ be rendered:

```jsx
class App extends React.Component {
  render() {
    return <MyComponent logger={(message) => console.log(message)} />;
  }
}
```

While we still pass the same function to `MyComponent`, we create a new function identity with each new render. Because of this, the "shallow comparison" will no longer match between the previous and new props and will fail. A re-rendering is triggered.

The same applies to Objects and Arrays:

```jsx
<MyComponent
  logConfig={{ logLevel: 'info' }}
  logEntries={['Message 1', 'Message 2']} />  
```

This would also trigger a re-render as the `logConfig` object or array would be replaced with each new render.

### "Pure" Function Components

**Class components** are derived from the `Component` or `PureComponent` class. By defining this class we are deriving from, we choose whether the component should re-render with each new change in a component which is higher up in the hierarchy. Regular **function components** do not offer said functionality as they are merely JavaScript functions. Since **React 16.6.0** React offers a new wrapper function called `React.memo()` which also allows us to optimize our re-renders in **function components**. We wrap the call around the function in question:

```jsx
const MyComponent = React.memo((props) => {
  return <p>I only re-render if my props change</p>;
});
```

Using `React.memo()`, the **function component** works similarly to the **class component** equivalent which is derived from the `React.PureComponent`.

Curious already? Play around with the demo to deepen your understanding:

```jsx
import React from "react";
import ReactDOM from "react-dom";

class ClassComponent extends React.Component {
  render() {
    return <p>Class Component: {new Date().toISOString()}</p>;
  }
}

class PureClassComponent extends React.PureComponent {
  render() {
    return <p>Pure Class Component: {new Date().toISOString()}</p>;
  }
}

const FunctionComponent = () => {
  return <p>Function Component: {new Date().toISOString()}</p>;
};

const MemoizedFunctionComponent = React.memo(() => {
  return <p>Memoized Function Component: {new Date().toISOString()}</p>;
});

class App extends React.Component {
  state = {
    lastRender: new Date().toISOString(),
  };

  componentDidMount() {
    this.interval = setInterval(() => {
      this.setState({ lastRender: new Date().toISOString() });
    }, 200);
  }

  componentWillUnmount() {
    clearInterval(this.interval);
  }

  render() {
    return (
      <div>
        <p>App: {this.state.lastRender}</p>
        <ClassComponent />
        <PureClassComponent />
        <FunctionComponent />
        <MemoizedFunctionComponent />
      </div>
    );
  }
}

ReactDOM.render(<App />, document.getElementById("root"));
```

The `App` Component triggers a re-render every 0.2 seconds without new **props** being passed. The question of all questions is: Which components are being re-rendered and which are not?

## Component composition - multiple components in one



