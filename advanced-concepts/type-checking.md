# Typechecking with PropTypes, Flow and TypeScript

**Typechecking** is a simple method to avoid potential errors in an application. The principle here is quite simple: Components should be "pure", as we already learned in the introduction. So they should not trigger any page effects and above all they should also generate the **identical output** for the **same input parameters** \(which are **props** and their derived **state** in the case of components\).

This means that it should be as predictable and as strict as possible which **props** can be entered into a component and which are processed by it. To ensure this, we can make use of the so-called **Typechecking**. JavaScript is basically an untyped language. A variable that was originally a **String** can be converted into a **Number** or even a **Object** without any problems for the JavaScript interpreter.

Although this can be very convenient during development because we don't have to commit ourselves, it opens the door for some annoying mistakes and makes it necessary to check the correct type manually on a regular basis. For example, if we want to access a deeply nested property `user.settings.notifications.newMessages`, we should first check whether `user` is an object at all and not `null`, then we should check whether the same applies to `settings`, and so on. Otherwise we might be dealing with a type error:

{% hint style="danger" %}
TypeError: Cannot read property 'settings' of undefined
{% endhint %}

**Typechecking** can therefore help us to detect such potential errors in advance. In addition to **Flow** and **TypeScript**, which allow static typing, there is also a quite simple React solution with the so-called **PropTypes**. While **Flow** and **TypeScript** generally allow static typing in JavaScript, React **PropTypes** are limited to React components only and are not applied outside of components. So if you like static typing, you should take a look at **Flow** or **TypeScript**.

## PropTypes

**PropTypes** go back to very early versions of React, long before React reached its current popularity, and in React 15.5 were swapped out of the core and into a separate `prop-types` package. While you could previously define your **PropTypes** directly in the core library using e.g. `React.PropTypes.string`, access now takes place using the previously imported `PropTypes` module: `PropTypes.string`.

This also means that the module must first be installed as `devDependency`. On the command line a simple one is enough:

```bash
npm install --save-dev prop-types
```

```bash
yarn add --dev prop-types
```

**PropTypes** serve as a kind of interface and determine which form or type a prop may take and whether it is optional or required. If there are deviations, React **in development mode** informs us. With a correctly released application that uses the Production version of React and/or was built with the environment variable `process.env.NODE_ENV=production`, we get these warnings **not** visible anymore!

But what does the use of **PropTypes** look like now? Here we have to distinguish between the **Class Component** and the **Stateless Functional Component**.

For the **Class Component**, the **PropTypes** are a static property `propTypes` of the component:

```jsx
import React from 'react';
import ReactDOM from 'react-dom';
import PropTypes from 'prop-types';

class EventOverview extends React.Component {
  static propTypes = {
    date: PropTypes.instanceOf(Date).isRequired,
    description: PropTypes.string,
    ticketsUrl: PropTypes.string,
    title: PropTypes.string.isRequired
  };

  render() {
    const { date, description, ticketUrl, title } = this.props;
    return (
      <div>
        <h1>{title}</h1>
        <h2>{date.toLocaleString()}</h2>
        {description &&& (
          <div className="description">{description}</div>
        )}
        {ticketsUrl && <a href={ticketsUrl}>Tickets!</a>}
      </div>
    );
  }
}

ReactDOM.render(
  <EventOverview date={new Date()} title="Learn and understand React" />,
  document.getElementById('root')
);
```

In this example, we would like to display an overview of an event. We define that the `EventOverview` component must have the two props `date` and `title` **must**, furthermore the two props `description` and `ticketsUrl` **may** have. Whether a prop is **required** can be indicated by the appended `.isRequired`. The `date` prop in our example must be an instance of the native JavaScript `Date` object, `title` must be a `string`. The two optional props `description` and `ticketsUrl` do not have to be passed; however, if they are passed, they must also be of type `string`.

If any of these conditions do not apply, React will alert us quite clearly with a warning in the console:

{% hint style="danger" %}
Warning: Failed prop type: Invalid prop \`title\` of type \`number\` supplied to \`EventOverview\`, expected \`string\`.
{% endhint %}

With **Stateless Functional Components** the **PropTypes** are defined in the same way, but of course we don't have a class in which we can define a `static propTypes` property. Here we can simply add a `propTypes` property to the function itself. It'll look that way:

```jsx
const EventOverview = ({ date, description, ticketUrl, title }) => (
  <div>
    <h1>{title}</h1>
    <h2>{date.toLocaleString()}</h2>
    {description &&& (
      <div className="description">{description}</div>
    )}
    {ticketsUrl && <a href={ticketsUrl}>Tickets!</a>}
  </div>
);

