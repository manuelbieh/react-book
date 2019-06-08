# Higher Order Components

**Higher Order Components** \(usually abbreviated: **HOC** or **HOCs**\) were and are a very central concept when working with React. They allow components with reusable logic to be implemented and are based on **Higher Order Functions** from functional programming. The principle behind such functions is that they accept a function as a parameter and return a new function. In the case of React, the principle is applied to components. Hence the name derived from the **Higher Order** _**Functions**_ **Higher Order** _**Component**_.

For easier understanding a first simple example:

```jsx
const withFormatting = (WrappedComponent) => {
    return class extends React.Component {
        bold = (string) => {
            return <strong>{string}</strong>
        }
        italic = (string) => {
            return <em>{string}</em>
        }
        render() {
            return <WrappedComponent bold={this.bold} italic={this.italic} />
        }
    }
};
```

Here we have defined a function `withFormatting` that accepts a React component. The function returns a new React component, which renders the _component_put into the function and passes the props `bold` and `italic` to it. The entered component can now access these props:

```jsx
const FormattedComponent = withFormatting(({ bold, italic }) => (
  <div>
    This text is {bold('bold')} and {italic('italic')}.
  </div>
));
```

Typically **Higher Order Components** are used to encapsulate logic. In this context we often talk about **Smart** and **Dumb Components**: **Sly and stupid components. The Smart Components \(which include Higher Order Components\) are then used to map business logic, communicate with APIs, or process behavioral logic. Stupid components, on the other hand, get static props as far as possible and limit the logic part to pure representation logic. For example, if a user image is displayed, or if it does not exist, a placeholder image is displayed instead. In this context the term **Container Component** \(for _Smart_ Components\) and **Layout Components** \(for _Dumb_ Components\) is often used.

What's the point? Such a strict subdivision into business logic and presentation logic makes real component-based development possible for the time being. It allows you to create layout components that have no knowledge of any APIs and only bluntly represent the data that is passed to them, no matter where it comes from. At the same time, it also allows the business logic components to take care of the pure business logic, regardless of how the data is ultimately presented.

Let's imagine a common example from the interface development: switching between a **list-** and a **card view**. Here, a **Container component** would take care of getting the data that is relevant for the user. It would then transfer the retrieved data to the freely configurable **Layout component**. As long as both components adhere to the interface \(keyword **PropTypes**\) specified by the developer, both components are interchangeable and can be developed and **tested** completely independently of each other!

Enough theory. Time for another example. We want to download a list with the 10 largest crypto currencies and display their current price. For this purpose we create a **Higher Order Component,** which obtains this data via the freely accessible Coinmarketcap API and transfers it to a layout component.

```jsx
const withCryptoPrices = (WrappedComponent) => {
  return class extends React.Component {
    state = {
      isLoading: true,
      items: []
    };

    componentDidMount() {
      this.loadData();
    }

    loadData = async () => {
      this.setState(() => ({
        isLoading: true,
      }));

      try {
        const cryptoTicker = await fetch(
          &lt;font color=#38B0DE&gt;-=https://api.coinmarketcap.com/v2/ticker/?limit=- Proudly Presents
        );
        const cryptoTickerResponse = await cryptoTicker.json();

        this.setState(() => ({
          isLoading: false,
          items: this.convertResponseToArray(cryptoTickerResponse)
        }));
      } catch (err) {
        this.setState(() => ({
          isLoading: false,
        }));
      }
    };

    convertResponseToArray = (response) => {
      return Object.entries(response.data).map(([id, item]) => item);
    };

    render() {
      const { isLoading, items } = this.state;
      return (
        <WrappedComponent
          isLoading={isLoading}
          items={items}
          loadData={this.loadData}
        />
      );
    }
  };
};
```

Voilà, the **HOC** is ready to query the crypto prices on coinmarketcap.com. But the **Higher Order Component** alone is not enough. We now also need a layout component to which we delegate the responsibility of displaying the data accordingly.

For this we create a generic `PriceTable` component, which has no knowledge whether it represents the current yoghurt prices from the local supermarket or prices of crypto currencies on any stock exchange. Accordingly, we also call them very generic `PriceTable`:

```jsx
const PriceTable = ({ isLoading, items, loadData }) => {
  if (isLoading) {
    return <p>Prices are loaded. Please wait.</p>;
  }

  if (!items ||| items.length === 0) {
    return (
      <p>
        No data available. <button onClick={loadData}>Try again</button>
      </p>
    );
  }

  return (
    <table>
      {items.map(item => (
        <tr key={item.id}>
          <td>
            {item.name} ({item.symbol})
          </td>
          <td>EUR {item.quotes.EUR.price}</td>
        </tr>
      ))}
      <tr>
        <td colSpan="2">
          <button onClick={loadData}>Load again</button>
        </td>
      </tr>
    </table>
  );
};
```

