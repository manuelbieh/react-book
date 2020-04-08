# Refs

**Refs** allow us to directly access DOM elements that were created during the render process. Even if this is not necessary in most cases, it can be useful in handful of situations in which it is hard not to revert to using them. Examples are calculating the position or size of an element in order to display a tooltip based on them or to focus on a form field using `.focus()` once the component has loaded.

React has evolved over the years and so have the methods dealing with **Refs**. But they all have one thing in common. Refs are always defined by the `ref` prop of a DOM element in JSX, or to be more precise `createElement()`.

But a word of warning before we proceed. Even though **Refs** exist, they should be used sparingly and only if the declarative way of re-rendering components with updated state and props does not help us. Manipulation of attributes or attribute values or adding or  removing classes, event listeners or changing properties like `aria-hidden` should always happen **declaratively** using props, state, JSX and re-rendering.

However, there are a few cases in which using Refs is acceptable or even necessary. These are:

* Setting focus on an input element \(`input.focus()`\) or calling methods such as `.play()` or `.pause()` on `video` or `audio` elements
* Invoking imperative animations
* Reading properties of elements currently in the DOM \(e.g. `.getBoundingClientRect()`\)

### String Refs

The simplest and oldest form of refs are **string refs**. Nowadays, it is not recommended to use them anymore as they can impact performance and might be deprecated in the future. I wanted to mention them for matters of completeness though as they are currently still part of the official API and as you might come across them during your work with React \(especially if you are working with legacy code\).

In order to define a **string ref**, you pass prop named `ref` to a DOM element and pass value to that prop in **string** form. The corresponding DOM element is now accessible to us using the instance method `this.ref` **inside of the component**.

```jsx
import React from 'react';
import ReactDOM from 'react-dom';

class ComponentWithStringRef extends React.Component {
  componentDidMount() {
    this.refs.username.focus();
  }
  
  render() {
    return (
      <input type="text" ref="username" name="username" />
    );
  }
}

ReactDOM.render(
  <ComponentWithStringRef />, 
  document.getElementById('app')
);
```

We can use `this.refs.username` to gain access to the input field, and then focus it by calling `this.refs.username.focus()`.

### Callback Refs

A more flexible alternative to **string refs** are the so-called **callback refs**. While being more flexible, their handling is slightly more cumbersome and needs to be managed explicitly. Using **callback refs**, it is possible to pass refs to a component's **child components** and access their corresponding DOM elements.

As the name might suggest, **callback refs** are declared using **callback form**. During **Mounting**, they only receive a single parameter: the DOM element or, if applied to a React component, its instance. We call the callback again during **Unmounting** but with `null` as a single parameter.

How you are using callback refs specifically is up to you, however it is a convention to save the reference to the DOM element as an instance property to be able to access it from anywhere inside the component.

Applying this to the above example, we end up with the following code:

```jsx
import React from 'react';
import ReactDOM from 'react-dom';

class ComponentWithCallbackRef extends React.Component {
  usernameEl = null;

  componentDidMount() {
    this.usernameEl.focus();
  }

  setUsernameEl = (el) => {
    this.usernameEl = el;
  };
      
  render() {
    return (
      <input type="text" ref={this.setUsernameEl} name="username" />
    );
  }
}

ReactDOM.render(
  <ComponentWithCallbackRef />, 
  document.getElementById('app')
);
```

**Callback refs** can also be used in child components and the **refs** can then also be accessed in their parent components. In order to do this, you pass the callback function of the child component a prop which is not allowed to be called `ref` \(as this is a reserved word\):

```jsx
import React from 'react';
import ReactDOM from 'react-dom';

class UsernameInput extends React.Component {
  render() {
    return (
      <input type="text" name="username" ref={this.props.inputRef} />
    );
  }
}

class ComponentWithRefChild extends React.Component {  
  componentDidMount() {
    this.usernameEl.focus();
  }

  usernameEl = null;

  setUsernameRef = (el) => {
    this.usernameEl = el;
  }

  render() {
    return (
      <div>
        Username:
        <UsernameInput inputRef={this.setUsernameRef} />
      </div>
    );
  }
}

ReactDOM.render(
  <ComponentWithRefChild />, 
  document.getElementById('app')
);
```

If you were to name the prop `ref`, leaving us with `<UsernameInput ref={this.setUsernameRef} />`, you would end up with a reference to the `UsernameInput` **instance** instead of its input element. If we used a **Function Component**, `UsernameInput` would have been `null` as SFC cannot be instantiated.

However, if forwarding refs to child components is something you are trying to achieve, you are better off using the `forwardRef()` method. I will explain it at the end of this chapter.

### Refs via createRef\(\)

With React 16.3, we've seen the introduction of the top-level API method `React.createRef()`. It resembles the usage of **callback refs** but differs in a few cases. As was the case with **callback refs**, you also need to take of ref handling yourself and it is still common practice to assign the **ref** to an **instance property**.

Instead of passing an almost identical method in the form of `(el) => { this.property = el }` each time, the reference is already created during the instantiation of the component which is then given to the `ref` prop of the element.

