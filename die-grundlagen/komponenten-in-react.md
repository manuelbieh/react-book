# Components in React

## The two manifestations of React Components

We already implemented a first simple `HelloWorld` component when we jumped into the cold water. But of course this was deliberately a very simple component, which was not very practical and did not contain everything React offered us. But it served as a good first illustration to get to know the basic functionality of **React** and **React components**.

The principle of **components** is simple: a **component** allows to divide complex user interfaces into small pieces. These are ideally reusable, insulated and self-contained. They process any input from outside in the form of so-called **Props** \(engl. for "Properties"\) and finally describe on the basis of their `render()` function what appears on the screen.

Components can occur roughly in two different variants: in the form of a simple function \(**Function Component**\), and **Class Components**, which represent an ordinary ES2015 class. Until the introduction of the React **Hooks** it was not possible to manage a local state in **Function Components**, so in some places the term **Stateless Functional Component** is still used, which comes from a time when **Function Components** could not hold a state. How **Hooks** work in detail and how State can be used in **Function Components** since React 16.8.0 is explained in detail in the chapter about **Hooks**.

### Function Components

The simplest way to define a component in React is certainly the **Function Component**, which, as the name suggests, is actually just an ordinary JavaScript function:

```jsx
function Hello(props) {
  return <div>Hello {props.name}</div>;
}
```

This function fulfills all criteria of a valid **React component**: It has as `return` value an explicit `null` \(`undefined` is **not** valid!\) or a valid `React.element` \(here in form of **JSX**\) and it receives a `props` object as first function argument, where even this is optional and can be `null` as well.

### Class Components / Class Components

The second possibility how a **React component** can be created is shown in the example below: **Class Components** or in good German **Class Components**. These consist of an ES2015 class that derives from the `React.Component` or `React.PureComponent`\(more on this later\) class and have at least one method called `render()`:

```jsx
class Hello extends React.Component {
  render() {
    return <div>Hello {this.props.name}</div>;
  }
}
```

Important difference here: While a **Function Component** gets its **Props** passed as function arguments, the `render()`-method of a **class component** itself doesn't get any arguments passed, but the **Props** can only be accessed via the instance property `this.props`!

The two above components result in a completely identical output!

{% hint style="info" %}
A criterion that both types of components have in common is that the `displayName`, i.e. the name of a valid component, always begins with a **uppercase letter**. The rest of the name can consist of uppercase letters or lowercase letters, the only important thing is that the first letter is always an uppercase letter!

If the name of a component starts with a lowercase letter, React treats it as a pure DOM element instead. `section` would interpret React as a DOM element, while a component of its own may well have the name `Section` and would be correctly distinguished from the `section` DOM element by its capital letter at the beginning of React.
{% endhint %}

How we work with the **State** within the **Components**, modify it and make it our own is very complex, which is why a separate chapter is dedicated to this topic. This follows directly after this one and I would recommend to finish this chapter first to understand the functionality of components before going any deeper.

### Special case PureComponent

A special form of the **Class Component** is the **Pure Component**. This inherits from `React.PureComponent` and works basically like the ordinary class component derived from `React.Component`. With the difference that React only renders this type of component if its **props** or **state** has changed compared to the previous render phase. It can be used to optimize performance.

The class does a superficial \(_"shallow"_\) comparison to see if the **references** are still identical to the previous rendering and **not** if their **values** have changed themselves. This means that a **Pure Component** may render again even if the values are identical but the references have changed, for example because a new Arrow function was created.

This can best be explained using a short example:

```jsx
const logFunction = (message) => console.log(message);

class App extends React.Component {
  render() {
    return <MyComponent logger={changeHandler} />;
  }
}
```

Here we define a function `logFunction` outside the class. The fictitious `MyComponent` component gets the **reference** to the `logFunction` as `logger` prop here. The **Identity** of the **Reference** remains here with each new rendering and a **Pure Component** would not be rendered again, as long as its own state has not changed, because the props here in the rendering phase match exactly with those from the previous rendering, so they are identical.

In the following example, however, the **Pure Component** would also be re-rendered:

```jsx
class App extends React.Component {
  render() {
    return <MyComponent logger={(message) => console.log(message)} />;
  }
}
```

Although we always pass the same function to the `MyComponent`, we also create a new function identity for the interpreter with each new rendering. Thus the reference to the old function is lost and the _Shallow-Comparison_, i.e. the superficial comparison between the previous and the new props fails, so it is different and triggers a rendering.

