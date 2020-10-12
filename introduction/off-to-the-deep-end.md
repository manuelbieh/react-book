# Into to the deep end

We have just talked about the "what", "when" and "where" but we have not yet talked about the "how". Let's write our first **React component.** In order to display our component in the browser, we need to not only install **React** but also **ReactDOM**, a package that enables us to mount our application in the browser — put simply: to use it in the browser.

Let's have a look at a very minimalist setup to get started with React:

```markup
<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8" />
<title>Hello React!</title>
</head>
<body>
<div id="root"></div>
<script crossorigin src="https://unpkg.com/react@16.8.4/umd/react.development.js"></script>
<script crossorigin src="https://unpkg.com/react-dom@16.8.4/umd/react-dom.development.js"></script>
<script>
// Placeholder for our first component
</script>
</body>
</html>
```

We build up the bare bones set up for a regular HTML document and load in **React** and **ReactDOM** in their latest _stable_ version from unpkg-CDN. We can now use React and ReactDOM as a global variable in the window object as `window.React` and `window.ReactDOM`. Apart from that we only see an empty page here with an empty `<div id="root"></div>`. This div is going to be used as our **mount node** to show our first React component.

{% hint style="info" %}
If we deal with a number of React components, we normally refer to this as an **app**, **web app** or **single page app**. The boundaries of when we call a component an app are fluid though. A few developers talk of an **app** if we only have a single component in our code. A clear definition of this does not exist though.
{% endhint %}

Let's start with a classic "Hello-World" example and include the script where we put the placeholder earlier.

```javascript
class HelloWorld extends React.Component {
  render() {
    return React.createElement('div', { id: 'hello-world' }, 'Hello World');
  }
}

ReactDOM.render(
  React.createElement(HelloWorld),
  document.getElementById('root')
);
```

And just like that we have implemented our first React component. If we include this snippet where we have put the placeholder earlier, we can see the following in the browser:

![Our first React component in the browser](../.gitbook/assets/first-component.png)

That does not look that complicated, does it? Let's go through the code one step at a time. I have highlighted all the relevant parts in the code in bold.

```jsx
class HelloWorld
```

Here, we are giving a name to a child component. We have named our first component **HelloWorld.** We can name our components anything we like, but there is one rule to keep in mind: React components start with a capital letter. Using helloWorld as a component name would not be valid, HELLOWORLD, however, would be — albeit very rarely seen in the wild.

Naming components follows the **UpperCamelCase** convention. It is not unusual to have long, self-explanatory names for our components — **UserNotificationView** is an entirely normal name, for example.

```jsx
extends React.Component
```

With this snippet, we are extending the internal React class `React.Component`. This transforms our own class into a component that we can use in React. Apart from `React.Component` there is also a `React.PureComponent` and a so-called _Functional component_. This is simply a JavaScript function that follows a particular pattern. We are going to look at both of these at a later stage, for now we can neglect them.

```jsx
render();
```

So far, our component only contains the obligatory `render()` method. This method is necessary to inform React how the component is displayed — we say "rendered" in the React world. A component has to have a `return` value. This value can either be an explicit `null` value to show nothing \(but not `undefined`\), another **React element**, or from Version 16 onward, a **string** or **array**.

In terms of an array, the array can include strings, numbers, React elements or `null` as values. The `render()` method **declaratively** describes the current state of our user interface. Everything that is included in our render method right after `return` is what the browser will display after rendering.

Even if it is entirely up to us how to structure our JavaScript classes, a common pattern has evolved to put our `render()` method at the end of all our methods. This is not mandatory of course but it aids readability. Many renowned engineers in the React scene advocate for this guideline too. The code guidelines by AirBnB also include this rule. Speaking from my own experience here, I can say that it does help to follow this guideline to make your day-to-day React work easier.

![Error message for missing render\(\) method](../.gitbook/assets/react-no-render-error.png)

![Error message for missing render\(\) method](../.gitbook/assets/invalid-react-element.png)

```jsx
React.createElement();
```

As mentioned previously, the `render()` method of a React component returns a **React element** in most cases. React **elements** are the smallest but also most significant building blocks in a React application and describe what a user will see on their screen. Apart from `React.cloneElement()` and `React.isValidElement()`, `React.createElement()` has been one of of the only 3 top level API methods for a long time. Since then though, a few other methods have been added but they are mainly for performance optimizations.