EventOverview.propTypes = {
  date: PropTypes.instanceOf(Date).isRequired,
  description: PropTypes.string,
  ticketsUrl: PropTypes.string,
  title: PropTypes.string.isRequired
}
```

And this would also be our **Stateless Functional Component** with **PropType**-Checking equipped!

In some cases, it is desirable to assign meaningful default values. React also offers us a possibility for this, the so-called `defaultProps`. These are used similarly to the `propTypes`, namely as a static property. But let's look at a quick example:

```jsx
const Greeting = ({ name }) => (
    <h1>Hello {name}!</h1>
);

Greeting.propTypes = {
    name: PropTypes.string.isRequired,
};

Greeting.defaultProps = {
    name: 'Guest',
};
```

We mark the `name` prop of the component as `string.isRequired`, so we expect that the prop will always be passed and that it will always be a string. Then we define a default value for the `name` prop. This is always used if no value is passed for the corresponding prop.

```jsx
<Greeting name="Manuel" />
```

So this causes the output: **Hello Manuel!**

```jsx
<Greeting />
// Or:
const user = {};
<Greeting name={user.name} />
```

This uses the `defaultValue`, in this case **Guest**, because of the missing or undefined `name` prop and displays it in both cases: \*\*React is smart enough to recognize if a `defaultValue` exists if a prop is missing but marked as `isRequired` and displays a warning only if a prop is missing and a `defaultValue` has not been defined at the same time.

{% hint style="info" %}
When deploying **Deployment in Production**, it is worth using the **Babel Plugin Transform React Remove Prop Types**. This saves a few more bytes of bandwidth as the `propType` definitions are removed from the build as they are **only considered in development mode** anyway.

You can find the plugin here:  
[https://github.com/oliviertassinari/babel-plugin-transform-react-remove-prop-types](https://github.com/oliviertassinari/babel-plugin-transform-react-remove-prop-types)

You can install it on the command line using :

```bash
npm install --save-dev babel-plugin-transform-react-remove-prop-types
```

or with yarn:

```bash
yarn add --dev babel-plugin-transform-react-remove-prop-types
```
{% endhint %}

## Flow

Unlike the **React PropTypes**, **Flow** is a **static Typechecker** for **all** JavaScript, not just for React components. Like React itself, **Flow** is developed by Facebook and fits seamlessly into most React setups. Until **Babel 6** it was even part of the `babel-preset-react` package, so it was "co-installed" for use with React and could be easily used without any additional effort.

Since **Babel 7** **Flow** has been outsourced to its own **Babel preset**, which can be installed just as easily via `npm install @babel/preset-flow` \(or similarly `yarn add @babel/preset-flow`\). Afterwards only the corresponding `@babel/preset-flow` has to be entered as preset in the Babel-Config. The **Preset** is required to remove the **Flow-Syntax**, which would not be a valid JavaScript, from the corresponding files in the **Build** process, so that there are no syntax errors when calling it in the browser.

Besides the Babel preset you also need the **Flow Executable**, which can be installed in its latest version by `npm install flow-bin` \(resp. `yarn add flow-bin`\). The **Flow Executable** then performs the actual **Typechecking**.

After **Flow** has been installed and the **Babel preset** has been set up, a **Flow-Config** is required. You can easily create them by calling `./node_modules/flow init` in the terminal in your project directory.

**Tip:** To avoid prefixing `./node_modules` each time Flow is called, you can make an entry in the `script` part of your `package.json`:

```javascript
{
  "scripts": {
    [...]
    "Flow": "Flow"
  }
}
```

This ensures that you can call Flow via npm or Yarn afterwards:

```bash
npm run flow init
```

or Yarn:

```bash
yarn flow init
```

After calling `flow init`, you should see a new file `.flowconfig` in your project directory, which looks pretty empty at first, but is needed by Flow. In this file you can later set options or specify which files should be checked with Flow or which not.

You have updated your Babel-Config, installed the `flow-bin` package into your project and created the `.flowconfig`? Great. Then we're ready to go. To verify that everything is set up correctly, you can call flow once. If you have added the flow entry from above to your `package.json`, you can do so with the `yarn flow` command in your terminal. If everything is set up correctly, you will see a message like the following:

```bash
No errors!
Done in 0.57s.
```

This means Flow has checked your files and found no errors. How also, we still haven't created any files with typechecking.

The default settings of Flow are that only files are checked which signalize to Flow that they contain type checks with a corresponding comment in the code. Simply add the following line to the top of any JavaScript file:

```javascript
// @flow
```

Let's look at the above example again. This time with **Flow** as Typechecker instead of **PropTypes**:

```jsx
// @flow
import * as React from 'react';
import * as ReactDOM from 'react-dom';

