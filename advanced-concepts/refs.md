# Refs

**References** \(i.e., **References**\) in React allow direct access to DOM elements generated during the rendering process. Although this is not necessary in React in most cases, there are some situations where it is not possible to avoid directly accessing the DOM elements. For example, when the position or size of an element is needed to show a tooltip based on it or to focus an input field with `.focus()` after loading a component.

Here React has evolved over time and so historically has given us various ways to create such **Refs**. The **Refs**, no matter in which form, are always defined by the `ref` prop of a DOM element in the **JSX** or `createElement()` call.

But a warning right away: Even though React allows direct access to DOM elements by using **Refs**, this should only be done if the declarative way, i.e. neurendering components with changed props and state, does not help. All manipulation of attributes or attribute values, adding or removing classes or event listers or changing other properties like `aria-hidden` should **always be done declaratively** via the props, the state, JSX and corresponding renderings!

However, there are a few cases where the use of a ref is useful or even necessary. That includes:

* Setting a focus on input elements \(`input.focus()`\) or calling other methods like `.play()` or `.pause()` on `video` and `audio` elements
* The triggering of imperative animations
* Reading properties of an element in the DOM \(e.g. via `.getBoundingClientRect()`\)

### String Refs

The simplest and oldest variants are the so-called **String Refs**. In the meantime, it is not recommended to use them, as they can impair performance and may be removed in the future. For the sake of completeness, I'd like to mention them anyway, as they're still part of the official API and you'll occasionally encounter them working with React, especially if you're dealing with legacy code.

To define a **String Ref**, give a prop with the name `ref` to a DOM element and assign a **String** as value to this prop. The corresponding DOM element is then **accessible within the component** in the instance property `this.ref`.

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

In the above example, we can access the surrounding div in the `componentDidMount()` lifecycle hook via `this.refs.wrapper` and the input field via `this.refs.username`. With `this.refs.username.focus()` you can focus on the latter.

### Callback Refs

An alternative to **String Refs** are the so-called **Callback Refs**. These allow more flexibility, but are of course more cumbersome to implement, since you have to take care of their handling yourself. With **Callback Refs** it is possible to pass them on to **Kind components** in order to access DOM elements within them.

**Callback Refs** are, as the name suggests, defined in **Callback-Form** and get **Mounting** as only parameter the DOM-Element or when applying to a React-Component the instance of the DOM-Element, when **Unmounting** the callback is called again, but then with `null` as parameter. 

What you do with them is up to you. However, a common approach is to store the reference to this DOM element as an instance property so that it can be accessed from anywhere within the component.

Applied to the example above, it would look like this:

```jsx
import React from 'react';
import ReactDOM from 'react-dom';

class ComponentWithCallbackRef extends React.Component {
  usernameEl = zero;

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

**Callback Refs** can also be used in child components and these **Refs** can then be accessed from within the parent component. For this you pass the callback function of the child component in a separate prop, which must not be called `ref`, because this is a reserved name:

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

  usernameEl = zero;

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

If you would name the prop `ref` here, `<UsernameInput ref={this.setUsernameRef} />`, you would get a reference to the `UsernameInput`-**instance** instead of its input element. With **Function Components** `UsernameInput` would even be `null`, because SFCs are not instantiated!

However, the `forwardRef()` method should be used for forwarding refs in child components if possible. This is described at the end of this chapter.

### Refs about createRef\(\)

New in React 16.3. is the top-level method `React.createRef()`. It is a bit similar to the **Callback Refs** in the way it is used, but with small differences. So you have to take care of the handling yourself. Due to their similarity to **Callback Refs**, it is common practice to assign the **Refs** to an **Instance property**.

Instead of passing an almost identical method in the form `(el) => { this.property = el }` each time, you create the reference when instantiating the component and pass it to the `ref` prop of the respective element.

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

In principle very similar to the **Callback Refs**, but with another decisive difference: You can access the corresponding reference here via `this.usernameEl.current`. 

The reference to the element is not stored in the instance property, which assigns the ref to it, but in its `.current` property. Otherwise their behavior is comparable to the Callback Refs. You can also pass these on to child components via their props and then access the respective DOM element from the parent component.

In the direct comparison here again the **Callback Ref**:

```jsx
class MyComponent extends React.Component {
  usernameEl = zero;
  