The `createElement()` method expects 1-n parameters:

1. "Type" - this can be a HTML element as a string, i.e. `'div'` , `'span'` or `'p'` but also other React components.
2. So-called "Props" - these are read-only "property objects" of a component. The name is derived from - you guessed it - properties.
3. As many child elements as you wish to put in. These can be React elements themselves, or arrays, functions or simply plain text. However, a component does not necessarily need to contain other child elements.

At the end of the day, **React elements** are nothing more than a never changing \(immutable\) JavaScript objects that contain properties to tell React how and what exactly needs displayed. This description of properties is used by React to construct a virtual blueprint of the component hierarchy. It resembles the representation of the HTML tree in the form of a JavaScript object. You often hear it being referred to as the VirtualDOM, however the term is losing popularity with the React team and they have distanced themselves from using it. This tree is used by React to only refresh the parts of the tree that have actually changed, e.g. when a user interacts with an app and changes data or fires an event. To do this, the previous tree is compared to the current tree.

As React does not refresh the entire application at once, or writes it to the DOM in its entirety once any state changes, it is a lot more performant than other frameworks or libraries. Other libraries or frameworks can often lead to unnecessarily many DOM mutations at the cost of performance. Using a special **reconciliation** algorithm that compares previous state with current state, React processes what exactly has changed, and can reduce the number of writes to a minimum.

However, in our day-to-day development we will hardly ever call `React.createElement()` in this form. Facebook has developed its own syntax extension for JavaScript called **JSX**. JSX reduces a great amount of work and simplifies a lot of work with React. Nevertheless, I am of the opinion that it is good to know of `React.createElement()`'s existence to understand how JSX works behind the scenes and to reduce the number of errors when debugging.

**JSX** resembles HTML or XML/XHTML in parts, however it is a lot more powerful as we can include JavaScript expressions within it. JSX is an abstraction that makes it a lot **easier** for the developer to create React elements. Our previous example

```jsx
React.createElement('div', { id: 'hello-world' }, 'Hello World');
```

would look like this in JSX:

```jsx
<div id="hello-world">Hello World</div>
```

For many beginners, this can look a little strange. I have heard this referred to as **JSX shock** but after playing around with JSX for a little while, it practicality becomes obvious. In my opinion, JSX is one of the reasons why React's usage has gained so much traction in such a short time.

Let's have a look at our example again. The `return` value of the `render()` method indicates that it will display an element of type `div` which shall contain the id `hello-world` and the child element \(a text node in this case\) containing `Hello World`.

```jsx
ReactDOM.render(Element, Container);
```

We have talked about most of the parts in our example but we have not yet looked at **ReactDOM**. This library enables React to interact with the DOM \(_Document Object Model_\) — put simply, it allows us to interact with the browser. Just like React itself, **ReactDOM** only contains a few top level API methods. We are going to concentrate on the `render()` method — the core of **ReactDOM** in the browser.

Although the naming might suggest otherwise, **ReactDOM**'s internal `render()` method does not directly relate to its counterpart used in React components. Using it in this case, enables us to render a **React element** onto a **"root node"**, meaning to display it on the screen. In our example, we render our `HelloWorld` component onto the `<div id="root"></div>`. It is important to understand that the root node is **not replaced**, but that the content is inserted **into the container**.

**ReactDOM** enables us to actually see our component in the browser. The content that we can see is described in the `render()` method, which contains a React element after the `return`. Calling `ReactDOM.render()` with the two parameters as shown above, will insert the **React** element into the given **container**.

{% hint style="info" %}
Calling the `ReactDOM.render()` function for the first time, any possibly existing content in the destination container is being replaced. With every subsequent call, React uses an internal comparison algorithm for best performance and to avoid re-rendering the entire application.

While this is good to know in theory, we normally do not call `ReactDOM.render()` more than once and only use it to initialize our Single Page App to load the page. React never changes the root node itself, only its contents. This means that if our container element has classes, ids or a data-attribute, these will remain intact after calling `ReactDOM.render()`.
{% endhint %}

And just like that, we have discussed the basics of how React works under the hood, implemented our first component and finally displayed it in the browser. Congrats!

