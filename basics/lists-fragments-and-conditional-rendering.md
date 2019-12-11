# Lists, Fragments, and Conditional Rendering

You have learned a lot about React so far. You now know why we need **props**, what **state** is and how it differs from **props**, you've learned how to implement a React component as well as to differentiate between a React **component** and a React **element**. You also learned how to use **JSX** to accurately depict the tree of elements that's being rendered and how you can leverage **lifecycle methods** to react to changes in our data. All of these lay the groundwork for simple React applications.

But there a few details that we have not examined yet and which have not been mentioned so far \(or not explained in detail\). However, these details will become more relevant the more complex our applications get.

**Lists** \(as in working with data arrays\), so-called **Refs** \(which are references to the DOM representation of a React element\), **Fragments** \(a special component that does not leave traces or elements in the output\) and **Conditional Rendering** based on props and state are part of these details and will thus be examined in this section.

They are too important to not mention them in the Basics section of the book but each topic on its own does not quite warrant its own chapter. Let's look at them now.

## Lists

Lists refer to plain JavaScript arrays in this instance: **simple** data that can be iterated over. They are essential for React, even plain JavaScript development and it's hard to imagine developing without them. **ES2015+** offers many nice declarative methods such as `Array.map()`, `Array.filter()` or `Array.find()` that can even be used as expressions in JSX \(if surrounded by curly braces `{}`\).

I've mentioned expressions in JSX and how to use them already in the chapter on JSX. But let's recap briefly: Arrays can be used as an expression in JavaScript and therefore also be included in JSX. If surrounded by curly braces, they will be treated as child nodes  during the transpilation step.

But there's more: `Array.map()` enables us to not only work with data but it can return modified item which in turn can also contain JSX. This adds further flexibility and allows us to transform data sets into React elements.

Let's assume that we want to show  list of cryptocurrencies. The array containing the data has the following form:

```jsx
const cryptos = [
    {
        id: 1,
        name: 'Bitcoin',
        symbol: 'BTC',
        quotes: { EUR: { price: 7179.92084586 } }
    },
    {
        id: 2,
        name: 'Ethereum',
        symbol: 'ETH',
        quotes: { EUR: { price: 595.218568203 } }
    },
    {
        id: 3,
        name: 'Litecoin',
        symbol: 'LTC',
        quotes: { EUR: { price: 117.690716234 } }
    }
];
```

First, we will aim to show this data as a simple unordered list in HTML. A resulting component could look similar to this:

```jsx
const CryptoList = ({ currencies }) => (
  <ul>
    {currencies.map((currency) => (
      <li>
        <h1>{currency.name} ({currency.symbol})</h1>
        <p>{currency.quotes.EUR.price.toFixed(2)} €</p>
      </li>
    ))}
  </ul>
);
```

The component could then be used like this:

```jsx
<CryptoList currencies={cryptos} />
```

The result is an unordered list with list items containing the crypto currency and their price. However, we also encounter an error in the console:

![](../.gitbook/assets/react-missing-key.png)

React expects a `key` prop for all of the values returned by arrays or iterators. The primary reasoning behind this rule is to make it easier for React's **Reconciler** \(the React Comparison algorithm\) to identify and compare list elements. The **Reconciler** can spot whether an array element was added, removed or modified if the `key` prop has been given. This prop needs to be a unique identifier which only appears **once in the array**. Normally, the id of a data set is used in practice.

In our example, we can take the id of each item. Amended, the example now looks like this:

```jsx
const CryptoList = ({ currencies }) => (
    <ul>
        {currencies.map((currency) => (
            <li key={currency.id}>
                <h1>{currency.name} ({currency.symbol})</h1>
                <p>{currency.quotes.EUR.price.toFixed(2)} €</p>
            </li>
        ))}
    </ul>
);
```

The key only has to be unique **within the iterator compared to its sibling elements but not inside the component**. We can easily use the `CryptoList` component elsewhere using the same key - even within the same component. Just don't use the key again in the same loop.