  render () {
    return (
      <input ref={(el) => { this.usernameEl = el; }} />
    )
  }
}
```

Access to the element via:  `this.usernameEl` 

And alternatively the reference created with **`React.createRef()`**:

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

Access to the element via: `this.usernameEl.current`.

### Forwarding of Refs

The so-called **Ref forwarding**, i.e. the _forwarding of Refs_ \ (references to a component or a DOM element\) makes it possible to pass a reference through a component to a child component. This is not necessary in the vast majority of cases, but can sometimes become important, especially when creating a reusable component library. 

A **Ref** is forwarded via the method `React.forwardRef()`. It is given a function as a parameter to which it passes props and the **Ref**.

Sounds awfully awkward again, so let's take a look at a code example. In the following example, we first implement our own input component without ForwardRef:

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

In the `componentDidMount()` Lifecycle method we would not have access to our input field from the `UsernameField` component. Instead, the instance of the component itself would be **Ref**. However, since `UsernameField` is a **Function Component**, not even an instance of the component exists. So the `console.log` output would be at the place: `{ current: null }` - ugly, we want to get access to the `input` element to focus it for example.

It is sufficient to have the `UsernameField` component enclosed by a `React.forwardRef()` call. In the example above, we change the component as follows:

```jsx
const UsernameField = React.forwardRef((props, ref) => (
  <div className="myInput">
    <input ref={ref} {...props} />
  </div>
));
```

Here we tell React to please forward the `ref` prop on the `UsernameField` in our `App` component to the component. The **Ref** is then passed as the second parameter of the function and can be passed on to any element in the DOM or to another component. 

If the **Ref** is passed on to another component deeper in the tree, it should be noted that the same restrictions apply here: The corresponding component must either be a class component - then the reference would point to the instance of the class - or the component must in turn make a redirection via `forwardRef()`.

#### Caution with Higher Order Components!

Caution is advised when implementing **Higher Order Components**. If it is unclear whether forwarded refs should be accessed in them, they themselves must be enclosed by a `forwardRef()` call.

Let's extend our example from above and assume we want to create a **HOC** to display form elements in a certain uniform style. Therefore we create the **HOC** `withInputStyles`. This can and will enclose `input` elements and in these it should not be excluded that we assign them a ref. 

The whole process is a bit complicated and it was very hard for me to put it into words that would have been easy to understand, so I'd like to let the commentated code speak instead. Once the principle of **Higher Order Components** and **forwardRefs** is clear, the example should be sufficient to understand the implementation. And even if not, a little consolation in advance: This is an application that should be so rare in practice that one will rarely need it in a real application. 

For the sake of completeness, however, I would like to mention it here. So here's the example:

```jsx
import React from "react";
import ReactDOM from "react-dom";

const withInputStyles = (InputComponent) => {
  class WithInputStyles extends React.Component {
    render() {
      // We get the forwarded ref from the props of the component
      const { forwardedRef, ...props } = this.props;
      
      // ... and use it as a ref for the HOC-encapsulated 
      // Component on:
      return (
        <InputComponent
          {...props}
          ref={forwardedRef}
          style={{ border: "2px solid black", padding: 8 }}
        />
      );
    }
  }
  
  // We return a ForwardRef from the HOC
  return React.forwardRef((props, ref) => (
    // We pass the ref as a temporary prop `forwardedRef`.
    <WithInputStyles {...props} forwardedRef={ref} />
  ));
};

// Here we forward the ref of our component to the input using 
// of an ordinary React.forwardRef() call
const UsernameField = React.forwardRef((props, ref) => (
  <input ref={ref} {...props} />
));

// Here we connect the HOC with our UsernameField component
const StyledUsername = withInputStyles(UsernameField);

class App extends React.Component {
  // This is where we usually create the ref that we access later.
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