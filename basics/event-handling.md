# Event Handling

An essential part in the development of applications with a complex user interface is of course the interaction between the user and the interface itself - especially in the form of **events**.

I push a button and something happens. I write a text in a field and something happens. I choose an item from a list and something happens. In simple JavaScript the browser provides us with the methods `addEventListener()` and `removeEventListener()`. These two event methods can in most cases be omitted almost completely in React, since React comes with its own system for defining user interaction. And that's it. Don't be scared about inline events.

These **Inline-Events** are very similar to the HTML event attributes \(e.g. `<button onclick="myFunction" />`\) and yet their functionality is fundamentally different.

We've been taught over the years to web developers to separate their event listers from their markup, **Separation of Concerns**, then React comes along and just does everything differently again.

And that's a good thing in this case, because by using our own event management system, React does a lot of the work for us and also allows us to easily stay in the component context by implementing all event handlers as **class methods** and thus encapsulating both presentation logic and behavioral logic in a single component. No tedious and confusing jumping back and forth between controllers and views!

## Differences to the native Event API

Events in React and JSX are defined very similar to HTML event attributes. But there are some differences. So events are defined in React in **CamelCase**-form instead of in **Lowercase**. So `onclick` becomes `onClick` in React, `onmouseover` becomes `onMouseOver`, `ontouchstart` becomes `onTouchStart` and so on.