If your data set does not contain an easily distinguishable id, the **index** of the array can be used as a **last resort.** This is not recommended though and can lead to unexpected behavior in the rendering of user interfaces and also impact performance.

It's important that the `key` prop is always **present in the top-level component or array element within the iterator** but not in the JSX returned by the component. 

To illustrate this point further, I'm going to transform the above list item into its own `CryptoListItem` component.

```jsx
const CryptoListItem = ({ name, symbol, quotes }) => (
  <li>
    <h1>{name} ({symbol})</h1>
    <p>{quotes.EUR.price.toFixed(2)} €</p>
  </li>
);
```

What do we notice? The `key` prop which we added beforehand is no longer present. Our `map()` call would now look like this:

```jsx
const CryptoList = ({ currencies }) => (
  <ul>
    {currencies.map((currency) => (
      <CryptoListItem
        key={currency.id}
        name={currency.name}
        symbol={currency.symbol}
        quotes={currency.quotes}
      />
    ))}
  </ul>
);
```

Although we actually render a `<li></li>` element under the hood, the `<CryptoListItem />` needs to contain the `key` prop as it is the top level component within our `Array.map()` call in this JSX snippet.

A little bit off-topic: We could even simplify the `CryptoList` component by using the **Object spread syntax**:

```jsx
const CryptoList = ({ currencies }) => (
  <ul>
    {currencies.map((currency) => <CryptoListItem key={currency.id} {...currency} />)}
  </ul>
);
```

This way, all properties of the `currency` objects are passed as props to the `CryptoListItem` component.

If we didn't use an iterator such as `Array.map()`, a list would need to be explicitly defined like this:

```jsx
const MyList = () => (
  <ul>
  {[
    <li key="1">One</li>,
    <li key="2">Two</li>,
    <li key="3">Three</li>
  ]}
  </ul>
);
```

## Fragments

Fragments are some sort of a special component and allow us to create valid JSX without leaving visible traces in the rendered markup. They are a solution to the "problem" of only ever returning a single element at the top in JSX. This is valid JSX:

```jsx
render() {
  return (
    <ul>
      <li>Bullet Point 1</li>
      <li>Bullet Point 2</li>
      <li>Bullet Point 3</li>
    </ul>
  );
}
```

But this isn't:

```jsx
render() {
  return (
    <li>Bullet Point 1</li>
    <li>Bullet Point 2</li>
    <li>Bullet Point 3</li>
  );
}
```

In this example, multiple elements are being returned in the `render()` method without a surrounding element leading to an error. We do not always want to create new elements though, especially if the surrounding element is found in a parent component and the child element is found in its own component. 

On top of that, many elements \(such as `table`, `ul`, `ol`, `dl`, ...\) do not allow for `div` elements to be used as an intermediary wrapper \(ignoring the fact that we would also litter the markup by using `divs`  everywhere for now\). As we are only permitted to ever return a single root element from a component, **Fragments** can be incredibly useful. We could transform our example from above into the following:

```jsx
render() {
  return (
    <React.Fragment>
      <li>Bullet Point 1</li>
      <li>Bullet Point 2</li>
      <li>Bullet Point 3</li>
    </React.Fragment>
  );
}
```

The rule that every element returned by a loop needs to have a `key` prop, still holds. Using a **Fragment** this is still possible. Let's illustrate this further by examining another slightly more complex yet more realistic example:

```jsx
import React from 'react';
import ReactDOM from 'react-dom';

const TicketMeta = ({ metaData }) => (
  <dl>
    {Object.entries(metaData).map(([property, value]) => (
      <React.Fragment key={property}>
        <dt>{property}</dt>
        <dd>{value}</dd>
      </React.Fragment>
    ))}
  </dl>
);

ReactDOM.render(
  <TicketMeta
    metaData={{
      createdAt: '2018-06-09',
      author: 'Manuel Bieh',
      category: 'General',
    }}
  />,
  document.getElementById('root')
);
```

The resulting output would be the following:

```markup
<dl>
  <dt>createdAt</dt>
  <dd>2018-06-09</dd>
  <dt>author</dt>
  <dd>Manuel Bieh</dd>
  <dt>category</dt>
  <dd>General</dd>
</dl>
```

