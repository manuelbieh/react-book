# Typechecking with PropTypes, Flow and TypeScript

**Typechecking** is a simple method to avoid potential errors in an application. We have talked about basic principles to avoid potential errors, one of the most important being the existence of "Pure" components meaning that components do not have side effects. The non-existence of side effects describe the situation in which the **same input parameters** \(in the case of React we are talking about **props** and **state**\) should always render the **same output**.

We should be able to foresee and be clear about which props are passed into a component and how they are being processed. In order to achieve this, we can employ **typechecking** to make this process easier. JavaScript is slightly unusual in the sense that a variable which used to be **String** can easily be converted into a **Number** or even an **Object** without the JavaScript interpreter even complaining.

Even if this sounds useful for development, it opens doors for errors and also forces us to manually check for the correct types. When we want to access nested properties such as `user.settings.notifications.newMessages` we have to check user is actually an Object and not null, subsequently perform the same check for settings and so on. Otherwise, we might run into type errors:

{% hint style="danger" %}
TypeError: Cannot read property 'settings' of undefined
{% endhint %}

**Typechecking** can help us to catch such errors before they actually happen. Apart from **Flow** and **TypeScript** which allow for their own static typing, React provides its own simple solution called **PropTypes**. While **Flow** and **TypeScript** can also be used in JavaScript applications, **PropTypes** are exclusively used in React components. If you are interested into static types, you might like exploring **Flow** or **TypeScript** in greater detail.

## PropTypes

**PropTypes** have a long standing history in React and were part of React before it even gained popularity. In React version 15.5 **PropTypes** were removed from the core of React and converted into their own `prop-types` package. While you defined **PropTypes** via `React.PropTypes.string` in the past, you now access them via importing the `PropTypes` module and `PropTypes.string`.

This change also means that we now have to install the `PropTypes` module as a `devDependency`. We can execute the following step in the command line:

```bash
npm install --save-dev prop-types
```

```bash
yarn add --dev prop-types
```

**PropTypes** can be seen as some kind of interface and define which types the props can take and whether they are required or optional. If **PropTypes** encounter discrepancies, **React** will inform us of these as long as we find ourselves in **Development mode**. When applications have been built correctly and use either the production build of React or an environment variable `process.env.NODE_ENV=production`, this warning will not be shown anymore.

But how do **PropTypes** work? To answer this question, one needs to differentiate between **Class components** and **Function components**.

**Class components** implement **PropTypes** as a static property of the component in question:

```jsx
import React from 'react';
import ReactDOM from 'react-dom';
import PropTypes from 'prop-types';
​
class EventOverview extends React.Component {
  static propTypes = {
    date: PropTypes.instanceOf(Date).isRequired,
    description: PropTypes.string,
    ticketsUrl: PropTypes.string,
    title: PropTypes.string.isRequired
  };
​
  render() {
    const { date, description, ticketUrl, title } = this.props;
    return (
      <div>
        <h1>{title}</h1>
        <h2>{date.toLocaleString()}</h2>
        {description && (
          <div className="description">{description}</div>
        )}
        {ticketsUrl && <a href={ticketsUrl}>Tickets!</a>}
      </div>
    );
  }
}
​
ReactDOM.render(
  <EventOverview date={new Date()} title="React Deep-Dive" />,
  document.getElementById('root')
);
```

In this example, we want to display an event overview. Thus, we have defined that the `EventOverview` component **has to** accept both, a `date` and a `title` prop. It **can** furthermore have a `description` prop as well as a `ticketUrl` prop. One can append `.isRequired` to the PropTypes to indicate that the prop is **required**. We can read from our list of **PropTypes** that the date prop has to be an instance of the native `Date` object and that `title` has to be of type `string`. If the optional props `description` and `ticketsUrl` have been passed, they also need to be of type `string`.

If any of these conditions have not been met, React will gracefully inform us with a warning in the console:

{% hint style="danger" %}
Warning: Failed prop type: Invalid prop \`title\` of type \`number\` supplied to \`EventOverview\`, expected \`string\`.
{% endhint %}

**Function Components** do not have classes, thus we cannot define a `static propTypes` property. However, we can easily add a `propTypes` property to the function, which will result in the following:

```jsx
const EventOverview = ({ date, description, ticketUrl, title }) => (
  <div>
    <h1>{title}</h1>
    <h2>{date.toLocaleString()}</h2>
    {description && <div className="description">{description}</div>}
    {ticketsUrl && <a href={ticketsUrl}>Tickets!</a>}
  </div>
);

