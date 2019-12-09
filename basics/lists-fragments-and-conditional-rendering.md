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

First, we will aim to show this data as a simple unordered list in HTML. A resulting component could look similar to this&gt;

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

The component could be used like this:

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