This applies equally to objects and arrays:

```jsx
<MyComponent
  logConfig={{ logLevel: 'info' }}
  logEntries={['Message 1', 'Message 2']}
/>
```

A rendering would be triggered here because the `logConfig` object or `logEntries` array would also be regenerated on the spot with every new rendering.

### "Pure" Function Components

**Class components** derive from the classes `Component` or `PureComponent` and we determine by selecting the class from which we derive whether the component should be rendered each time a higher-level component in the hierarchy is changed. We do not have this option with **Function Components**, which are only simple JavaScript functions. Since **React 16.6.0\*\*** React also offers the possibility of rendering optimization in **Function Components** with the new wrapper function `React.memo()`. For this purpose, it is simply placed around the corresponding function:

```jsx
const MyComponent = React.memo((props) => {
  return <p>I will only be re-rendered if my props change</p>;
});
```

The **Function Component** then behaves like a comparable **Class component** derived from the `React.PureComponent` class.

If you have become curious, you can also let off steam with the following demo for easier understanding:

```jsx
import React from 'react';
import ReactDOM from 'react-dom';

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

ReactDOM.render(<App />, document.getElementById('root'));
```

Here the `App` component automatically triggers a rendering every 0.2 seconds without passing **Props** to its child components. Price question: Which two components are each rendered with new and which two are not?

## Component Composition - multiple components in one

Previously, our example components only output DOM elements. **React components** can also contain other React components. It is only important that the component is in the same scope, i.e. that it was either defined directly in the same scope or, when using CommonJS or ES modules, was imported into the current file using `require()` or `import`.

**An example:**

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

The `<MyApp>` component returns a `<div>` that uses the `Hello` component from the previous example twice to greet Manuel and Tom. The result:

```jsx
<div>
  <div>Hi Manuel</div>
  <div>Hello Tom</div>
</div>
```

Important: A component may only ever return a single **Root element**! This may be:

* a single React element:

```jsx
<Hello name="Manuel />
```

* Also in nested form, as long as there is only a single element on the outer layer:

```jsx
<Parent>
  <Child />
</Parent>
```

* a DOM-element \(also this element may be nested and contain other elements\):

```jsx
<div>...</div>
```

* ...or self-closing:

```jsx
<img src="logo.jpg" alt="Picture: Logo" />
```

* Or even:

```javascript
nil;
```

... but \*\*not `undefined`!

Since **React 16.0.0\*\*** these may also be:

* an array containing valid return values \(s.o.\):

```jsx
[<div key="1">Hello</div>, <Hello key="2" name="Manuel" />];
```

* A simple string:

```javascript
Hello World
```

* or a so-called "fragment" - a kind of special "component" that does not itself appear in the rendered output and can serve as a container if, on the other hand, you would violate the rule of returning only a root element from the function or creating invalid HTML:

```jsx
<React.Fragment>
  <li>1</li>
  <li>2</li>
  <li>3</li>
</React.Fragment>
```

With **Transpiling** with **Babel from version 7** you can also use the `Fragment` short form, which consists of an empty opening and closing element:

```jsx
<>
  <li>1</li>
  <li>2</li>
  <li>3</li>
</>
```

Components can be put together as desired \(_"composed"_\). It often makes sense to divide large and complex components into individual, smaller and clearer components in order to make them easier to understand and ideally even reusable. This is often a living process, where from a certain point it becomes apparent that a subdivision into several individual components might be useful.

## Divide components - keep an overview

Let's take a look at an exemplary header bar that contains a logo, a navigation and a search bar. Not an unusual pattern, then, if you look at ordinary web applications:

```jsx
function Header() {
  return (
    <header>
      <div className="logo">
        <img src="logo.jpg" alt="Image: Logo" />
      </div>
      <ul className="navigation">
        <li>
          <a href="/">Homepage</a>
        </li>
        <li>
          <a href="/team">Team</a>
        </li>
        <li>
          <a href="/services">Services</a>
        </li>
        <li>
          <a href="/contact">Contact</a>
        </li>
      </ul>
      <div className="searchbar">
        <form method="post" action="/search">
          <p>
            <label htmlFor="q">Search:</label>
            <input type="text" id="q" name="q" />
          </p>
          <input type="submit" value="Search" />
        </form>
      </div>
    </header>
  );
}
```

