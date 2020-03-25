# Event Handling

The interaction between a user and the interface is foundational part of developing applications with complex user interfaces - especially with regard to **events**.

I click a button and something happens. I write text into an input field and something happens. I select an element from a list and something happens. In vanilla JavaScript, the browser provides us with the `addEventListener()` and `removeEventListener()` methods. In React however, you can safely ignore them in most use cases. React provides its own system to define user interaction and does so \(don't be scared now\) with **inline events**.

These **inline events** resemble HTML attributes \(for example `<button onclick="myFunction" />`\) but work entirely different.

I know this is a little frustrating. For years, web developers have labored away and learned that event listeners should be neatly separated from the markup - **separation of concerns** anyone? But React introduces a very different way of dealing with events.

Behind the scenes, React facilitates a lot of the hard work and also enables to us to safely and easily stay in the component context by allowing the definition of event handlers as **class methods**. Layout logic as well as behavioral logic is encapsulated in a single component meaning we do not have to jump between many different controllers or views.

### Differences between The React event handlers and the native Event API

As mentioned previously, React and JSX events resemble HTML attribute definitions. But there are differences: events in React are defined with **camelCase** instead of **Lowercase** meaning `onclick` is changed to `onClick` in React, `onmouseover` is now defined by `onMouseOver` and `ontouchstart` would be written as `onTouchStart` - you get the picture.

The first parameter that is passed to the event handler is not an object of type `Event` as could be assumed. Instead, React supplies its own wrapper for the native event object, named a `SyntheticEvent`. The wrapper is part of React's event system and also works as some sort of normalizing layer to ensure cross-browser compatibility and, as opposed to some other browsers, it strictly follows the [event specifications of W3C](https://www.w3.org/TR/DOM-Level-3-Events/).

In order to prevent the standard behavior of the browser during an event, we cannot simply return `false` from the event handler. React forces us to explicitly call `preventDefault()` - another fundamental difference to usage in the native Browser API.

Last but not least, let us look at the event attribute or in React's case the event **prop**. React uses a **function reference** instead of a plain string \(which would be the standard in HTML\) which mandates the use of curly braces to inform JSX that a JavaScript expression is used.

This would look similar to:

```jsx
<button onClick={validateInput}>Validate</button>
```

For comparison purposes, this is how a similar event would look in HTML:

```text
<button onclick="validateInput">Validate</button>
```

While this might seem alienating to use function references as a prop, it offers a lot of advantages. We gain cross-browser compatibility basically "for free"! React neatly registers events in the background with `addEventListener()` and also safely and automatically removes them as soon as the component _unmounts_. How convenient!

### Scopes in event handlers

Usually the use of ES2015 classes in React mandate that event handlers have to be defined as methods of the current class component. However, class methods **are not automatically bound to the instance**. Let's unpack what this means: initially `this` will be `undefined` in all of our event handlers.

Here is an example to reiterate this:

```jsx
class Counter extends React.Component {
  state = {
    counter: 0
  };

  increase() {
    this.setState((state) => ({
      counter: state.counter + 1
    }));
  }

  render() {
    return (
      <div>
        <p>{this.state.counter}</p>
        <button onClick={this.increase}>+1</button>
      </div>
    );
  }
}
```

An `onClick` event is added to increment the counter by one each time the user presses a button labelled `+1`. But when the user clicks the button, instead of seeing the actual counter, they will receive an error message:

{% hint style="danger" %}
**TypeError**

Cannot read property 'setState' of undefined
{% endhint %}

Why's that? The answer is **scoping!** Whenever we click the button in the `increase()` event handler, we actually operate outside of the component instance. This means we cannot access `this.setState()` resulting in our above error. While it might seem annoying, it is not actually something React has thought up but it's actually standard behavior for ES2015 classes. But fear not, there are a number of techniques to combat this:

#### Method binding in the render\(\) method

Probably the most trivial solution is to _bind_ the method inside of the `render()` method. We add a `.bind(this)` to the reference of the class method:

```jsx
<button onClick={this.increase.bind(this)}>+1</button>
```

The method is now invoked in the **scope of the component instance** and our counter starts to increment the count as intended. While you might come across this method quite a few times, it is not entirely recommended and has one obvious advantage. With every call of the function, a new function is created "on-the-fly" which is different to the one before. A simple check using `shouldComponentUpdate()` to compare `this.props.increase === prevProps.increase` would yield `false` every single time and possibly even lead to re-render of the the component. Even if the function has not changed at all. Therefore, using this method is actually considered a **performance bottleneck** and should thus be avoided.

#### Method binding in the constructor

Another neater solution to bind a method to a class instance is to bind it when initializing a class in the constructor:

```jsx
class Counter extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      counter: 0
    };
    this.increase = this.increase.bind(this);
  }
  // […]
}
```

This way, the method is only bound to the instance **once** and possible checks to compare the method's likeness would always yield `true`. Thus, expensive `shouldComponentUpdate()` calls can easily be avoided. However, using this method of binding the method to the class instance is not without problems either: if the component in question does not yet use a constructor, it will now certainly have to. In order to to this, we need to call the `super(props)` method to pass the **props** of the component to the `React.Component` parent class. In the end, we end up writing the name of the method twice. Once on its own, and once to define and bind its `this`.

Using this method allows us to avoid potential performance bottlenecks, even if a little bit more verbose. But it could still be considered a little bit messy and cumbersome. We will now look at an even easier way to bind a method to a class instance.

#### **Class properties**

**Beware:** in order to use method I am about to explain, you need to have installed the babel plugin `@babel/plugin-proposal-class-properties`. But as most React setups already include this by default, I will assume that we can use **class properties** safely and without error. If this is not the case for some reason, event handler methods should always be bound in the constructor.

But how exactly do we bind our method via a **class property**? To be entirely correct: we're cheating. Instead of defining a real class method as shown in the above example, we define a **public class property** which is passed an **arrow function**:

```
class Counter extends React.Component {
  state = {
    counter: 0,
  };
  increase = () => {
    this.setState((state) => ({
      counter: state.counter + 1,
    }));
  }
}
```

The most important factor hides within the first line. Instead of:

```jsx
increase() { … }
```

We write:

```jsx
increase = () => { … }
```

**Problem solved!**

As mentioned earlier, we define a **real class method** within our first example whereas we assign a property within the class with same name an **arrow function** as a **value**. As it is not binding its own `this`, we access the `this` of the class instance instead.

### Events outside of the component context

While you can certainly also implement native browser events in React, you should try to use React's own event system whenever possible. It offers cross-browser compatibility, follows the W3C standard for browser events and also optimises when possible.

From time to time however, it is necessary to define events outside of the component context. Some classic examples are `window.onresize` and `window.onscroll`. React's event system does not support global events outside of the component context but if you want to define native browser events you can do so in the `componentDidMount()` method. You should pay attention though, whenever an event listener is added with `addEventListener()`, these **need to removed** once you're done with them.

The `componentWillUnmount()` method is the perfect place to do this. While it might seem annoying, global events can cause **performance bottlenecks** or even **memory leaks** if not removed properly as they would be added again each time a component is mounted and called multiple times.

### The `SyntheticEvent` Object

React does not pass a native object to its event handlers but an object of type `SyntheticEvent`. Its primary purpose is to ensure cross-browser compatibility. If you ever feel an urge to access the original event though \(I actually never felt the need to\), React provides it to you via the object property `nativeEvent`.

But that is not the only way how the `SyntheticEvent` object and native event object differ: the `SyntheticEvent` object is **short-lived** and **nullified** shortly after the event callback has been called \(mainly for performance reasons\). Accessing properties of the event object is not possible anymore once outside the original event handler.

What does that mean in detail? Let's look at another example:

```jsx
class TextRepeater extends React.Component {
  state = {};

  handleChange = (e) => {
    this.setState((state) => ({
      value: e.target.value
    }));
  };

  render() {
    return (
      <div>
        <input type="text" onChange={this.handleChange} />
        <p>{this.state.value}</p>
      </div>
    );
  }
}
```

An `onChange` event is registered that is added into a paragraph once the value in the text field has changed. In order to access the provided value, the `Event` object provides a property called `target`. You might have encountered it already as it has been used in jQuery and vanilla JavaScript too. The `target` allows us to access the element on which the event has been performed, the text field in our case. This in turn contains a `value` property which we can use to write the current value of the text field into state.

We are running into a bit of a problem though: `this.setState()` uses an **updater function** or more precisely a callback. However, it happens outside of the event handler scope meaning the `SyntheticEvent` has already been reset or `e.target` does not exist anymore.

{% hint style="danger" %}
**TypeError**

Cannot read property 'value' of null
{% endhint %}

The easiest solution for this problem is to define and object literal instead of an updater function:

```jsx
handleChange = (e) => {
  this.setState({
    value: e.target.value
  });
};
```

While this would certainly solve the problem, it would not help us much. We still encounter the first problem when trying to access the properties of the `SyntheticEvent` object if, for example, it was wrapped within a `setTimeout()` callback. We need to come up with another solution.

#### Writing values into variables

In most situations it is sufficient to write certain values that should later be accessed in a callback into their own variable. The callback does not try to acccess the `SyntheticEvent` anymore but only the variable which has been assigned a value from the `SyntheticEvent`.

```jsx
handleChange = (e) => {
  const value = e.target.value;
  this.setState(() => ({
    value: value
  }));
};
```

This works! Bonus points for using **object destructuring** and the **object property shorthand**.

```jsx
handleChange = (e) => {
  const { value } = e.target;
  this.setState(() => ({ value }));
};
```

#### Persisting `SyntheticEvents` with `e.persist()`

While it is not used much in practice, it is theoretically possible to use the `SyntheticEvent` object's `persist()` method to keep a reference to the event in question. This could possibly be useful when trzing to pass a `SyntheticEvent` object to a callback function **outside** of the event handler.

If you ever come across this situation though, it might be worth to considering whether that code of the callback function should actually live in the event handler itself. Our example function would look like this:

```jsx
handleChange = (e) => {
  e.persist();
  this.setState(() => ({
    value: e.target.value
  }));
};
```

First, the `e.persist()` method is invoked. Second, the **updater function** can safely access `e.target` and its `value` property.

### Summary

{% hint style="info" %}

- **Always** use event props in JSX to define events:`onChange`, `onMouseOver`, `onTouchStart`, `onKeyDown`, `onAnimationStart`etc \(even if it seems a little odd at first\)
- Event handlers have to be explicitly bound to the class instance if other class methods like `this.setState()` are accessed. **Public Class Properties** und **Arrow Functions** are the more elegant ways to do this.
- Avoid defining your own events with `addEventListener()`API. If at all necessary, do not forget to remove the event when unmounting your component with`removeEventListener()`
- `SyntheticEvent`objects are „nullified“. Beware of using callback functions outside of the event handler. The event object might not exist anymore at the time of calling the callback.
- `event.persist()` can force React to prevent resetting the event object to `null`.
  {% endhint %}