EventOverview.propTypes = {
  date: PropTypes.instanceOf(Date).isRequired,
  description: PropTypes.string,
  ticketsUrl: PropTypes.string,
  title: PropTypes.string.isRequired,
};
```

And just like that, we have added `propTypes` to our **Function Component**.

In some cases, it can be useful to define sensible default values if no specific values have been passed. React allows us to define `defaultProps` which work similar to `propTypes` in that they can also be added as a static property. Here's a quick example:

```jsx
const Greeting = ({ name }) => <h1>Hello {name}!</h1>;

Greeting.propTypes = {
  name: PropTypes.string.isRequired,
};

Greeting.defaultProps = {
  name: 'Guest',
};
```

The `name` prop of the `Greeting` component is marked as required by `string.isRequired` which means that we usually expect that a value is passed in the form of a string. If however no value is passed for the prop, the default value will be used.

```jsx
<Greeting name="Manuel" />
```

will result in the following output: **Hello Manuel!**

```jsx
<Greeting />;
// or:
const user = {};
<Greeting name={user.name} />;
```

This example however will fall back to the `defaultValue` which we have defined in the `defaultProps` as either no prop is passed at all or if the value is undefined. In this case, **Hello Guest!** will be displayed as we set **Guest** as our `defaultValue`. React will figure out if a `defaultValue` for a `isRequired` prop exists and only display a warning if the prop is actually missing and no `defaultValue` has been defined.

{% hint style="info" %}
When **Deploying to production** it is worth installing **Babel-Plugin-Transform-React-Remove-Prop-Types**. This will save us a couple of bytes, as `propType` definitions are removed from the build and are only really taken into account in **Development Mode**.

You can find the plugin here:  
[https://github.com/oliviertassinari/babel-plugin-transform-react-remove-prop-types](https://github.com/oliviertassinari/babel-plugin-transform-react-remove-prop-types)

You can install it via the command line with:

```bash
npm install --save-dev babel-plugin-transform-react-remove-prop-types
```

Or with yarn:

```bash
yarn add --dev babel-plugin-transform-react-remove-prop-types
```

{% endhint %}

## Flow

In contrast to **React PropTypes**, **Flow** is a **static typechecker** for **JavaScript** not just for React components. **Flow** is also developed in-house at Facebook and thus integrates nicely with most React setups. Up to version **Babel 6**, it even came pre-installed as part of the `babel-preset-react` and could be used without any further setup.

Since **Babel version7**, **Flow** has been ported to its own **Babel preset**. In order to install it you can run `npm install @babel/preset-flow` or `yarn add @babel/preset-flow`. In addition to the installation step, you also have to manually set the`@babel/preset-flow` as a present in the Babel config. **Presets** allow us to remove non-JavaScript syntax — in this case **Flow syntax** — during the build process so that we will not run into errors in the browser.

Apart from the Babel Preset, the **Flow executable** needs to be installed via `npm install flow-bin` \(or `yarn add flow-bin`\) which takes care of the actual **typechecking**.

Once **Flow** has been installed and the Babel Preset has been set up, you have to create a **Flow config** by executing `./node_modules/flow init` in the terminal in your current project directory.

**Hint:** To avoid prepending `./node_modules` every time you are calling Flow, you can make an addition in the `script` section of the `package.json`:

```javascript
{
  "scripts": {
    [...]
    "flow": "flow"
  }
}
```

This allows us to call Flow via npm or yarn:

```bash
npm run flow init
```

```bash
yarn flow init
```

Once `flow init` has been executed, a new file called `.flowconfig` should have been created in the project directory. The file itself looks very empty for now but Flow needs it to function correctly. In the future, you can manage which files should be checked by Flow or which should not based on the options set in these files.

Did you update your Babel config, install `flow-bin` in your object and created the `.flowconfig`? Awesome. We can now start typechecking with Flow. In order to check that everything has been set up correctly, you can execute `flow.` If you added the entry to the `package.json` as shown above, running `yarn flow` will suffice. When everything has been set up correctly, a message such as this one is displayed to you:

```bash
No errors!
Done in 0.57s.
```

This means that Flow has checked all of our files and has not found any errors. But this is kind of self-explanatory as we have not created any files containing any typechecks - yet.

The standard settings of Flow mandate that only those files will be typechecked that have included a specific code comment at the top of the file:

```javascript
// @flow
```

Let us look at our previous example, but this time using **Flow** instead of **PropTypes**:

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
        {description && <div className="description">{description}</div>}
        {ticketsUrl && <a href={ticketsUrl}>Tickets!</a>}
      </div>
    );
  }
}

ReactDOM.render(
  <EventOverview date={new Date()} title="React Deep-Dive" />,
  document.getElementById('root')
);
```