We already know that components in React can easily include other components, and that this component-based approach is consistent with React's idea and mindset. So what's the best thing to do here? Right: we divide our already relatively large, confusing component into several smaller canapés, each of which serves only one specific purpose.

There's the logo, which we can certainly use again elsewhere. The navigation can possibly be used again in a sitemap in addition to the header. Also the search bar should perhaps sometime no longer only be used in the header, but perhaps also on the search result page itself.

Speaking in terms of components, we then end up with the following end result:

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
      <li>
        <a href="/">Homepage</a>
      </li>
      <li>
        <a href="/team">Team</a>
      </li>
      <li>
        <a href="/services">Services</a>
      </li>
      <li>
        <a href="/contact">Contact</a>
      </li>
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
        <input type="submit" value="Search" />
      </form>
    </div>
  );
}

function Header() {
  return (
    <header>
      <Logo />
      <Navigation />
      <Searchable />
    </header>
  );
}
```

Even though the code has now become longer, we still have some great advantages.

### Easier collaboration

All components can \(and should!\) be stored in a separate file, which makes working in a team immensely easier. Thus, each team member or even individual teams within a large project team could be primarily responsible for one or more components \("Ownership take over"\) and make changes in these, while the risk of overwriting a colleague's changes or later having to resolve merge conflicts in Git decreases immensely. Teams become _consumers_ of components of other teams that provide a simple interface for their components based on available props.

### Single Responsibility Principle

We now also have "speaking" components, each of which has a clearly defined task that is directly visible by its name. The logo shows me the same logo wherever it is used. If I want to make a change to the search bar later, I search specifically for Searchbar.js and change it according to my new requirements. The header component serves as a parent component that is itself responsible for containing all its components and bringing them with it wherever it is used.

### Reusability

And last but not least, we have created reusability by the way. If I want to use the logo not only in the header but also in the footer, nothing prevents me from using the same component in my footer component. If I have different page areas with different layouts, but they all represent the same header, I can use my slim and clear header component wherever I need it. The consumer of a component does not even need to know which individual components it consists of. It is sufficient to just import the desired component, as it takes care of its own dependencies.

## Props - the "data recipients" of a component

Now I've already written so much about **props**. So it's high time to unveil the secret and go into it more closely. So what's props?

The **props** allow components to receive any type of data and access this data within the **component**. If we think back to our **functional component**, we may remember that in this case the **props** were actually passed to the function as a very common argument. The principle is similar with a **class component**, with the difference that the **props** are passed into the component via the **Constructor** of the class and are available via `this.props` within the class instance, instead of via a function argument, as is the case with functional components. This is what React does when parsing the `createElement()` calls.

It is important to note that whenever a component receives new **props** from the outside, this triggers a rendering of the component! This behavior can be explicitly suppressed using the `shouldComponentUpdate()` **Lifecycle method**, but there is more on this in the following chapter on **State and Lifecycle methods**. First of all the general principle is important: if a **component** receives new **props** from outside, this causes React to render a **component** together with its child components anew.

### Props are readonly within a component

Regardless of how the **props** land in whatever kind of component, they have one thing in common: they are readonly **within the component** always, so \(and can\) only be read, not modified! The connoisseur also speaks here of **Immutability** or **Immutable Objects**. To work with variable data, the React **State** comes into play later. But one thing at a time.

If a function does not modify its input and also has no external dependency, then in functional programming one speaks of a pure function \(**Pure Function**\) and the idea behind it is quite simple: This is to ensure that a function is self-contained and therefore remains unimpressed if something changes outside the function. The function receives all required parameters, is free of side effects \(engl: **Side Effects**\) and thus always achieves exactly identical output with the same input values. \*\*Same input, same output.

In other words, no matter which variables outside the function change their value, no matter how often other functions are called elsewhere: If a **Pure Function** gets the same parameters as before, it returns the same result as before. Always and without exception.

Why is that important? Well, React follows the principle of **Pure Functions** with its components. If a component receives the same props from outside and the state of the component is the same, the output should always be identical.

### Pure Functions in detail

Since the principle of **Pure Functions** is a fundamental one when working with React, I would like to take a closer look at these examples. This is mainly about theory, which certainly sounds more complicated than the practical work with React will be later on. Nevertheless I would like to mention them for a better understanding.

#### Example of a simple pure function

```javascript
function pureDouble(number) {
  return number * 2;
}
```

Our first simple function gets a number, doubles it and returns the result. No matter if I call function 1, 10 or 250 times: If I give the function e.g. a `5` as value, I get a `10` back. Always and without exception. Same input, same output.

#### Example of an Impure Function

```javascript
function impureCalculation(number) {
  return number + window.outerWidth;
}
```

The second function is no longer _pure_, because it does not always reliably deliver the same output, even if its input is identical. Currently my browser window is 1920 pixels wide. If I call the function with `10` as argument, I get `1930` back \(`10 + 1920`\). Now I reduce the window to 1280 pixels and call the function again, with exactly the same `10` as argument on I still get a different result \(`1290`\) than the first time. It is therefore not a **Pure Function**.

One way to make this function "pure" would be to give it my window width as another function argument:

```javascript
function pureCalculation(number, outerWidth) {
  return number + outerWidth;
}
```

So the function still returns a result when `pureCalculation(10, window.outerWidth)` is called which depends on my window width, but the function is _"pure"_ because it still returns the same output with the same input. This can be more easily understood by reducing the function to its essential properties:

```javascript
function pureSum(number1, number2) {
  return number1 + number2;
}
```

**Same input, same output.**

#### Further example of an Impure Function

Imagine we want to implement a function that receives as input an object with parameters to a car.

```javascript
var car = { speed: 0, seats: 5 };
function accelerate(car) {
  car.speed += 1;
  return car;
}
```

The above example is also a function that is not "pure", because it modifies its input value and thus has a different result as output value at the second call than at the first call:

```javascript
console.log(accelerate(car));
// {speed: 1, seats: 5}

