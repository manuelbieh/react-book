# Portals

**Portals** allow us to render components in DOM nodes which are _outside ****_of the parent-node of the current component hierarchy but still have access to the current component environment. A common example \(but of course not the only one\) is an overlay which is rendered in its own `<div>` outside of the actual application.

The portal remains in the context of the component that has created it and thus has access to all data that is also available to the parent component such as its props and state. However, they are placed in entirely different locations in the rendered HTML compared to the rest of the application. Being able to access props and state is crucial for **Portals**, as they allow us to access common Context such as translations.

### Creating portals

Creating a portal is relatively simple compared to other concepts we have learned about so far. The component intended to use the portal has to call the `createPortal()` method from ReactDOM and pass in a valid component as the _first_ and an existing destination node as the _second_ parameter.

The following example shows a common HTML snippet using portals:

```markup
<!doctype html>
<html>
<head>
<title>Portals in React</title>
</head>
<body>
<div id="root"><!-- this is where our React App is located --></div>
<div id="portal"><!-- this is where the content of the portal will be stored --></div>
</body>
</html>
```

While this snippet shows the corresponding React App:

```jsx
import React from 'react';
import ReactDOM from 'react-dom';

const App = () => {
  return (
    <div>
      <h1>Portals in React</h1>
    </div>
  )
}

ReactDOM.render(<App />, document.querySelector('#root'));
```

As our `<App />` is placed into the the `div` with the id of `root`, the `<body>` of the app would now result in this HTML snippet:

```markup
<body>
  <div id="root">
    <div>
      <h1>Portals in React</h1>
    </div>
  </div>
  <div id="portal"><!-- this is where the content of the portal will be stored --></div>
</body>
```

Each additional component or each additional HTML element which is used in the JSX of our App component would end up in the `div` with the `id="root"`. If we are dealing with a **Portal** however, the code would resemble the following:

```jsx
import React from 'react';
import ReactDOM from 'react-dom';

const PortalExample = () => {
  return ReactDOM.createPortal(
    <div>Portal says Hello</div>,
    document.querySelector('#portal')
  );
}
```

Here, we can see the `createPortal()` method in action: first, we indicate which type of JSX should be rendered and second, we pass in the type of container where the JSX should be rendered into. Let's place this `PortalExample` component into our App:

```jsx
import React from 'react';
import ReactDOM from 'react-dom';

const App = () => {
  return (
    <div>
      <h1>Portals in React</h1>
      <PortalExample />
    </div>
  )
}

ReactDOM.render(<App />, document.querySelector('#root'));
```

The resulting `<body>` in the HTML document will look like this:

```markup
<body>
  <div id="root">
    <div>
      <h1>Portals in React</h1>
    </div>
  </div>
  <div id="portal">
    <div>Portal says Hello</div>
  </div>
</body>
```

The Portal is rendered into the `#portal` node instead of the `#root` node where all other content including the component itself is placed. A Portal is rendered once the component mounts and is removed from the DOM if we the component containing the portal is removed from the component tree.