As opposed to **PropTypes**, we start off by defining a **Type definition** with the name of `PropsT`. You can choose freely which names to give to your Type definitions, but it has become some sort of Developer convention to use the suffix of `T` or `Type`. However, it is not enforced or technically necessary. This newly defined type can then be passed to the component using a so-called "generic type".

```jsx
class EventOverview extends React.Component<PropsT>
```

Type definitions can be inlined as well, however the longer the list of the definitions, the harder it becomes to read them in this form.

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

But let's inspect the **Type definition** a little further: just as we already learned in the section on **PropTypes**, we define which **props** can be passed to the component and which **type** they have to be. In this case, we have defined a `date` prop which needs to be an instance of `Date`, the optional props `description` and `ticketsUrl` of type `string` which have been marked as optional due to their use of the `?` after their name, and finally the `title` prop which also needs to be passed in the form of a `string`. In contrast to **PropTypes** where one needs to explicitly indicate which props are required by using `isRequired`, Flow syntax describes which fields are optional instead.

**Function components** can also be typed using Flow by passing the props and their Type as a function argument:

```jsx
const EventOverview = (props: PropsT) => (/*…*/);
```

Or in even shorter, destructured form:

```jsx
const EventOverview = ({ date, description, ticketUrl, title }: PropsT) => (/*…*/);
```

You can also use inline definitions in Function components:

```jsx
const EventOverview = ({
  date,
  description,
  ticketUrl,
  title,
}: {
  date: Date,
  description?: string,
  ticketsUrl?: string,
  title: string,
}) => {
  /*…*/
};
```

But that's not all! **Flow** can check any JavaScript and not only those bits that describe the props of React components. You can also type the State of your component by passing a second parameter in the so-called **Generics**.

```jsx
// @flow
import * as React from 'react';
import * as ReactDOM from 'react-dom';
​
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

In contrast to previous examples in the book, the imports follow a slightly different structure. Instead of:

```jsx
import React from 'react';
```

React has been imported like this:

```jsx
import * as React from 'react';
```

This allows us to also import React's **type definitions** which is necessary if we want to return a React element from a function and type it, for example.

## TypeScript

**TypeScript** has been created by Microsoft and is a so-called **typed superset** of JavaScript meaning that it cannot be directly executed in the browser but has to be compiled into "real" JavaScript first. Nevertheless, valid JavaScript is always valid **TypeScript**. **TypeScript** might look very similar to **Flow** at first glance, but differs in the sense that it offers a lot more functionality as it is a superset of JS. **Flow** is only a typechecker. Before **ES2015** was formally introduced, a lot of functionality such as classes and imports were already readily available in **TypeScript**.

**TypeScript** has been gaining momentum and lots of popularity lately and can now be found in many React projects. While I certainly find **TypeScript** worth mentioning in this chapter, **TypeScript** alone could easily fill up a whole book by itself which is why I would rather not go into more detail at this point.

**TypeScript** files are easy to spot as they usually have the file ending of `.ts` or, if **TypeScript** is used in a React project, `.tsx`.

With the release of **Babel 7**, the integration of **TypeScript** in projects has become much easier as one does not have to install the **TypeScript** compiler \(`tsc`\) anymore but can simply use a Babel plugin. This plugin is installed once you install the Babel Preset `@babel/preset-typescript`. If you are looking for more information on how to use **TypeScript** in your React projects, the official **TypeScript** documentation is a great place to start: [https://www.typescriptlang.org/docs](https://www.typescriptlang.org/docs).
