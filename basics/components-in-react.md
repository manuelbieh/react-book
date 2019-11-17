# Components in React

## The two types of React components

We've already implemented our first `Hello World` component in the section "Into the deep end". While this component taught us the basics of **React** and **React components**, it was a simplistic example and did not make use of everything React offers.

**Components** are relatively easy to explain: they allow us to de-construct complex user interfaces into multiple parts. Ideally, these parts are reusable, isolated and closed in itself. They can deal with any input from the outside so-called **props** \(from "properties"\) and describe what's going to be rendered on the screen by whatever they define in the `render()` method.

We can cluster components into two different versions: **function components**, which are expressed in the form of a function, and **class components**, which build upon the newly introduced ES2015 classes. Until recently the term **Stateless Functional Component** was very common as state could only managed in class components before the introduction of React Hooks. We are going to take a deep dive into **Hooks** and how state is managed from React 16.8.0 onward in a later chapter.

### Function Components

The simplest way to define a React component is in the way of a **function component**. You might have guessed, function components are just regular JavaScript functions:

```jsx
function Hello(props) {
  return <div>Hello {props.name}</div>
}
```

To be classified as a valid **React component** the function has to fulfill two criteria. It has to either return an explicit `null` \(`undefined` is not allowed\) or a valid `React.element` \(in the form of JSX\) and receives a `props` object. But even the `props` object can be optional and take in `null` as an argument.

### Class Components

The second method of defining a **React component** has already been shown to you in a previous chapter! But let's mention it once again - I am talking about **Class components**. These are based upon the ES2015 classes and extend `React.Component` or `React.PureComponent`. Don't worry if you to not know what `PureComponent` is for now, we are going to look at them soon. Class components have at least one method named `render()`:

```jsx
class Hello extends React.Component {
  render() {
    return <div>Hello {this.props.name}</div>
  }
}
```

There is one big pitfall to look out for: **Function components** take in their **props** as function arguments. **Class components** on the other hand cannot take in arguments and can only access its **props** via the instance method `this.props`. 

Both of these components will render the exact same result!

{% hint style="info" %}
One thing to keep in mind is to always name your components according to the standard. Both function and class components require their `displayName`\(so their actual component name\) to start with a **capital letter**. In between the rest of the characters, capital and lower case letters can be used interchangeably as long as the first letter is **capitalized**.

If you mistakenly name your React component something along the lines of `section`, React would interpret it as the corresponding DOM element which is of course something we would like to avoid.`Section` on the other hand would be a valid name for a component and would thus be differentiated from the regular DOM element.
{% endhint %}

I am going to consciously avoid going into detail about `state` in this chapter. State is a very complex topic and is thus getting a chapter of its own. I would advise you to work through this chapter first to understand how components work to then delve deeper into state in the next chapter.

### Special case: PureComponent

The **Pure Component** is a special form of the **Class Component**. It inherits from `React.PureComponent` and works similarly to a `React.Component` with the difference that React only renders a Pure Component if its **props** or **state** have changed compared to the previous render phase. It is often used to optimize performance.

This is achieved by performing a "shallow" comparison to identify whether the **references** are identical to the previous render. It does not matter if the **values** themselves did not change, a **Pure Component** will still re-render if the references have changed.

This sounds a little abstract. Let us look at another two examples to illustrate the point:

```jsx
const logFunction = (message) => console.log(message);

class App extends React.Component {
  render() {
    return <MyComponent logger={logFunction} />;
  }
}
```

We have defined a function called `logFunction` outside of the class in this example. `MyComponent` receives a prop called ~~`logger`~~ with a **reference** to the `logFunction`. The **identity** of the **reference** will stay the same after a re-render. A **pure component** would not re**-**render if its state did not change as the props are identical to that of the previous render.

However, in the following example a **pure component** _would_ be rendered:

```jsx
class App extends React.Component {
  render() {
    return <MyComponent logger={(message) => console.log(message)} />;
  }
}
```

