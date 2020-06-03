# Higher Order Components

**Higher Order Components** \(or **HOC** or **HOCs** for short\) were, and still are, a central concept in React. They allow you to implement components with reusable logic and are loosely tied to **Higher Order Functions** from functional programming. These kind of functions take a function as a parameter and return another function. In terms of React, this principle is applied to components. **Higher Order Components** derive their name from those **Higher Order Functions**.

Let us look at an example to illustrate this concept better:

```jsx
const withFormatting = (WrappedComponent) => {
  return class extends React.Component {
    bold = (string) => {
      return <strong>{string}</strong>;
    };
    italic = (string) => {
      return <em>{string}</em>;
    };
    render() {
      return <WrappedComponent bold={this.bold} italic={this.italic} />;
    }
  };
};
```

We have defined a function called `withFormatting` that takes in a React component. The function will return a new React component which in turn renders the component that was passed into the function and equips it with the props `bold` and `italic`. The component can now access these props:

```jsx
const FormattedComponent = withFormatting(({ bold, italic }) => (
  <div>
    This text is {bold('bold')} and {italic('italic')}.
  </div>
));
```

Typically, **Higher Order Components** can be used to encapsulate logic. They relate closely to the concept of **smart** and **dumb** components. Smart components \(which also encompass **Higher Order Components**\) are used to display business logic, deal with API communication or behavioral logic. _Dumb components_ in contrast mostly get passed static props and keep logic to a minimum (which is only used to for display-logic). For example, it might decide whether to show a profile image or, if it is not present, show a placeholder image instead. Sometimes, we also refer to **Container Components** \(for _Smart_ components\) and **Layout Components** \(for _Dumb_ components\).

But why do we categorize components this way? This strict divide into business logic and display logic enables component based development. It allows us to create layout components which do not know of possible API connections and only display data which is passed to them, no matter where it comes from. It also enables the business logic components to only concern themselves with business logic without caring about how the data is displayed in the end.

Assume we want to switch between a **list** and **map** view in a user interface. A **container component** will be in charge of gathering the data which is needed for the user and will pass them to the configurable **layout component**. As long as both components keep to the interface the developer has set up \(think **PropTypes**\), both components are easily interchangeable and can be tested and developed independently.

But enough of the theory. Let's look at an example. We are going to load a list of the 10 biggest cryptocurrencies and their current price. To obtain the data from the Coinmarketcap API, we create a **Higher Order Component** which loads the data and passes it to the layout component:

```jsx
const withCryptoPrices = (WrappedComponent) => {
  return class extends React.Component {
    state = {
      isLoading: true,
      items: [],
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
          'https://api.coinmarketcap.com/v2/ticker/?limit=10&convert=EUR'
        );
        const cryptoTickerResponse = await cryptoTicker.json();

        this.setState(() => ({
          isLoading: false,
          items: this.convertResponseToArray(cryptoTickerResponse),
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

Et voila! We have written an **HOC** to obtain the data of the crypto prices from coinmarketcap.com. But the **Higher Order Component** itself is not enough to make this work: we also need to define a layout component to which we delegate the task of displaying the data.

In order to do that, we define a generic `PriceTable` component which does not know about which data exactly it displays: it could be current yoghurt prices from the local supermarket or cryptocurrency prices from a stock market. Thus, we have given it a very generic name, `PriceTable`:

```jsx
const PriceTable = ({ isLoading, items, loadData }) => {
  if (isLoading) {
    return <p>Prices are being loaded. Please wait.</p>;
  }

  if (!items || items.length === 0) {
    return (
      <p>
        No data available. <button onClick={loadData}>Try again!</button>
      </p>
    );
  }

  return (
    <table>
      {items.map((item) => (
        <tr key={item.id}>
          <td>
            {item.name} ({item.symbol})
          </td>
          <td>EUR {item.quotes.EUR.price}</td>
        </tr>
      ))}
      <tr>
        <td colSpan="2">
          <button onClick={loadData}>Reload</button>
        </td>
      </tr>
    </table>
  );
};
```

This component knows about three props: `isLoading`, to inform it which data is still being loaded, `items`, which represents an array of articles with their corresponding prices and `loadData`, a function which allows us to start another API request to obtain new data.

Both components act independently of one another. The `PriceTable` is not limited to showing cryptocurrency prices, and the `withCryptoPrices` component does not necessarily need to display its data in a `PriceTable` component. We managed to write two completely encapsulated and reusable components.

But how do we combine these two components now? We can simply pass the `PriceTable` component as a parameter to the `withCryptoPrices` HOC component. This will look like this:

```jsx
const CryptoPriceTable = withCryptoPrices(PriceTable);
```

Whenever an instance of the `CryptoPriceTable` is rendered, the **Higher Order Component** will trigger an API request in the `componentDidMount()` lifecycle method and pass its result to the `PriceTable` component. The `PriceTable` then only needs to concern itself with displaying the data:

```jsx
ReactDOM.render(<CryptoPriceTable />, document.getElementById('root'));
```

This opens up a number of opportunities for us. First of all, both components are able to be independently tested. I will provide a bit more information in a later chapter about how exactly we can test layout components with snapshot testing.

We also have the opportunity to combine other layout components with the `withCryptoPrices` HOC. To illustrate this, we are going to display the prices in CSV format. Our HOC will remain the same whereas the layout component can be implemented as such:

```jsx
const PriceCSV = ({ isLoading, items, loadData, separator = ';' }) => {
  if (isLoading) {
    return <p>Prices are loaded, please wait.</p>;
  }

  if (!items || items.length === 0) {
    return (
      <p>
        No data available <button onClick={loadData}>Try again</button>
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
};
```

And just like that we have implemented our very first own CSV layout component. We check again whether the data is being loaded, then whether `items` are present. This could also be extracted into another HOC component as HOCs can be nested as many times as you like. In the end, they are all just functions which are passed as parameters to yet another function.

At last, we can render the output: we iterate through the list of `items`, pick the relevant properties `name`, `symbol` and `quotes` via **Object Destructuring** and then wrap the individual lines with a `pre` element to correctly display the end of the line.

In contrast to the `PriceTable` component, we have introduced another optional prop: `separator` - to tell the render component how many separating symbols it should use to display the data. This separator prop can be passed as a simple prop \(as is common in JSX\):

```jsx
const CryptoCSV = withCryptoPrices(PriceCSV);

ReactDOM.render(<CryptoCSV separator="," />, document.getElementById('root'));
```

However, by introducing this change in the CSV component, another change needs to made in the `withCryptoPrices` HOC. So far only the `isLoading`, `items` and `loadData` props are passed to the child component \(`WrappedComponent`\):

```jsx
return (
  <WrappedComponent
    isLoading={isLoading}
    items={items}
    loadData={this.loadData}
  />
);
```

In order to pass the `separator` prop which was defined in `<CryptoCSV separator="," />` correctly to the `PriceCSV` component, we need to inform the **HOC** to also pass other remaining props to the `WrappedComponent`. You can either choose to explicitly pass other props or to only pass the **remaining** ones:

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

`{...this.props}`can be used to pass the remaining props to the child component using Spread Syntax from ES2015+.

{% hint style="info" %}
**Higher Order Components** are a great way to "centralize" logic and structure applications better. Logic can be easily extracted from layout components which would further complicate such components. Even though they have been a fairly new concept in React, the concept itself is a very old one.

**Higher Order Components** are still widely used and there's nothing controversial about their usage. However, newer concepts to achieve the same or similar objectives have been introduced, of which many increase readability. Two of these are **Functions as a Child** and the newer **Context API** — which has been introduced in React in Version 16.3.0. Both of these will be explained in the next two chapters.
{% endhint %}