console.log(accelerate(car));
// {speed: 2, seats: 5}
```

So how do we now ensure that our last example also becomes "pure"? By no longer modifying the input value and instead creating a new object based on the input value each time, and returning this new object from the function:

```javascript
var car = { speed: 0 };
function accelerate(car) {
  return {
    speed: car.speed + 1,
  };
}
```

New result:

```javascript
console.log(accelerate(car));
// {speed: 1}

console.log(accelerate(car));
// {speed: 1}
```

\*\*Same input, same output, we're pure.

You may be wondering why I'm telling you this now and why I'm boring you with boring theory here when you're just trying to learn React \(at least that's what I'd think if I imagined reading this book for that reason, too\).

React is a very liberal library that gives the developer a lot of freedom. But one thing is the top priority and React doesn't really have any fun with that: \*\*With regard to their props, components must behave like "Pure Functions" and always generate the same output for the same props!

If you don't stick to this, working with React can lead to very peculiar effects, unwanted and incomprehensible results, and make your life hell by bugfixing. And that's exactly why you're learning React: because you want a simple yet incredibly powerful tool that allows you to develop professional user interfaces in an incredibly fast time, without driving yourself crazy. React offers you all this as long as you stick to this rule!

At the same time, however, this has the very pleasant side effect that components can usually also be tested very easily.

So and what exactly does the _"readonly within a component"_ mean? This is explained quite quickly with our new knowledge about "Pure Functions": no matter how I access the props in the component, whether directly via the `props` argument of a **Function Component**, via the `constructor()` in a **class component** or anywhere else within a **class component** via `this.props`: I can and may \(and should! and wants!!\) not change the value of the submitted **props**.

The situation is different, of course **outside** of components. Here I can easily change the value \(provided, of course, we're not in another component that the prop we're trying to modify just got itself handed in\).

#### What is not possible

```jsx
function Example(props) {
  props.number = props.number + 1;
  props.fullName = [props.firstName, props.lastName].join(' ');
  return (
    <div>
      ({props.number}) {props.fullName}{' '}
    </div>
  );
}

ReactDOM.render(
  <Example number={5} firstName="Manuel" lastName="Bieh" />,
  document.getElementById('app')
);
```

**output:**

{% hint style="danger" %}
TypeError: Cannot add property number, object is not extensible
{% endhint %}

Here I try to directly change the `number` and `fullName` props within my `Example` component, which of course can't work, because we learned that props are **readonly** basically.

#### What is possible though

Sometimes, however, I would like to derive a new value from a prop submitted here. That's no problem at all, React 16.3.0 even offers a comprehensive function `getDerivedStateFromProps()`, which I'll discuss in detail in the next chapter.

But if I just want to display a value that is derived from the prop I receive as a component, I can do that by just adjusting the output based on the prop without writing back.

```jsx
var React = require('react');
var ReactDOM = require('react-dom');

function Example(props) {
  return (
    <div>
      ({props.number + 1}) {[props.firstName, props.lastName].join(' ')}
    </div>
  );
}