type PropsT = {
  date: Date,
  description?: string,
  ticketsUrl?: string,
  title: string,
};

class EventOverview extends React.Component<PropsT> {
  render() {
    const { date, description, ticketUrl, title } = this.props;
    return (
      <div>
        <h1>{title}</h1>
        <h2>{date.toLocaleString()}</h2>
        {description &&& (
          <div className="description">{description}</div>
        )}
        {ticketsUrl && <a href={ticketsUrl}>Tickets!</a>}
      </div>
    );
  }
}

ReactDOM.render(
  <EventOverview date={new Date()} title="Learn and understand React" />,
  document.getElementById('root')
);
```

Unlike the **PropTypes** we first define a **Type Definition** with the name `PropsT`. The name can be freely chosen here. Often, `T` or `Type` are appended to the name of the type definition to make it obvious to developers that it is just that type. But from a purely technical point of view this is not necessary. We then pass the just defined type to the component in the form of a so-called "Generic Type":

```jsx
class EventOverview extends React.Component<PropsT>
```

Type definitions can also be defined inline. However, from a certain number this also affects the readability. In our example, they'd look like this:

```jsx
class EventOverview extends React.Component<{
  date: Date,
  description?: string,
  ticketsUrl?: string,
  title: string,
}> {
  […]
}
```

But let's have a closer look at the **Type-Definition**. As with the **PropTypes** we determine here which **Props** a component can get passed and of which types they must be. There is a `date` prop, which must consist of a `date` instance and is required. Next are `description` and `ticketsUrl`, which are marked with a `?` after their name as **optional** and, if passed, must be of type `string`. Finally, a `title` prop is expected, which must also be a `string`, but is not optional. In contrast to **PropTypes**, here the required props do not have to be marked with `isRequired`, but on the contrary the optional props have to be marked as optional with question marks.

**Function components** can be typed in the same form when the props are passed as function arguments:

```jsx
const EventOverview = (props: PropsT) => (/*...*/);
```

or in destructured form:

```jsx
const EventOverview = ({ date, description, ticketUrl, title }: PropsT) => (/*...*/);
```

or as an inline definition:

```jsx
const EventOverview = ({
    date,
    description,
    ticketUrl,
    title
}: {
    date: Date,
    description?: string,
    ticketsUrl?: string,
    title: string
}) => {
  /*…*/
};
```

But that's not all. Flow, unlike **PropTypes**, can check all JavaScript, not just props of React components. This means that the state of a component can also be typed. A second parameter in the so-called **Generics** is provided for this purpose:

```jsx
// @flow
import * as React from 'react';
import * as ReactDOM from 'react-dom';

type PropsT = {
  date: Date,
  description?: string,
  ticketsUrl?: string,
  title: string,
};

type StateT = {
  isBookmarked: boolean,
};

class EventOverview extends React.Component<PropsT, StateT> {
  state = {
    isBookmarked: false,
  };
  […]
}
```

In contrast to previous examples in the book, the imports here have a slightly different form. Instead of

```jsx
import React from 'react';
```

... React was imported in this chapter as follows:

```jsx
import * as React from 'react';
```

This means that the **Type Definitions** provided by React were also imported at the same time. This is necessary if, for example, we want to return a react element from a function and type it.

## TypeScript

**TypeScript** is developed by Microsoft and is a so-called typed **Superset** of JavaScript, which means that it cannot be executed directly in the browser, but is first compiled into "real" JavaScript in an intermediate step by a compiler, valid JavaScript but is always also valid **TypeScript**. **TypeScript** looks at first sight similar to **Flow** and works similar. While **Flow** is only a pure **Typechecker**, **TypeScript** as Superset brings a bit more. Long before **ES2015** it was possible to use classes and imports in **TypeScript**.

In the JavaScript community **TypeScript** is becoming more and more popular and also in connection with React it is found more and more often. For this reason I don't want to leave it unmentioned here, but at the same time I don't want to go into too much depth because **TypeScript** alone would provide enough material for your own book.

With regard to React, it is important to know that TypeScript files usually have a `.ts' file extension, if a file also contains JSX, the file must end with`.tsx\`.

With the release of Babel 7 the integration was also simplified and it no longer needs the **TypeScript** compiler \(`tsc`\) but can be used as a Babel plugin. The plugin is installed with the Babel preset `@babel/preset-typescript`.

