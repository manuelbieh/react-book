# State and Lifecycle Methods

Let's get to what makes working with React really efficient: **State** and the so-called **Lifecycle-Methods**.

As already mentioned in the previous chapter, components can hold, manage and change their own state, the **State**. The principle applies here: **If the state of a component changes, this always triggers a rendering of the component!** This behavior can also be explicitly prevented by using `PureComponent`, which makes sense in some cases. But the principle remains unchanged: A state change triggers rendering of a component and its child components unless they are themselves implemented as `PureComponents` or enclosed by a `React.memo()` call.

This is helpful in that we no longer have to manually call `ReactDOM.render()` whenever we think something has changed on our interface, but the components can decide for themselves instead.

In addition to the state itself, there are also a handful of so-called **Lifecycle methods**. These are methods that can be **optionally** defined in a **class component** and called by React on certain occasions. For example, when a component is mounted for the first time, the component receives new props, or the state within the component has changed.

For the first time since **React 16.8.0**, with the introduction of **Hooks** also **Function Components** can manage their own **State** and react to certain **Lifecycle** events. This chapter deals primarily with class components in which state and lifecycle events are implemented as methods of the class components. Since **Hooks** is a very comprehensive and new concept, I have dedicated a separate chapter to them, in which most of the topics covered in this chapter will once again be described specifically for **Function Components**.

## A first stateful component

The **State** within a class component is available via the instance property `this.state` and is thus encapsulated within a **component**. Neither parent nor child components can easily access the state of another component.

To define an initial state in a component there are two simple ways; a third way, which is a little bit extended in functionality, we will get to know later with the **Lifecycle method** `getDerivedStateFromProps()`.

**Initial State** can be defined by setting the instance property `this.state`. This is possible either in the constructor of a **class component**:

```javascript
class MyComponent extends React.Component {
  constructor(props) {
    this.state = {
      counter: props.counter,
    };
  }
  render() {
    // ...
  }
}
```

... or by defining the state as **ES2017** **Class Property**, which is much shorter, but currently still requires the **Babel plugin** `@babel/plugin-proposal-class-properties` \(before Babel 7: `babel-plugin-transform-class-properties`\):

```javascript
class MyComponent extends React.Component {
  state = {
    counter: this.props.counter,
  };
  render() {
    // ...
  }
}
```

**Create React App** supports the **Class Property Syntax** by default and since many React projects today are completely or at least partially based on the CRA setup or variants of it, this syntax is used and can be used in most projects today. Should this not be the case in a project, I strongly recommend the installation and use of the Babel plugin, as it really saves many unnecessary lines of code in the daily work with React, while at the same time it is set up in just a few minutes.

Once the **State** has been defined, we can access it within the **class component** using `this.state` **reading**. \*\*Reading is a key word here. Because even if it is possible in principle to change the state directly via `this.state`, this should be avoided for various reasons.

## Change the state with this.setState\(\)

To change state, React provides its own method within a **class component**:

```javascript
 this.setState(updatedState)
```

Whenever the state within a component is to be changed, `this.setState()` should be used. Calling `this.setState()` will cause React to execute corresponding **Lifecycle methods** \(like e.g. `componentDidUpdate()`\) and render a component **new!** If we would change the state directly instead, e.g. write `this.state.counter = 1;`, this would have no effect on our component for the time being and everything would look like before, because the rendering process **wouldn't be triggered** \*\*. React then has no knowledge of the change to the State!

The method, however, is somewhat more complex than it might seem at first glance. And so not only is the old state replaced by the new state and a rendering triggered; a lot of other things happen as well. One at a time.

First of all, the function **can accept two different types of arguments**. This is a **object** with new or updated state properties, as well as a **Updater function,** which returns an object or `null` if nothing is to be changed. Existing properties of the same name within the state object are **overwritten**, all others remain  **untouched!** If we want to reset a property in the state, we have to set it explicitly to `null` or `undefined`. The transferred state is therefore always merged with the existing state **\*\*, never** replaced!\*\*

Let's take again our state defined above with a `counter` property, whose initial value for this example is first of all `0`. Now we change the state and want to add a `date` property with the current date. Deliver as object would be our call:

```javascript
this.setState({
  date: new Date(),
});
```

If we use a **Updater function** instead, it would be our call:

```javascript
this.setState(() => {
  return {
    date: new Date(),
  };
});
```

Or short:

```javascript
this.setState(() => ({
  date: new Date(),
});
```

Our component then owns the new state:

```javascript
{
  counter: 0,
  date: new Date(),
}
```

In order to ensure that the current state is always accessed, a **Updater function** should be used that passes the current state as the first parameter. A popular bug that has happened to many developers while working with React is to access `this.state` right after a `setState()` call and wonder that the state is still the old one.

{% hint style="danger" %}
React "collects" successive `setState()` calls and **does not execute them immediately** to avoid unnecessarily frequent and unnecessary component rendering. Fast successive `setState()` calls are executed later in collected form as a **batch process**. This is important to know because we cannot access the newly set state immediately after a `setState()` call using `this.state`.
{% endhint %}

Imagine a situation where our `counter` state is to be increased three times in rapid succession. Intuitively you would probably write the following code:

```javascript
this.setState({ counter: this.state.counter + 1 });
this.setState({ counter: this.state.counter + 1 });
this.setState({ counter: this.state.counter + 1 });
```

What do you think the new state is like when the initial state was `0`? `3`? Wrong. He's "1"! This is where React's **batching mechanism** comes in. To avoid a user interface that updates too quickly, React waits here for the time being. At the end you can roughly equate the above code **expressed in simple terms** from the functional principle\*\* with:

```javascript
this.state = Object.assign(
  this.state, 
  { counter: this.state.counter + 1 },
  { counter: this.state.counter + 1 },
  { counter: this.state.counter + 1 },
);
```

The `counter` property overwrites itself again and again during a batch update, but always takes `this.state.counter` as the base value for the increase by 1. After all state updates have been executed, React then calls the `render()` method of the component again.

If a **Updater function** is used, the most current state is passed to this function as a parameter and we have access to the state valid at the time of the function call:

```javascript
this.setState((state) => ({ counter: state.counter + 1 });
this.setState((state) => ({ counter: state.counter + 1 });
this.setState((state) => ({ counter: state.counter + 1 });
```

In this case we use such an **Updater function** and update the most current value. The value of `this.state.counter` would be `3` as expected, because we access the current state with every call via the `state` parameter. Basically, however, it is recommended that you first generate the required values and then pass them collectively in a single `setState()` call. This ensures that there are no unnecessary `render()` calls in the meantime with the state immediately obsolete again in the next moment.

Should it become necessary to access the newly set state immediately after a `setState()` call, React offers the option of passing a callback function as an optional second parameter. This is called **after** the state has been updated, so that the new state can be safely accessed via `this.state`.

```javascript
this.setState(
  {
    time: new Date().toLocaleTimeString() 
  }, 
  () => {
    console.log('New Time:', this.state.time);
  }
);
```

## Lifecycle methods

**Lifecycle methods** can be implemented as methods of a **class component** and are executed by React at different times during a **component lifecycle** \(hence the name\).