ReactDOM.render(
  <Example number={5} firstName="Manuel" lastName="Bieh" />,
  document.getElementById('app')
);
```

**output:**

```jsx
<div>(6) Manuel Bieh</div>
```

In this case I only modify the output based on the `props`, but not the `props` object itself. That's not a problem at all.

#### What is also possible

Now it remains to be finally clarified, how props can be changed outside of a component, because until now there was only talk of props not being changed within a component.

This, too, can best be explained by means of a concrete, but still rather abstract example:

```jsx
var React = require('react');
var ReactDOM = require('react-dom');

var renderCounter = 0;
setInterval(function() {
  renderCounter++;
  renderApp();
}, 2000);

const App = (props) => {
  return <div>{props.renderCounter}</div>;
};

function renderApp() {
  ReactDOM.render(
    <App renderCounter={renderCounter} />,
    document.getElementById('app')
  );
}

renderApp();
```

What's going on here? First of all we set a variable `renderCounter` to the initial value `0`. This variable counts to us how often we render our `App` component, or more precisely how often we end up calling the `ReactDOM.render()` function, which then ensures that the `App` component is rendered again each time it is called.

Then we start an interval, which executes the mentioned function regularly every 2000 milliseconds. The interval not only executes the function every 2 seconds, but also counts up our `renderCounter` variable by 1 at the same time. What's happening here is very exciting: We're modifying the `renderCounter` prop of our app **"from the outside "**.

The component itself remains completely "pure". It is called with:

```jsx
<App renderCounter={5} />
```

she gives us back as a result:

```markup
<div>5</div>
```

No matter how often the component has actually been rendered. \*\*Same input, same output.

Within our component we are and remain _pure_. We don't modify the input value and we don't have any direct external dependencies in the component that could affect our rendering result. The value is only changed outside of our component and entered into the **component**, which does not interest us at this point, because it is important for us that our component continues to deliver the same result with the same props. And that is undoubtedly given here. Whoever modifies the props outside our component, how often and in what form, is completely irrelevant to us, as long as we do not do this ourselves inside our component. Okay, principle understood?

#### Props are an abstracted functional argument

Since props, reduced to the essentials, are nothing more than a functional argument, they can also appear in their various forms. Anything that functions or constructors in JavaScript accept as an argument can also be used as a value for a prop. From a simple string to objects, functions or even other react elements \(which, as we already know, are nothing else than a call to `createElement()`\) behind the scenes\) this can be almost anything, as long as it is a valid expression.

```jsx
<MyComponent
  counter={3}
  text="example"
  showStatus={true}
  config={{ uppercase: true }}
  biggerNumber={Math.max(27, 35)}
  arbitraryNumbers={[1, 4, 28, 347, 1538]}
  dateObject={Date}
  dateInstance={new Date()}
  icon={
    <svg x="0px" y="0px" width="32px" height="32px">
      <circle fill="#CC3300" cx="16" cy="16" r="16" />
    </svg>
  }
  callMe={() => {
    console.log('Somebody called me');
  }}
/>
```

Even though most props make little sense here and only serve for illustration, they are syntactically correct JSX, demonstrate how powerful they are and in which different forms they can appear.

### Props are not limited to one nesting level

A component that receives props can easily pass them on to child elements. On the one hand this can be helpful if you have to divide large components into several smaller components and pass on certain props to child components, but on the other hand it can sometimes lead to complex applications where it is difficult to see where the exact origin of a prop is and where I have to start looking if I want to change the value of a prop.

```jsx
function User(props) {
  return (
    <div>
      <h1>{props.name}</h1>
      <UserImage image={props.image} />
      <ListOfPosts items={props.posts} />
    </div>
  );
}

ReactDOM.render(
  <User name={user.name} image={user.image} posts={user.posts} />,
  document.getElementById('app')
);
```

### The most important points at a glance

{% hint style="info" %}

Components must behave as **Pure Functions** with regard to their props and always generate the same output for the same props.

* Props within a component are always to be regarded as **readonly**.
* Components can get any number of props\*\* passed to them
* In JSX, you pass props in a similar form to HTML attributes.
* Unlike HTML, JSX allows various types of values. Values that are not of type String are enclosed in **curly braces**.
* Props can accept **all JavaScript expressions** \("Expressions"\) as value
* Received props can be passed on to child elements as many levels deep in the component tree as you like