The first parameter passed to the event handler is not an object of the type `Event`, but a so-called `SyntheticEvent`, a React-own wrapper around the native event object. This wrapper is part of the React Event System and serves as a kind of normalization layer to ensure cross-browser compatibility. In contrast to some browsers, it strictly adheres to the [W3C event specification](https://www.w3.org/TR/DOM-Level-3-Events/).

Another difference to the native browser event API is that `preventDefault()` must be called explicitly to suppress the default behavior of the browser during an event, instead of just returning `false` from the event handler.

Last but not least, the value specified in the event attribute \(or Event-**Prop** in JSX\) is a **reference to a function** instead of a string as in HTML. Consequently, we need the curly braces here to tell JSX that it is a JavaScript expression.

What does it look like? There you go:

```jsx
<button onClick={validateInput}>Validate</button>
```

Whereas a comparable event in HTML would look like this:

```markup
<button onclick="validateInput">Validate</button>
```

This approach, which at first glance might seem a bit strange to some, has some great advantages. And so we get cross-browser compatibility "for free", as already mentioned at the beginning. React registers the events cleanly in the background using `addEventListener()` and automatically removes them for us as soon as the component is removed \(_unmounted_\). Practical.

## Scopes in event handlers

When using ES2015 classes in React, it is usually the case that event handlers are implemented as methods of the class component. Note, however, that **class methods are not automatically bound to the instance**. Sounds complicated, but only means that `this` in your event handler is `undefined` for now.

Here is a brief example:

```jsx
class Counter extends React.Component {
  state = {
    counter: 0,
  };

  increase() {
    this.setState((state) => ({
        counter: state.counter + 1,
    }));
  }

  render() {
    return (
      <div>
        <p>{this.state.counter}</p>
        <button onClick={this.increase}>+1</button>
      </div>
    )
  }
}
```

So here we define an `onClick` event to increase the counter by `1` each time the user clicks the `+1` button. When clicking on the button, however, our user sees instead:

{% hint style="danger" %}
**TypeError**

Cannot read property 'setState' of undefined
{% endhint %}

And why is that? **Scoping!** Since we move outside the component instance when clicking the button in the event handler `increase()`, we cannot access `this.setState()` either. This is not a faulty behavior of React, but the default behavior of ES2015 classes. There are now several ways to solve this problem.

### Method binding in the render\(\) method

The most trivial method is binding the method within the `render()` method. We add a `.bind(this)` to the reference to the class method:

```jsx
<button onClick={this.increase.bind(this)}>+1</button>
```

The method is now called **in the scope of the component instance** and our counter counts up without problems. This pattern can be seen quite often because it is simply implemented, but it has a basic disadvantage. This is because each time the method is called "on-the-fly", a new function is created that is no longer identical to the previous one. A simple check in a `shouldComponentUpdate()`method, which checks for equality of `this.props.increase === prevProps.increase`, would always result in `false` and possibly cause a rendering of a component, even if the function itself has not changed. This is a potential **Performance-Bottleneck** and should be avoided!

### Method binding in the constructor

Another cleaner way to bind a method to a class instance is to do this in the constructor when instantiating a class.

```jsx
class Counter extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      counter: 0,
    };
    this.increase = this.increase.bind(this);
  }
  // […]
}
```

In this way, the method is bound to the instance only  **once**. A possible check for equality of methods would result in `true` and unnecessary renderings could be avoided by using reasonable `shouldComponentUpdate()`checks. But this way also brings with it a lot of overhead. Provided that we are not already using a constructor, we have to implement it now. We should call the `super(props)` method to pass the **Props** of the component to the `React.Component` parent class. Finally, we write twice the name of the method we want to bind by overwriting it with the `this` bound version of itself, so to speak.

That's still better than the first method, even if it's more paperwork, because we don't need potential performance bottlenecks. But there is another simple way to bind a method to an instance with minimal additional effort.

### Use of class properties

**Attention:** this way requires the use of the Babel plugin `@babel/plugin-proposal-class-properties`. However, since this is standard in most React setups, as described in the introduction, I assume that **Class Properties** can be used. If this is not the case, event handler methods should always be bound in the constructor!

But how do we now bind our method by using **Class Properties**? Strictly speaking: by cheating! Instead of implementing a real class method as in the first example, we define a **Public Class Property**, which is assigned as value a function bound to the respective class by **Arrow Function**. This will look like this:

```jsx
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

The decisive factor here lies in the first line. Instead of

```jsx
increase() { ... }
```

Let's write:

```jsx
increase = () => { ... }
```

\*\*Problem solved!

The difference here, as mentioned, is that in the first example we implement a **true class method** and in the second case we assign a **Arrow Function as value** to a property of the same name of this class instead. Since it doesn't bind its own `this`, we access the `this` of the class instance here.

## Events outside the component context

The use of React does not exclude the implementation of native browser events. However, React's own event system should always be used if possible, as it provides cross-browser compatibility, works according to the W3C standard for browser events, and makes numerous optimizations.

Occasionally, however, it is necessary to define events outside the component context. A classic example are `window.onresize` or `window.onscroll` events. In this case React does not offer any possibility to define global events outside the component-specific JSX. Here the `componentDidMount()` method is the right place to define it. However, you should always make sure that events defined with `addEventListener()` are \*\*always removed!

The right place **for that** is then the `componentWillUnmount()` method. If your own defined global events are not removed, they are **replenished** with each mounting of a component and also **called again**, which can ultimately lead to **performance bottlenecks** and even **memory leaks**.

## Working with the `SyntheticEvent` object

\*\*React does not pass a native event object to event handlers, but a separate object of the type `SyntheticEvent`. This serves in particular to ensure cross-browser compatible behavior. However, React provides the original event in the object property `nativeEvent`. So if you ever need to access the original event \(never happened to me before\): you can find it in the `e.nativeEvent` property.

But the `SyntheticEvent` object has another special feature compared to the native event object, because it is **shortlived**. For performance reasons, the object is **nullified** after the event callback is called. In short, it is reset to its initial state. It is therefore not possible to access the properties of the event object outside the original event handler without further ado.

What's the meaning of this? Well, here's an example:

```jsx
class TextRepeater extends React.Component {
  state = {};

  handleChange = (e) => {
    this.setState((state) => ({
      value: e.target.value,
    }));
  }

  render() {
    return (
      <div>
        <input type="text" onChange={this.handleChange} />
        <p>{this.state.value}</p>
      </div>
    )
  }
}
```

We register an `onChange` event, which displays the entered value in a paragraph below the text field when the text field is changed. To access the entered value in the event handler, you may already know this from native JavaScript or from jQuery events, the `Event` object provides the property `target`. This gives us the element on which the event took place, in our example the text field. This in turn has a `value` property by which we then access the current value of the text field to write it to our state.

But here we have to do with a pitfall: the `this.setState()` call uses a **Updater function**, i.e. a callback. This takes place outside the actual event handler scope. This means that the `SyntheticEvent` was already reset at this point and `e.target` does not exist anymore at the time the updater function was called:

{% hint style="danger" %}
**TypeError**

Cannot read property 'value' of null
{% endhint %}

The simplest solution here would be to use an object literal instead of the updater function:

```jsx
handleChange = (e) => {
  this.setState({
    value: e.target.value,
  });
}
```

That would solve our problem in this case, but it still doesn't help us much. We encounter the same problem when we want to access properties of the `SyntheticEvent` object, e.g. in a `setTimeout()` callback. So we'll have to think of something else.

### Write values to variables

In most cases it should be sufficient if single values, which are to be accessed later in the callback, are written into a variable. In the callback, the `SyntheticEvent` is no longer accessed, but only the variable that has been assigned a value from the `SyntheticEvent`.

```jsx
handleChange = (e) => {
  const value = e.target.value;
  this.setState(() => ({
    value: value,
  }));
}
```

It works. Elegance bonus points are awarded when using **Object Destructuring** and the **Object Property Shorthand**:

```jsx
handleChange = (e) => {
  const { value } = e.target;
  this.setState(() => ({ value }));
}
```

### Persisting SyntheticEvents with `e.persist()`

Theoretically possible, in practice rather irrelevant from my experience: The `SyntheticEvent` object provides its own method `persist()` with which a reference to the corresponding event can be maintained \(i.e. persisted\). A possible use case here would be to pass the entire `SyntheticEvent` object to a callback function **outside** the event handler.

However, should this be necessary, it might be worth considering whether the code of the external callback function would not be better stored in the event handler itself. Our example function from above looks like this in this case:

```jsx
handleChange = (e) => {
  e.persist();
  this.setState(() => ({
    value: e.target.value,
  }));
}
```

First we call said `e.persist()` method. Then we can also access `e.target` and its `value` property in the **Updater function**.

## Conclusion

{% hint style="info" %}

* To define events always use the event props in JSX: `onChange`, `onMouseOver`, `onTouchStart`, `onKeyDown`, `onAnimationStart` etc. even if it should take getting used to at first.
* Event handlers must be explicitly bound to the class instance when accessing other class methods such as `this.setState()`. The most elegant way to do this is **Public Class Properties** and **Arrow Functions.**
* Defining your own events via the `addEventListener()`-API should be avoided if possible. If you can't avoid it for once, don't forget to remove these events when unmounting the component with `removeEventListener()`!
* `SyntheticEvent` objects are nullified. Caution when using in callback functions outside the respective event handler! Here it is quite possible that the event object no longer exists when the callback is called.
* With `event.persist()` you can force that the event object is not set to `null` by React.