It would simply not be possible to wrap a `div` or `span` or another element around the `<dt></dt>` or `<dd></dd>`. Trying to do that would result in this:

```markup
<dl>
  <div>
    <dt>createdAt</dt>
    <dd>2018-06-09</dd>
  </div>
  <div>
    <dt>author</dt>
    <dd>Manuel Bieh</dd>
  </div>
  <div>
    <dt>category</dt>
    <dd>General</dd>
  </div>
</dl>
```

... and is invalid HTML! A `dl` element only permits `dt` and `dd` as its child element. The **Fragment** helps us to alleviate these situations and create valid JSX without creating invalid markup. It was only introduced in React 16.3 and meant that some React components were unnecessarily complex to deal with this problem and avoid violating JSX or HTML rules. 

**Fragment** components can also be imported directly from React using a named import:

```javascript
import React, { Fragment } from 'react';
```

Now, the notation of `<Fragment>` instead of `<React.Fragment>` can be used saving us multiple key strokes.

Using Babel 7 for transpilitation, we can shorten the notation even further. An _empty_ element can created using:

```jsx
<>Fragment in Kurzform-Syntax</>
```

This is a neat and tidy method to reduce the amount of fragments we need to explicitly define by name. But be careful: using the shorthand is not possible in loops as empty elements cannot contain any props. Elements in a loop require us to define a `key` prop thus forcing us to use `<React.Fragment>` instead in this case.

## Conditional rendering

Rendering components based on different conditions, **conditional rendering** for short, is a central concept in React. As React components are composed of JavaScript functions, objects and classes under the hood, conditions function and behave just as in regular JavaScript.

A React component renders a **state** of a **user interface** based on its **props** and its current **state**. In an ideal situation, free from **side effects**. To correctly deal with these different and changing parameters, using the render function to react to different conditions is a powerful feature. If my parameter is A, render this; if my parameter is B, render that. If I have incoming data in a list, show this data as an HTML list. If I do not have any data, show a placeholder instead.

If this sounds relatively simple, I can assure you, it is. But one should still be aware of knowing how to do conditional rendering in JSX. The `render()` function of **class components** as well as **stateless functional** **components** can return a **React element** \(also in form of JSX\), a **string**, a **number,** `null` \(if nothing should be rendered\) or an **array** of these types.

There are a few methods of keeping the `render()` method nice and clean which I will explain now.

### if/else

Probably the most simple and commonly known way to **conditionally render** is using `if`/`else`:

```jsx
const NotificationList = ({ items }) => {
  if (items.length) {
    return (
      <ul>
        {items.map((notification) => (
          <li>{notification.title}</li>
        ))}
      </ul>
    );
  }
  return <p>No new notifications</p>
};
```

Relatively simple use case. The `NotificationList` component receives a list of items in the form of props. If the list contains any entries at all, they are rendered as a list item of an unordered list. If however the list is empty, a message is returned informing the user that no new notifications are available.

Another more complex example. Let's imagine we are working with a value that we want to make editable. Our component differentiates between the different modes of `edit` and `view`. Depending on which mode we are currently in, we either want to simply show text \(**View mode**\) or be able to see a text field containing the previously entered value \(**edit mode**\).

```jsx
import React from 'react';
import { render } from 'react-dom';

class EditableText extends React.Component {
  state = {
    value: null,
  };

  static getDerivedStateFromProps(nextProps, prevState) {
    if (prevState.value === null) {
      return {
        value: nextProps.initialValue || '',
      };
    }
    return null;
  }

  handleChange = (e) => {
    const { value } = e.target;
    this.setState(() => ({
      value,
    }));
  }

  setMode = (mode) => () => {
    this.setState(() => ({
      mode,
    }));
  }

  render() {
    if (this.state.mode === 'edit') {
      return (
        <div>
          <input 
            type="text" 
            value={this.state.value} 
            onChange={this.handleChange} />
          <br />
          <button onClick={this.setMode('view')}>Done</button>
        </div>
      );
    }

    return (
      <div>
        {this.state.value}
        <br />
        <button onClick={this.setMode('edit')}>Edit</button>
      </div>
    )
  }
}

render(
  <EditableText initialValue='Example' />, 
  document.getElementById('root')
);

```

