# Portals

**Portals** allow us to render components in DOM nodes which are _outside_ of the parent-node of the current component hierarchy but still have access to the current component environment. A common example \(but of course not the only one\) is an overlay which is rendered in its own `<div>` outside of the actual application.

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
  );
};

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
};
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
  );
};

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

### Portals and their relationship to their parent component

In order to further our understanding of portals, we are going to build — surprise surprise — a modal portal. The basis is formed by the same HTML which we have used in the introduction of portals before. There are two divs in the example: one in which our application is rendered and another in which we render the portal.

This time however, the modal will only open once a user has clicked a button. The portal will contain a button which allows the user to close the window. A state variable called `modalIsOpen` is used to alternate between the two states and is either true or false. The `ModalPortal` component will be rendered via an `&&` conditional in JSX, thus it is only shown if this.state.modalIsOpen is true.

During the time in which the value of the state changes from `false` to `true`, the `ModalPortal` component is mounted and the ModalPopup is rendered into the `<div id="portal">` with a slightly transparent background. Once the value changes from `true` to `false` again, the `ModalPortal` is removed from the App component in the component tree. React takes care to ensure that the `ModalPortal` component and its contents are not found on the page anymore.

In code form, we are left with the following example:

```jsx
import React from 'react';
import ReactDOM from 'react-dom';

const ModalPortal = (props) => {
  return ReactDOM.createPortal(
    <div
      style={{
        background: 'rgba(0,0,0,0.7)',
        height: '100vh',
        left: 0,
        position: 'fixed',
        top: 0,
        width: '100vw',
      }}
    >
      <div style={{ background: 'white', margin: 16, padding: 16 }}>
        {props.children}
      </div>
    </div>,
    document.getElementById('portal')
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
        <button onClick={this.openModal}>Open Modal</button>
        {this.state.modalIsOpen && (
          <ModalPortal>
            <p>This text is opened in a Portal.</p>
            <button onClick={this.closeModal}>Close Modal</button>
          </ModalPortal>
        )}
      </div>
    );
  }
}
ReactDOM.render(<App />, document.getElementById('root'));
```

We need to pay special attention to the `this.closeModal()` method. Even though this method is defined in the `App` component, it is called within the `ModalPortal` component in the context of the `App` component once a user has clicked on the button "Close Modal".

This method can also alter the state of component via `modalIsOpen` even though the component is not placed within `<div id="root">` as the rest of the components. Portals allow us to do this as the content is placed within the same component tree **within React**. The **resulting HTML** however, is different and the code is placed in a different `<div>` to the rest of the application.