While we still pass the same function to `MyComponent`, we create a new function identity with each new render. Because of this, the "shallow comparison" will no longer match between the previous and new props and will fail. A re-rendering is triggered.

The same applies to Objects and Arrays:

```jsx
<MyComponent
  logConfig={{ logLevel: 'info' }}
  logEntries={['Message 1', 'Message 2']} />  
```

This would also trigger a re-render as the `logConfig` object or array would be replaced with each new render.

### "Pure" Function Components

**Class components** are derived from the `Component` or `PureComponent` class. By defining this class we are deriving from, we choose whether the component should re-render with each new change in a component which is higher up in the hierarchy. Regular **function components** do not offer said functionality as they are merely JavaScript functions. Since **React 16.6.0** React offers a new wrapper function called `React.memo()` which also allows us to optimize our re-renders in **function components**. We wrap the call around the function in question:

```jsx
const MyComponent = React.memo((props) => {
  return <p>I only re-render if my props change</p>;
});
```

Using `React.memo()`, the **function component** works similarly to the **class component** equivalent which is derived from the `React.PureComponent`.

Curious already? Play around with the demo to deepen your understanding:

```jsx
import React from "react";
import ReactDOM from "react-dom";

class ClassComponent extends React.Component {
  render() {
    return <p>Class Component: {new Date().toISOString()}</p>;
  }
}

class PureClassComponent extends React.PureComponent {
  render() {
    return <p>Pure Class Component: {new Date().toISOString()}</p>;
  }
}

const FunctionComponent = () => {
  return <p>Function Component: {new Date().toISOString()}</p>;
};

const MemoizedFunctionComponent = React.memo(() => {
  return <p>Memoized Function Component: {new Date().toISOString()}</p>;
});

class App extends React.Component {
  state = {
    lastRender: new Date().toISOString(),
  };

  componentDidMount() {
    this.interval = setInterval(() => {
      this.setState({ lastRender: new Date().toISOString() });
    }, 200);
  }

  componentWillUnmount() {
    clearInterval(this.interval);
  }

  render() {
    return (
      <div>
        <p>App: {this.state.lastRender}</p>
        <ClassComponent />
        <PureClassComponent />
        <FunctionComponent />
        <MemoizedFunctionComponent />
      </div>
    );
  }
}

ReactDOM.render(<App />, document.getElementById("root"));
```

The `App` Component triggers a re-render every 0.2 seconds without new **props** being passed. The question of all questions is: Which components are being re-rendered and which are not?

## Component composition - multiple components in one

We've only really included DOM elements in our example components so far. But **React components** can also include other React components. By defining the component in the same scope or by using CommonJS or ES-Modules imports with`require()` or `import`, other components can be used in the same scope.

**Example:**

```jsx
function Hello(props) {
  return <div>Hello {props.name}</div>;
}

function MyApp() {
  return (
    <div>
      <Hello name="Manuel" />
      <Hello name="Tom" />
    </div>
  );
}

ReactDOM.render(<MyApp />, document.getElementById('app'));
```

`<MyApp />` returns a `<div>` which contains the `Hello` component which is used to greet both Manuel and Tom. The result:

```jsx
<div>
  <div>Hello Manuel</div>
  <div>Hello Tom</div>
</div>
```

Important: ****A component can only ever return _one_ **root element**. This can either be:

* a single React element:

```jsx
<Hello name="Manuel />
```

* in nested form - as long as only a single element is on the outer layer:

```jsx
<Parent>
  <Child />
</Parent>
```

* a DOM element - which in turn can also be nested and include other elements:

```jsx
<div>…</div>
```

* ... or it can be self--closing:

```jsx
<img src="logo.jpg" alt="Bild: Logo" />
```

* or simply:

```javascript
null
```

... however **never** `undefined`.

Since **React 16.0.0** we are also allowed to return:

* an array which contains valid return values:

```jsx
[
  <div key="1">Hello</div>, 
  <Hello key="2" name="Manuel" />
]
```

* a simple string

```javascript
'Hello Welt'
```

* or a so-called "Fragment" - a special "component" which does not appear in the rendered output and can act as a container if one otherwise violated the rule of only ever returning one root element:

```jsx
<React.Fragment>
  <li>1</li>
  <li>2</li>
  <li>3</li>
</React.Fragment>
```

Since **Transpiling** with **Babel version 7**, fragments can also be expressed in their short form which contain an empty opening and closing element:

```jsx
<>
  <li>1</li>
  <li>2</li>
  <li>3</li>
</>
```

Components can be composed into anything you like. It makes sense to break up large and complex components into many smaller and easier to read components to improve readability and make them reusuable. This process often happens organically as you program as you will come to a point at which you will notice that breaking up your components would become useful.

## Dividing components - keeping an overview

Let's have a look at an example of a Header component which includes a logo, a navigation and a search bar. This is a common example which we come across plenty in web development:

```jsx
function Header() {
  return (
    <header>
      <div className="logo">
        <img src="logo.jpg" alt="Image: Logo" />
      </div>
      <ul className="navigation">
        <li><a href="/">Homepage</a></li>
        <li><a href="/team">Team</a></li>
        <li><a href="/services">Services</a></li>
        <li><a href="/contact">Contact</a></li>
      </ul>
      <div className="searchbar">
          <form method="post" action="/search">
            <p>
              <label htmlFor="q">Search:</label>
              <input type="text" id="q" name="q" />
            </p>
            <input type="submit" value="Suchen" />
          </form>
      </div>
    </header>
  );
}
```

We have just learned that React components can easily be composed of smaller other React components. This way of breaking up components is highly encouraged in React. So what could we do with the above code snippet to make it easier for ourselves? That's right: we can break up our relatively big component into multiple smaller ones which all only do one thing and do it well.

First, the logo can easily be used elsewhere on the page which is a totally valid assumption. Second, the navigation might not only appear in the header but might also be included in the sitemap. Last, the search bar might eventually not only be used in the header but also on its own dedicated search results page. 

Transferring this approach to our code, we achieve the following result:

```jsx
function Logo() {
  return (
    <div className="logo">
      <img src="logo.jpg" alt="Image: Logo" />
    </div>
  );
}

function Navigation() {
  return (
    <ul className="navigation">
      <li><a href="/">Homepage</a></li>
      <li><a href="/team">Team</a></li>
      <li><a href="/services">Services</a></li>
      <li><a href="/contact">Contact</a></li>
    </ul>
  );
}

function Searchbar() {
  return (
    <div className="searchbar">
      <form method="post" action="/search">
        <p>
          <label htmlFor="q">Search:</label>
          <input type="text" id="q" name="q" />
        </p>
        <input type="submit" value="Suchen" />
      </form>
    </div>
  );
}

function Header() {
  return (
    <header>
      <Logo />
      <Navigation />
      <Searchbar />
    </header>
  );
}
```

Even if the code has become longer, we have created a number of improvements.

### Simpler collaboration

All components can and should be saved in their own files so other team members or even complete teams can own one or multiple components. This will spread ownership across teams and make responsibilities clearer. Moreover, it reduces the risk of accidentally overwriting a colleague's file or to continually deal with merge conflicts in Git. Teams become _consumers_ ****of other teams' components which offer a simple interface of their component with possible props.

### Single responsibility principle

Our components now all have clearly defined responsibilities. Each component only fulfills a single task which can be inferred from its name. For example, the logo will be displayed and look the same whereever I use it in my application. If the search bar ever needs amended, one can simply search for the Searchbar.js code and modify the code according to specifications and it will update everywehre it is used. The header component is used as a overarching component whose responsibility it is to hold all the components of the header and inject them wherever the header is used.

### Reusability