The relevant part can be found in the `render()` method of the component. The state is tested against its property value in `mode`: if `edit` is currently the value in state, we directly return the input field with an "early return". If this is not the case, we assume that the "standard case" is taking place meaning that the current mode is `view`. The `else` part of this condition is not actually necessary here and would only add unnecessary complexity. Both times, the text is rendered with the difference that it is an editable `value` of an `input` field in one case, and a simple text node in the other. A button is included to switch between the different modes of `view` and `edit`.

 Such `if`, `if`/`else` or `if`/`else if`/`else`**state** and **props** within a component. I will explain them in more detail in just a moment.

### null

No, this is not a mistake. Returning `null` is the most simple case of **conditional rendering**. If the `render()` method of a component returns `null`, nothing is rendered and also does not appear in the DOM. This can be useful for displaying error components that should only be displayed if the error has actually occurred.

```jsx
render() {
  if (!this.state.error) {
    return null;
  }

  return (
    <div className="error-message">{this.state.error.message}</div>
  );
}
```

Using this form of a conditional check, we test whether the state contains an error property. If this isn't the case, `null` is being returned. Otherwise, an error message containing the error in state is returned in a `div`. We achieve this by using a simple `if` condition similar to what we have used above.

### Ternary Operator

Using the conditions that we have just seen, are often used to deal with relatively big changes in a component. In many situations, we only want to differentiate between minor differences, for example setting a CSS class if a certain state is set. The ternary operator helps us to do just that. Let's refresh our knowledge. A ternary operator takes the following form: `condition ? met : not met`. For example: `isLoggedIn ? ' Logout' : ' Login';`

That's our first example for a ternary operator in JSX! It can be used within props but also in conditions to differentiate between different forms of output to render based on conditions. To make the example more concrete, we could use the info from above to include it in a button:

```jsx
render() {
  const { isLoggedIn } = this.props;
  return (
    <button type="submit">{ isLoggedIn ? 'Logout' : 'Login' }</button>
  );
}
```

We will always return a button, but based on its `isLoggedIn` prop it can either include **Logout** or **Login**.

Equally, the **ternary operator** can be used in props. Let's assume that we want to render a list of users in which some users have been deactivated. In this case, we want to be able to set a class to mark it using CSS. Markup that deals with this problem could look like this:

```jsx
render() {
  const { user } = this.props;
  return (
    <div className={user.isDisabled ? 'is-disabled' : 'is-active'}>{user.name}</div>
  );
}
```

Users that have been deactivated are now marked with the `is-disabled` class, whereas active users are denoted with `is-active`.

Even complex JSX can be displayed using the **ternary operator**. As long as you follow the rule to use parentheses if the JSX spans multiple lines: 

```jsx
render() {
  const { country } = this.props;
  return (
    <div>
      <p>State:</p>
      {country === 'de' ? (
        <select name="state">
          <option value="bw">Baden-Württemberg</option>
          <option value="by">Bayern</option>
          <option value="be">Berlin</option>
          <option value="bb">Brandenburg</option>
          […]
        </select>
      ) : (
        <input type="text" name="state" />
      )}
    </div>
  );
}
```

A select list containing all German counties is rendered if we previously selected `de` \(for **Germany**\). In all other cases a simple text input is shown to user in which they can enter their county freely. However, careful consideration should be given when to use the **ternary operator**: it can become a little hard to read quickly if complex JSX is used.

### Logical AND \(`&&`\) und Logical OR \(`||`\)

The **Logical Operator** seems to resemble the **Ternary Operator** at first glance, but it is even shorter and more precise. As opposed to the **Ternary operator**, a second "else" case is not needed and can be skipped. If the condition of the **Logical AND Operator** is not met, the expression simply returns `undefined` resulting in no visible markup for the user interface:

```jsx
render() {
  const { isMenuVisible } = this.props;
  return (
    <header>
      { isMenuVisible && <Menu /> }
    </header>
  );
}
```

In this example, we test against the value of the `isMenuVisible` prop and check if it is `true`, if that **is** the case, it will return the `Menu` component. If the result is `false`, `undefined` is returned and nothing else will be rendered to the screen.

In combination with the **Logical OR Operator**, we can emulate the behavior of the **Ternary Operator**:

```jsx
render() {
  const { isLoggedIn } = this.props;
  return (
    <button type="submit">{ isLoggedIn && 'Logout' || 'Login' }</button>
  );
}
```

The button will be labelled **Logout** if the `isLoggedIn` **prop** is `true`, or **Login** if the user is logged out.

### Custom `render()` methods

Another way to increase the readability during complex **conditional rendering** is to move parts from the regular `render()` method to separate `renderXY()` methods. The regular `render()` method still forms the core of the component and decides which oarts of the user interface to show to the use. Thus, this method should not become overly complex or contain an unnecessary amount of logic.

It is not uncommon to move parts of long and complex `render()` methods into much smaller, more digestable chunks and implement these as custom class methods. If proper naming is used, this technique usually aids readability and understanding. Often these custom `render()` blocks are combined with `if` blocks:

```jsx
class Countdown extends React.Component {
  renderTimeLeft() {
    // […]
  }
  
  renderTimePassed() {
    // […]
  }
  
  render() {
    const { currentDate, eventDate } = this.props;
    if (currentDate < eventDate) {
      // currentDate is before eventDate so render countdown
      return this.renderTimeLeft();
    }
    // time is over so render how much time has passed since then
    return this.renderTimePassed();
  }
}
```

This **can** improve readability of the `render()` method but also increases the complexity of the component slightly. Many people recommend to move parts of the code into their own **function components** instead though \(myself included\).

{% hint style="info" %}
As soon as you begin to consider moving parts of your code into custom `render()` methods within your component, you should think about moving these into their own separate **function components**.
{% endhint %}

### Custom Components with complex conditions

Instead of using multiple `render()` methods, we can create new **function components**. These will receive **props** from their parent component and then take care of their own data display as an independent, self-managed and testable component.

Careful consideration should be given as to how and which data actually needs passed down to the new child component\(s\) and should not contain too much logic or state. 

Using custom components becomes useful once the `render()` method in one component has become too complex or if the same elements are used repetitvely within a component.

Let's look at a form which consists of many similar text fields. Each of these text fields is embraced by its own paragraph, contains a label and a `type` attribute. The label also needs to be equipped with an id that needs uniquely set for each field.

```jsx
render() {
  return (
    <form>
      <p>
        <label for="email">
          Email
        </label>
        <br />
        <input type="email" name="email" id="email" />
      </p>
      <p>
        <label for="password">
          Password
        </label>
        <br />
        <input type="password" name="password" id="password" />
      </p>
      <input type="submit" value="Send" />
    </form>
  );
}
```

We only have two form fields in this example but in most projects you will find many more fields in way more complex forms. Even in this case, it might make sense to extract recurring fields into their own components to save ourselves keystrokes.

First, create a `TextField` component and then cut the recurring JSX from the form component into this one:

```jsx
const TextField = ({ id, label, ...HTMLInputAttributes }) => (
  <p>
    <label for={id}>
      {label}
    </label>
    <br />
    <input {...HTMLInputAttributes} id={id} />
  </p>
);

export default TextField;
```

This new component receives an `id` which is needed to link the label to the text input. Using **Object Rest/Spread**, we add all remaining props to the `input` element as attributes, transforming the component into the following:

```jsx
render() {
  return (
    <form>
      <TextField name="email" label="Email" id="email" type="email" />
      <TextField name="password" label="Password" id="password" type="password" />
      <input type="submit" value="Send" />
    </form>
  );
}
```

We have just transformed long and potentially hard to read markup into a clean and precise `render()` method which only consists of very few components. If we ever wanted to add a change which should affect all of our text fields \(for example adding a class\), this can be added with ease by only changing data in one place - in the new `TextField` component.