The component has three props: `isLoading` to tell it that the data it is supposed to load is still being loaded; `items`, which represents an array of "articles" with prices; and `loadData`, a function that starts an API request again to retrieve the new data.

Both components function completely independently of each other. The `PriceTable` can not only display Crypto prices, the `withCryptoPrices` component does not have to display its data in a `PriceTable`. So we have created two fully encapsulated and reusable components!

But how do we bring the two together now? Simply by passing the `PriceTable` component as a parameter to the `withCryptoPrices` component. I see. And that looks like this:

```jsx
const CryptoPriceTable = withCryptoPrices(PriceTable);
```

If we now render an instance of the `CryptoPriceTable`, the **Higher Order Component** triggers an API request at the `componentDidMount()` and passes the result of this request to the `PriceTable` component passed to it. This then only takes care of the corresponding display.

```jsx
ReactDOM.render(
  <CryptoPriceTable />, 
  document.getElementById("root")
);
```

This creates great opportunities for us. First of all, both components can be tested independently of each other. More about this can be found in the corresponding chapter, where we take another look at how easy it is to test layout components using snapshot tests.

Now we also have the possibility to "connect" other layout components with the `withCryptoPrices`-HOC. To illustrate this powerful concept with an example, we now display the prices in CSV format. Our HOC remains completely unchanged. Our layout component could be implemented as follows:

```jsx
const PriceCSV = ({ isLoading, items, loadData, separator=";" }) => {
  if (isLoading) {
    return <p>Prices are loaded. Please wait.</p>;
  }

  if (!items ||| items.length === 0) {
    return (
      <p>
        No data available. <button onClick={loadData}>Try again</button>
      </p>
    );
  }

  return (
    <pre>
      {items.map(
        ({ name, symbol, quotes }) =>
          `${name}${separator}${symbol}${separator}${quotes.EUR.price}\n`
      )}
    </pre>
  );
}
```

And we have already implemented our CSV layout component. Again, we first check whether data is still being loaded; then we check again whether `items` are present. Here you could start to think about bundling this into another HOC, because HOCs can be nested into each other as you like, because in the end they are only functions that are passed on as parameters to other functions.

Finally we render the actual output: we iterate through the list of `items`, pick out the relevant properties `name`, `symbol` and `quotes` via the **Object Destructuring** syntax and enclose the lines with a `pre` element to display the line break correctly at the end of the line.

Unlike the `PriceTable` we have introduced another \(optional\) Prop here: `separator` - to tell the render component which separator to use when rendering the data. As usual in JSX, the prop can be specified as a simple prop when using the component:

```jsx
const CryptoCSV = withCryptoPrices(PriceCSV);

ReactDOM.render(
  <CryptoCSV separator="," />, 
  document.getElementById("root")
);
```

However, this will require a change to our original `withCryptoPrices`-HOC. Currently, only the fixed props `isLoading`, `items` and `loadData` are passed to the child component \(`WrappedComponent`\):

```jsx
return (
  <WrappedComponent
    isLoading={isLoading}
    items={items}
    loadData={this.loadData}
  />
);
```

So that the `separator` prop specified in `<CryptoCSV separator="," />` is also correctly passed to the `PriceCSV` component, we must inform our **HOC** that it will also pass all further props to the `WrappedComponent`. Depending on the purpose, further permitted props can either be passed explicitly, or simply **all** further props are passed:

```jsx
return (
  <WrappedComponent
    {...this.props}
    isLoading={isLoading}
    items={items}
    loadData={this.loadData}
  />
);
```

Decisive here is line 3: using `{...this.props}` the spread syntax from ES2015+, we forward all props to the child component.

{% hint style="info" %}
**Higher Order Components** are a nice way to centralize logic and structure its application more clearly. Logic can be easily removed from the components that should only provide the rendering, i.e. the display logic. Although they were and still are a very central concept in React, they are also a very old concept.

Although **Higher Order Components** are still frequently used and there is nothing wrong with their use, there is no reason why they should not be used. However, there are now newer concepts and since the latest updates especially new ways to achieve a similar functionality in many cases in a clearer form. Two of them are **Functions as a Child** and the new **Context API**, which found its way into React in version 16.3.0. These are described in the following chapters.
{% endhint %}