The **Lifecycle** of a component starts at the moment it is **instantiated** or **mounted**, i.e. it is within the `render()` method of a parent component and is actually part of the returned component tree. The lifecycle ends when the component is removed from the tree of elements to be rendered. Meanwhile, there are still **lifecycle methods** that react to **updates** and errors. Or just to the fact that they are now removed \(unmounted"\_\).

### Overview of lifecycle methods

In the following the list of **Lifecycle methods** in the order, when and in which phase they are called by React, if they were defined in a component:

#### Mount phase

The following methods are called **once** when a component is rendered for the first time, that is, when it is added to the DOM for the first time:

* ¶¶constructor\(props\)¶¶
* `static getDerivedStateFromProps(nextProps, prevState)`
* `componentWillMount(nextProps, nextState)` \(deprecated in React 17\)
* `render()`
* ¶ \`componentDidMount\(\)\`\`

#### Update phase

The following methods are called when components receive an update either by entering new props from outside, by changing their own state, or by explicitly calling the `forceUpdate()` method provided by React:

* `componentWillReceiveProps(nextProps)` \(deprecated in React 17\)
* `static getDerivedStateFromProps(nextProps, prevState)`
* \`shouldComponentUpdate\(nextProps, nextState\)\`\`
* `componentWillUpdate(nextProps, nextState)` \(deprecated in React 17\)
* `render()`
* `getSnapshotBeforeUpdate(prevProps, prevState)`
* `componentDidUpdate(prevProps, prevState, snapshot)`

#### Unmount phase

There's only one method here. This is called as soon as the component is removed from the DOM. This is useful to remove event listeners or `setTimeout()`/`setInterval()` calls added when mounting the component:

* ¶ \`componentWillUnmount\(\)\`\`

#### Error handling

Finally, there is a new method added to React 16 that is called whenever an error occurs during rendering, in one of the lifecycle methods, or in the constructor of a **child component**:

* ¶ \`componentDidCatch\(\)\`\`

Components that implement a `componentDidCatch()` method are also called **Error Boundary** and are used to represent an alternative to the erroneous element tree. This can be a high-level component \(related to its position within the component hierarchy\) that basically displays an error page and prompts the user to reload the application should an error occur. However, it can also be a low-level component that only outputs a short error text next to a button if the action triggered by the button has thrown an error.

### Lifecycle methods in practice

Let's take a look at how the **Lifecycle methods** behave in a simple component. For this purpose, we implement a component that changes its own state every second and displays the current time. To do this, when **mounting** the component, i.e. in the `componentDidMount()` method, an interval is started which updates the state of the component, triggering a rendering and displaying the current time again:

```jsx
import React from 'react';
import ReactDOM from 'react-dom';

class Clock extends React.Component {
  state = {
    date: new Date(),
  };

  componentDidMount() {
    this.intervalId = setInterval(() => {
      this.setState(() => ({
        date: new Date(),
      }));
    }, 1000);
  }

  componentWillUnmount() {
    clearTimeout(this.intervalId);
  }

  render() {
    return (
      <div>{this.state.date.toLocaleTimeString()}</div>
    );
  }
}

ReactDOM.render(<Clock />, document.getElementById('root'));
```

Here we see the lifecycle methods `componentDidMount()` and `componentWillUnmount()` in use. We define a **Default-State** with a property `date` that holds an instance of the Date object. When **Mounting** the component \(`componentDidMount()`\) is started, the interval is started via `setInterval()` and its interval ID is stored in the instance property `this.intervalId`. Since the interval calls the `setState()` method every second, the component also regularly causes a rendering, i.e. the `render()` method is called again and displays the current time again.

Since the interval function is basically independent of the React component, and apart from calling the component's `setState()` method, has no connection to it, React does not automatically stop the function's interval call when we no longer need the component. We have to take care of this ourselves and React has the next **Lifecycle method** for us: `componentWillUnmount()`.

This method is called immediately before React removes the component from the DOM and can be used to abort running XHRs, remove event listeners or just end a running function interval. This is exactly what we do here: before removing the component, we call `clearTimeout()` and pass the interval ID we previously stored in the corresponding instance property to the function.

If we should forget this, React's development mode will remind us with a warning at the latest when calling `this.setState()` in an already removed component:

{% hint style="danger" %}
**{\*Warning:** Can't call setState \(or forceUpdate\) on an unmounted component. This is a no-op, but it indicates a memory leak in your application. To fix, cancel all subscriptions and asynchronous tasks in the componentWillUnmount method.  
in clock
{% endhint %}

Unlike in some previous examples, here we call the `ReactDOM.render()` method only once. The component then "takes care of itself" and triggers a render process as soon as its **State** has been updated. This is the usual procedure for developing applications based on React. A single `ReactDOM.render()` call and from there the app manages itself, allows interaction with the user, reacts to state changes and regularly renders the interface anew.

### The interplay of state and props

We have now seen examples of components that process props and components that are **stateful**, i.e. manage their own local state. But there is a lot more to discover. Only the combination of several different components makes React the powerful tool in user interface development that it is. A component can have its own **State** and at the same time pass it on to child components via their **Props**. Not only is it possible to strictly separate business logic from presentation/layout, but it also allows us to develop wonderfully task-based components that each represent only a small part of the application.

When separating business and layout components, react jargon usually refers to **Smart** \(business logic\) and **Dumb** \(layout\) components. **Smart Components** should be entrusted as little as possible or not at all with the representation of the user interface, while **Dumb Components** should be free of any business logic or side effects, i.e. they should concentrate on the pure representation of static values.

Let's look at the interaction of several components in another example:

```jsx
const ShowDate = ({ date }) => (
  <div>Today is {date}</div>
);

const ShowTime = ({ time }) => (
  <div>It's {time} clock</div>
);

class DateTime extends React.Component {
  state = {
    date: new Date(),
  };

  componentDidMount() {
    this.intervalId = setInterval(() => {
      this.setState(() => ({
        date: new Date(),
      }));
    });
  }

  componentWillUnmount() {
    clearInterval(this.intervalId);
  }

  render() {
    return (
      <div>
        <ShowDate date={this.state.date.toLocaleDateString()} />
        <ShowTime time={this.state.date.toLocaleTimeString()} />
      </div>
    )
  }
}

ReactDOM.render(<DateTime />, document.getElementById('root'));
```

Granted: The example is very constructed, but demonstrates easily understandable the interaction of several components. In this sense, the `DateTime` component is our **logic component**: She takes care of the time to "get" and update, but then leaves the **display components** to output by passing the date \(`ShowDate`\) or the time \(`ShowTime`\) through the props. The display components themselves are implemented as simple **Function Components**, since a **Class component** is simply not necessary here and would only generate unnecessary overhead.

### The role of lifecycle methods in the interaction of components

At the beginning I have mentioned some more **Lifecycle methods** besides the `componentDidMount()` and `componentWillMount()` and `componentWillMount()` used so far in the examples. These are also taken into account, if implemented in a **class component**, for the various events of React.

For this purpose we want to create an exercise component which outputs the different **Lifecycle methods** as a debug message in the browser console. Strictly speaking, there are two components, one of which serves as a parent component, the other as a child component, which gets props handed in by its parent component \(and in this case simply ignored\).

```jsx
import React from 'react';
import ReactDOM from 'react-dom';

const log = (method, component) => {
  console.log(`[${component}]`, method);
};

class ParentComponent extends React.Component {
  state = {};

  constructor(props) {
    super(props);
    log('constructor', 'parent');
  }

  static getDerivedStateFromProps() {
    log('getDerivedStateFromProps', 'parent');
    return zero;
  }

  componentDidMount() {
    log('componentDidMount', 'parent');
    this.intervalId = setTimeout(() => {
      log('state update', 'parent');
      this.setState(() => ({
        time: new Date().toLocaleTimeString(),
      }));
    }, 2000);
  }

  shouldComponentUpdate() {
    log('shouldComponentUpdate', 'parent');
    return true;
  }

  getSnapshotBeforeUpdate() {
    log('getSnapshotBeforeUpdate', 'parent');
    return zero;
  }

  componentDidUpdate() {
    log('componentDidUpdate', 'parent')
  }

  componentWillUnmount() {
    log('componentWillUnmount', 'parent');
    clearInterval(this.intervalId);
  }

  render() {
    log('render', 'parent');
    return <ChildComponent time={this.state.time} />;
  }
}

class ChildComponent extends React.Component {
  state = {};

  constructor(props) {
    super(props);
    log('constructor', 'child');
  }

  static getDerivedStateFromProps() {
    log('getDerivedStateFromProps', 'child');
    return zero;
  }

  componentDidMount() {
    log('componentDidMount', 'child');
  }

  shouldComponentUpdate() {
    log('shouldComponentUpdate', 'child');
    return true;
  }

  getSnapshotBeforeUpdate() {
    log('getSnapshotBeforeUpdate', 'child');
    return zero;
  }

  componentDidUpdate() {
    log('componentDidUpdate', 'child');
  }

  componentWillUnmount() {
    log('componentWillUnmount', 'child');
  }

  render() {
    log('render', 'child');
    return <div>{this.props.time}</div>;
  }
}

ReactDOM.render(<ParentComponent />, document.getElementById('root'));
```

These two components lead reliably \(i.e. reproducibly\) to the following output:

```text
[parent] constructor
[parent] getDerivedStateFromProps
[parent] render
[child] constructor
[child] getDerivedStateFromProps
[child] render
[child] componentDidMount
[parent] componentDidMount
[parent] state update
[parent] shouldComponentUpdate
[parent] render
[child] getDerivedStateFromProps
[child] shouldComponentUpdate
[child] render
[child] getSnapshotBeforeUpdate
[parent] getSnapshotBeforeUpdate
[child] componentDidUpdate
[parent] componentDidUpdate
[parent] componentWillUnmount
[child] componentWillUnmount
```

Oh, wow. So there's a lot going on here. So let's go line by line and start with the mount phase.

#### `constructor(props)`

First, the `constructor` of the `ParentComponent` component is always called. React goes from "outside to inside" here. So components higher up in the component hierarchy are instantiated first, then their `render()` method is called. This is necessary because otherwise React wouldn't know which child components to process and consider, since React only considers child components that are actually returned by the `render()` method of their parent component and executes the **Lifecycle methods** for them.

The constructor gets as parameter the **Props** of the component and should pass them via `super(props)` to its parent class \(usually this is the `React.Component` or `React.PureComponent` class\), because `this.props` is otherwise `undefined` in the constructor, which can lead to unwanted behavior and bugs.

In many cases, the constructor is no longer necessary if the Babel plugin **"Class Properties "** is used and both state and instance methods are implemented as class properties. If this is not the case, the constructor is the place to set an initial state \(e.g. `this.state = { }`\) and to bind instance methods to the respective class instance using `.bind()` \(e.g. `this.handleClick = this.handleClick.bind(this)`\). The latter is necessary, since instance methods would otherwise lose their reference to the component when used as event listeners within JSX, and `this` would not refer to the instance of the component.

#### `static getDerivedStateFromProps(nextProps, prevState)`

The constructor is followed by the static `getDerivedStateFromProps()` method. Since this is a **static** method \(and as such must be marked with the keyword `static` like in our example above\), it has no access to the instance of the component using `this`. The method is used to calculate the **next state** of the component based on the props submitted to the component and, if applicable, the last state. This is returned as an object from the method. If no changes to the state are necessary, `null` should be returned. The behavior is identical to `this.setState()`, and only the part of the state that is in the returned object is updated. These properties are merged with the **last state** to form a **new state**.

The method has been controversially discussed within the community because it replaces the **Lifecycle method** `componentWillReceiveProps()` now marked as **deprecated**, but does not allow access to the instance of the component. The React developers have justified this step with the fact that it can lead to unwanted behavior during the revised asynchronous rendering of components and have declared the method \(as well as `componentWillMount()` and `componentWillUpdate()`\) as "unsafe". The term explicitly does not mean security \(in the sense of security gaps\) but only the fact that components that use these lifecycle methods can cause 17 bugs and other undesirable effects in React.

The `getDerivedStateFromProps()` method should also remain free of side effects \(e.g. do not initiate XHRequests\) but only calculate the new state of the component instance based on the current props or **derive** \(= derive\). Unlike the constructor, the method is called not only when the component is mounted \(that is, when it is "mounted" in the DOM\), but also when the component receives props again. These must not necessarily have changed in content.

#### `render()`

Once the instance of a component has been created and its state derived, React already calls the `render()` method. The `render()` method describes our user interface and thus also which child components should be rendered. In our example above, we have only one child component: the `ChildComponent` component.

And so the game starts again: `constructor()`, `getDerivedStateFromProps()` and then the `render()` method of the child component is called. So the exact same behavior as with the `ParentComponent` component. The child component in the example has no other child components of its own. If it had these, the lifecycle methods above would be executed here as well, until React meets a component at this branch, which doesn't return any React components anymore, but DOM elements like `div`, `p`, `section`, `span`, etc.. \(of course also combinations of these\) or `null` or even an array, which again also contains no components.

#### `componentDidMount()`

If such a component is reached, it is the `componentDidMount()` method's turn. This method is called once a component and all its child components have been rendered. From this moment on, the DOM node of the component can be accessed if necessary. The method is also the right place to start e.g. timeouts or intervals or to initiate network requests e.g. via XHR/Fetch.

The method is called "from inside to outside". So it is the child components' turn as soon as they have been rendered, then the parent components' turn. So we can see in the log output above that the `componentDidMount()` method of the `ChildComponent` is called first, only after that the `ParentComponent` is called.

In our example we start a `setTimeout()` call once in the `ParentComponent` which changes the state of the component after 2000 milliseconds to demonstrate the lifecycle methods which are called when updating a component. The mount phase is now complete and any further changes to the state of the mounted components will cause React to call the lifecycle methods from the update phase. This is the case here after 2000 milliseconds when the ParentComponent modifies its own state using `this.setState()`.

#### `shouldComponentUpdate(nextProps, nextState)`

If a component is updated, this is always the case if the component changes its state or receives props from outside, `shouldComponentUpdate()` is called. But be careful, here there is a difference depending on whether the props have changed or the state: if a component gets new props from outside, `getDerivedStateFromProps()` is called first.

The `shouldComponentUpdate()` method serves as a "help" to tell React if expensive rendering is necessary at all. The method gets the **next props** and the **next state** as parameters and can decide on their basis whether a rendering should be executed. The method must either return `true`, which will trigger a rendering, or `false`, which will prevent both `componentDidUpdate()` , `getSnapshotBeforeUpdate()` and `render()` from being called in this component.

In complex applications it is often the case that the update cycle is only triggered because something has changed in another part of the application, in a parent component, but this change is irrelevant for child components. The `shouldComponentUpdate()` method is very helpful when it comes to optimizing rendering performance, as it prevents unnecessary renderings.

If we would return `false` from the `shouldComponentUpdate()` method in our `ParentComponent` above, our log output would be much shorter: lines 14-18 would be missing. The component would not be re-rendered, a call to the `render()` method would not take place, so the `ChildComponent` would not be re-rendered and consequently its update methods would not be called.

In the code example, however, we return `true`. This also calls the `render()` method of the `ParentComponent`. This renders again the `ChildComponent`, which gets the updated **State** of the `ParentComponent` in its **Props** and already we are in the update cycle of the `ChildComponent`.

As with the mount cycle, `getDerivedStateFromProps()` is called to derive a new state based on the new props. Then `shouldComponentUpdate()` is called here as well. For example, we could check whether the props relevant to this component have changed at all, and if they haven't, we could save rendering the component by simply returning `false`. We don't, so the next step is the mandatory call to the `render()` function. The next **Lifecycle method** is called directly after this.

#### `getSnapshotBeforeUpdate(prevProps, prevState)`

This method is quite new and was introduced in React 16.3.0 along with `getDerivedStateFromProps()` to meet the requirements of the new asynchronous rendering in React. It gets the **last props** and the **last state** passed and has last access to the current state of the HTML document, more precisely the DOM, before React applies the possible changes from the last `render()` call.

This can be useful, for example, if you want to remember the current scroll position in a long table or list to be able to jump back to the corresponding line when updating the list. The method can return any value or `null` here. What `getSnapshotBeforeUpdate()` returns is then passed as a third parameter to the next lifecycle method, `componentDidUpdate()`.

From my experience, this method is very rarely used, probably the rarest of all **Lifecycle methods**, because when working with React you rarely have to access DOM elements directly. Many things that would have been done with the DOM API in an imperative approach can be solved here directly in the abstract component tree, i.e. via JSX.

#### `componentDidUpdate(prevProps, prevState, snapshot)`

The last method from the update cycle is `componentDidUpdate()`. This is called after `getDerivedStateFromProps()` has derived the new state, after `shouldComponentUpdate()` has returned `true` if implemented, and after `getSnapshotBeforeUpdate()` has taken a snapshot of the last state of the DOM.

The method gets the **last props** and the **last state** as parameters, i.e. the respective props and the respective state before the component was updated, and, if the component has a `getSnapshotBeforeUpdate()` method, its return value as third parameter.

Similar to `componentDidMount()` also `componentDidUpdate()` is resolved "from inside to outside", so the `componentDidMount()`-methods of the child component\(n\) are called first, then those of the parent component\(n\). This method is the ideal place to trigger side effects, e.g. start XHRs if certain properties of the component have changed. This can be determined by a simple comparison of the current props with the last props passed as parameters or the current state with the last state.

It is also safe to access the **current DOM** in this component, should this ever be necessary. By the time this method is called, React has already made the necessary changes resulting from the possibly changed JSX from the `render()` method and transferred them to the DOM.

And that concludes the **Update cycle** as well as the **Mount cycle**. While the mount cycle **is always run only once**, i.e. when a component is **rendered for the first time**, the update cycle can be triggered as often as you like as long as the component is mounted and will always run as soon as a component changes its state or moves to new props.

#### `componentWillUnmount()`

Admittedly, I cheated a bit in the log above, because the `componentWillUnmount()` method is always executed \(and only then\) when a component is completely removed from the DOM. It never will in the corresponding code example. A component is considered unmounted if it is explicitly removed by calling `ReactDOM.unmountComponentAtNode()` \(this applies especially to mount nodes\) or if it is implicitly no longer returned from the render\(\) method of a parent component.

Whenever this happens, the `componentWillUnmount()` method of a component is called; of course, as with all methods except `render()`, only if it has been implemented. This **Lifecycle method** is essential when it comes to "cleaning up". Here you can and **should** call up all the functions that are required so that the component does not "leave tracks". These can be outstanding timeouts \(`setTimeout`\) or continuous intervals \(`setInterval`\) but also DOM modifications, which were made outside the own component JSX, still running network connections or XHR/Fetch calls or also own event listeners, which were added by means of the DOM API method `Element.addEventListener()`.

Event Listener. Good cue. We'll take care of that in the next chapter, because in most cases the use of `addEventListener()` in React is no longer necessary, since React comes with its own event system for a better overview.

### Diagram of lifecycle methods

![diagram of the different lifecycle methods in their respective phases \(CC0 Dan Abramov\)](../.gitbook/assets/lifecycle-methods-2.jpg)

