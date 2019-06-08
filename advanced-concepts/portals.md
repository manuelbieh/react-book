# Portals

With **Portals** \, React provides the ability to render components in DOM nodes that are _outside_ the parent node of the component hierarchy, but still have access to the current component environment. A possible \(but by far not their only\) use case for this are overlays, which are rendered in their own `<div>` outside the actual application.

A portal is located further in the context of the component that creates the portal and thus has access to all data that is available in the parent component \(such as the props or the state\) but is located in HTML at a possibly completely different location than the rest of the application. This is important if, for example, data from the state of the parent component or a common context such as translations is to be accessed within the portal. 

### Create a portal

The creation of such a portal is very easy. So a component only has to call the `createPortal()` method from ReactDOM, give it a valid component as _first_ and a \(existing\) target node as _second_ parameter.

Let's assume the following HTML document:

```markup
<!doctype html>
<html>
<head>
<title>Portals in React</title>
</head>
<body>
<div id="root"><!-- here is our React App --></div>
<div id="portal"><!-- and here the content of our portal lands --></div>
</body>
</html>
```

And the following simple React App:

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

Since we render our `<App />` into the `div` with the id `root`, the `<body>` of our above app would look like this:

```markup
<body>
  <div id="root">
    <div>
      <h1>Portals in React</h1>
    </div>
  </div>
  <div id="portal"><!-- and here the content of our portal lands --></div>
</body>
```

Any additional component or HTML element we use in the JSX of our app component would end up in the `<div id="root">` accordingly. Unless, of course, it's a portal. Such a component would then look like this:

```jsx
import React from 'react';
import ReactDOM from 'react-dom';

const PortalExample = () => {
  return ReactDOM.createPortal(
    <div>Hello from the portal</div>,
    document.querySelector('#portal')
  );
}
```

In addition to the JSX that we want to output at this point, we also specify the target container and package both together nicely in a `ReactDOM.createPortal()` call, which we then return from the component \(or from the `render()` method for Class Components\) instead of the pure JSX. If we add our example app from above, the usage would look as follows:

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

The `<body>` of our HTML document is then the following:

```markup
<body>
  <div id="root">
    <div>
      <h1>Portals in React</h1>
    </div>
  </div>
  <div id="portal">
    <div>Hail from the portal</div>
  </div>
</body>
```

So the portal is rendered to the `#portal` node instead of the `#root` node where the component is located. A portal is always rendered when the component is _mounted_ and is therefore _removed_ from the DOM when the component that contains the portal is removed from the component tree.

### A portal in interaction with its parent component

In order to demonstrate the functionality of a portal even more clearly, the next step - surprise - is to develop a modal portal. As a starting point we use the identical HTML as in the introduction before. So we have two divs in which we render our application and our portal.

This time, however, we will only open the portal after the user has clicked a button. The portal itself then contains a button that closes the window again. We set the state property `modalIsOpen` in the parent component to `true` or `false` accordingly. We render the `ModalPortal` component via a `&&` conditional in JSX, i.e. only if the value of `this.state.modalIsOpen` is actually `true`.

The moment the value changes from `false` to `true`, the `ModalPortal` component is mounted and the modal popup is rendered with a slightly transparent black background in the `<div id="portal">`. If the value changes from `true` back to `false`, we take it out of the component hierarchy in the app component and React then automatically ensures that the `ModalPortal` component and its contents are no longer in the page.

And in the code this results in the following image:

```jsx
import React from "react";
import ReactDOM from "react-dom";

const ModalPortal = (props) => {
  return ReactDOM.createPortal(
    <div
      style={{
        background: "rgba(0,0,0,0.7)",
        height: "100vh",
        left: 0,
        position: "fixed",
        top: 0,
        width: "100vw",
      }}
    >
      <div style={ background: "white", margin: 16, padding: 16 }}>
        {props.children}
      </div>
    </div>,
    document.getElementById("portal")
  );
};

class App extends React.Component {
  state = {
    modalIsOpen: false,
  };

  openModal = () => {
    this.setState({ modalIsOpen: true });
  };

  closeModal = () => {
    this.setState({ modalIsOpen: false });
  };

  render() {
    return (
      <div>
        <h1>Portals in React</h1>
        <button onClick={this.openModal}>OpenModal</button>
        {this.state.modalIsOpen && (
          <ModalPortal>
            <p>This part is opened in a modal window.</p>
            <button onClick={this.closeModal}>Close Modal</button>
          </ModalPortal>
        )}
      </div>
    );
  }
}

ReactDOM.render(<App />, document.getElementById('root'));

```

The `this.closeModal` method is of particular interest here. This is defined as the method of the app component, but is called within the `ModalPortal` component when clicking the "Close Modal" button in the context of the `App` component. 

So it can easily change the `modalIsOpen` state of the component. And this although the component is not located inside `<div id="root>` like the rest of our small app. This is possible because it is a portal whose content is **from react view** in the same component tree as the app itself, but not **from HTML view**.

