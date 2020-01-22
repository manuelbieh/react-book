# Typechecking with PropTypes, Flow and TypeScript

**Typechecking** is a simple method to avoid potential errors in an application. We have talked about basic principles to avoid potential errors, one of the most important being the existence of "pure" components meaning that components do not have side effects. The non-existence of side effects describe the situation in which the **same input parameters** \(in the case of React we are talking about **props** and **state**\) should always render the **same output**.

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

But how do **PropTypes** work? To answer this question, one needs to differentiate between **Class Components** and **Stateless Functional Components**.

**Class Components** implement **PropTypes** as a static property of the component in question:

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
  <EventOverview date={new Date()} title="Learning React" />,
  document.getElementById('root')
);
```

In this example, we want to display an event overview. Thus, we have defined that the `EventOverview` component **has to** accept both, a `date` and a `title` prop. It **can** furthermore have a `description` prop as well as a `ticketUrl` prop. One can append `.isRequired` to the PropTypes to indicate that the prop is **required**. We can read from our list of **PropTypes** that the date prop has to be an instance of the native `Date` object and that `title` has to be of type `string`. If the optional props `description` and `ticketsUrl` have been passed, they also need to be of type `string`.

If any of these conditions have not been met, React will gracefully inform us with a warning in the console:

{% hint style="danger" %}
Warning: Failed prop type: Invalid prop \`title\` of type \`number\` supplied to \`EventOverview\`, expected \`string\`.
{% endhint %}

**Stateless Functional Components** do not have classes, thus we cannot defined a `static propTypes` property. However, we can easily add a `propTypes` property to the function, which will result in the following:

```jsx
const EventOverview = ({ date, description, ticketUrl, title }) => (
  <div>
    <h1>{title}</h1>
    <h2>{date.toLocaleString()}</h2>
    {description && (
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

And just like that, we have added `propTypes` to our **Stateless Functional Component**.

In some cases, it can be useful to define sensible default values if no specific values have been passed. React allows us to define `defaultProps` which work similar to `propTypes` in that they can also be added as a static property. Here's a quick example:

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

The `name` prop of the `Greeting` component is marked as required by `string.isRequired` which means that we usually expect that a value is passed in the form of a string. If however no value is passed for the prop, the default value will be used.

```jsx
<Greeting name="Manuel" />
```

will result in the following output: **Hello Manuel!**

```jsx
<Greeting />
// or:
const user = {};
<Greeting name={user.name} />
```

This example however will fall back to the `defaultValue` which we have defined in the `defaultProps` as either no prop is passed at all or if the value is undefined. In this case, **Hello Guest!** will be displayed as we marked **Guest** as our `defaultValue`. React will figure out if a `defaultValue` for a `isRequired` prop exists and only display a warning if the prop is actually missing and no `defaultValue` has been defined.

{% hint style="info" %}
When **Deploying to production** it's worth installing **Babel-Plugin-Transform-React-Remove-Prop-Types**. This will save us a couple of bytes, as `propType`definitions are removed from the build and are only really taken into account in **Development Mode**.

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

In contrast to **React PropTypes**, **Flow** is a **static typechecker** for all **JavaScript** not just for React components. **Flow** is also developed in-house at Facebook and thus integrates nicely with most React setups. Up to version **Babel 6**, it even came pre-installed as part of the `babel-preset-react` and could be used without any further setup.

Since **Babel version7**, **Flow** has been ported to its own **Babel preset**. In order to install you can run `npm install @babel/preset-flow` or `yarn add @babel/preset-flow`. In addition to the installation step, you also have to manually  set the`@babel/preset-flow` as a present in the Babel config. **Presets** allow us to remove non-JavaScript syntax - in this case **Flow syntax** - during the build process so that we won't run into errors in the browser.

Apart from the Babel Preset, the **Flow executable** needs installed via `npm install flow-bin` \(or `yarn add flow-bin`\) which takes care of the actual **typechecking**.

Once **Flow** has been installed and the Babel Preset has been set up, you have to create a **Flow config** by executing `./node_modules/flow init` in the terminal in your current project directory.

