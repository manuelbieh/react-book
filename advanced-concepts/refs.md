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

We can access the surrounding div using `this.ref.wrapper` and use `this.refs.username` to gain access to the input field. The latter can then be focused by `this.refs.username.focus()`.

### Callback Refs

A more flexible alternative to **string refs** are the so-called c**allback refs**. While being more flexible, their handling is slightly more cumbersome and needs managed explicitly. Using **callback refs**, it is possible to pass refs to their **child components** and access the corresponding DOM elements within these.

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