```jsx
import React from 'react';
import ReactDOM from 'react-dom';

class ComponentWithCreatedRef extends React.Component {
  usernameEl = React.createRef();

  componentDidMount() {
    this.usernameEl.current.focus();
  }

  render() {
    return (
      <input type="text" name="username" ref={this.usernameEl} />
    );
  }
}

ReactDOM.render(
  <ComponentWithCreatedRef />, 
  document.getElementById('app')
);
```

While this looks very similar to **callback refs**, there is an apparent difference: you access the reference via `this.usernameEl.current`.

The reference to the element is not saved in the instance property which you assign the ref to, but in its `.current()` property. Apart from this, the behavior of `createRef()` is similar to that of callback refs. They can still be passed to child components via props and the DOM element can be accessed in the parent component.

To have a direct comparison — this was the use case with **callback refs**:

```jsx
class MyComponent extends React.Component {
  usernameEl = null;
  
  render () {
    return (
      <input ref={(el) => { this.usernameEl = el; }} />
    )
  }
}
```

And this is the reference created with `React.createRef()`:

```jsx
class MyComponent extends React.Component {
  usernameEl = React.createRef();
  
  render () {
    return (
      <input ref={this.usernameEl} />
    )
  }
}
```

We're accessing the element via `this.usernameEl.current`.

### Ref forwarding

**Ref forwarding** \(references to a component or a DOM element\) enables passing a reference through a component to a child component. In most cases, this will not be necessary but it can become of interest if you are creating reusable component libraries.

A **ref** is forwarded via `React.forwardRef()` and is passed a function as a parameter during this process. The ref itself passes props as well as well as the **ref** to it.

This sounds incredibly cumbersome, so let us look at an example instead. Let us first implement an Input component without a forwardRef:

```jsx
import React from "react";
import ReactDOM from "react-dom";

const UsernameField = (props) => (
  <div className="myInput">
    <input ref={props.ref} {...props} />
  </div>
);

class App extends React.Component {
  usernameEl = React.createRef();

  componentDidMount() {
    console.log(this.usernameEl);
  }

  render() {
    return (
      <div className="App">
        <UsernameField ref={this.usernameEl} />
      </div>
    );
  }
}

ReactDOM.render(<App />, document.getElementById("root"));
```

In this case, `componentDidMount()` does not have access to our input field from the `UsernameField` component. Instead, the instance of the component would actually be the **ref** itself. As `UsernameField` is a **Function Component**, we do not even have an instance of the component. The associated`console.log` result would be: `{ current: null }` - not exactly ideal if we want to gain access to the input element in order to focus on it.

It is sufficient in this case to wrap the `UsernameField` component with a `React.forwardRef()` call. We can now amend the code of the `UsernameField` in the above example:

```jsx
const UsernameField = React.forwardRef((props, ref) => (
  <div className="myInput">
    <input ref={ref} {...props} />
  </div>
));
```

We have thus informed React that it should forward the `ref` prop on the `UsernameField` in our `App` component to the component. It is passed the **ref**  as a second parameter of the function and can pass this ref to any other element in the DOM or to another component.

If the **ref** is passed to a component lower in the component tree, the same restrictions apply: The relevant component either has to be a class component  \(then the reference would point to the instance of the class\) or the component needs to conduct ref forwarding via `forwardRef()`.

#### Be careful with Higher Order Components!

Careful consideration and care needs to be taken with **refs** while implementing **Higher Order Components**. If there is doubt whether these HOC should be able to access forwarded Refs, they also need to be wrapped by a `forwardRef()` call.

Let's extend our example from above and assume that we want to create a **HOC** to show all of our form elements in a unified layout. Thus, a **HOC** `withInputStyles` is created which can wrap `input` elements in which it is possible to assign them a ref. 

This procedure is a little complicated and I have found it a little complicated to explain this properly in written form without confusing the reader further. Instead, I invite you to inspect the following code closely and read its associated comments. As soon the idea of **Higher Order Components**  and **forwardRefs** begins to make sense, the example should be sufficient to understand the actual explanation. And if it is not clear from the get go: this case is so incredibly rare that you will not need it much in practice.

So let's look at the example of **forwardRef** and **Higher Order Components**: 

```jsx
import React from "react";
import ReactDOM from "react-dom";

const withInputStyles = (InputComponent) => {
  class WithInputStyles extends React.Component {
    render() {
      // We access the forwarded Ref in the props of the component 
      const { forwardedRef, ...props } = this.props;
      
      // ... and use it as the ref for the component wrapped by the HOC
      return (
        <InputComponent
          {...props}
          ref={forwardedRef}
          style={{ border: "2px solid black", padding: 8 }}
        />
      );
    }
  }
  
  // We return a forwardRef from the HOC
  return React.forwardRef((props, ref) => (
    // We pass the ref as the temporary prop "forwardRef"
    <WithInputStyles {...props} forwardedRef={ref} />
  ));
};

// Here, we pass the ref of our component to a regular React.forwardRef() call
const UsernameField = React.forwardRef((props, ref) => (
  <input ref={ref} {...props} />
));

// Here, we connect the HOC to the UsernameField component
const StyledUsername = withInputStyles(UsernameField);

class App extends React.Component {
  // Here we are creating the Ref as we usually do and to access it later
  usernameEl = React.createRef();

  componentDidMount() {
    this.usernameEl.current.focus();
  }

  render() {
    return (
      <div>
        <StyledUsername ref={this.usernameEl} />
      </div>
    );
  }
}
```

