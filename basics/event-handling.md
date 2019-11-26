# Event Handling

I know this is a little frustrating. For years, web developers have labored away and learned that event listeners should be neatly separated from the markup - The interaction between a user and the interface is foundational part of developing applications with complex user interfaces - especially with regard to **events**.

I click a button and something happens. I write text into an input field and something happens. I select an element from a list and something happens. In vanilla JavaScript, the browser provides us with the `addEventListener()` and `removeEventListener()` methods. In React however, you can safely ignore them in most use cases. React provides its own system to define user interaction and does so \(don't be scared now\) with **inline events**.

These **inline events** resemble HTML attributes \(for example `<button onclick="myFunction" />`\) but work entirely different.

**separation of concerns** anyone? But React introduces a very different way of dealing with events.

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

