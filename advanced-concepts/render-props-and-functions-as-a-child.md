# Functions as a Child and Render Props

**Functions as a Child** \(FaaC for short\) and **Render Props** are treated separately in the official React documentation. However, **Functions as Children** are already mentioned in the **Render Props** section hinting at the fact that they are similar — if a little different. Thus, I would like to discuss them both at the same time. But what are they even?

**Functions as a Child** \(also called **Functions as Children**\) and **Render Props** are patterns which allow you to bundle business or application logic in some sort of overarching component - similar to a **Higher Order Component**. In contrast to a HOC though, a function is called which is passed the relevant data as a parameter \(as opposed to returning a new component which receives data as props\). The function takes the form of a child element of the relevant component in the **Function as Children** pattern \(so `this.props.children`\). The **Render Props** pattern on the other hand introduces the function as a prop with the name of `render` \(or any other name\).

We've learned that the value of a prop in JSX can be any valid JavaScript expression. As invoked functions can also return expressions, we can also use the return value of this function as a prop. Strings, Booleans, Arrays, Objects, other React elements and `null` can also be passed as props. `children` are a special form of props, meaning that both of these lines will result in the same output when rendered:

```jsx
<MyComponent>I am a child element</MyComponent>
<MyComponent children="I am a child element" />
```

`props.children` can be used to access _I am a child element_ in `MyComponent`.

We can use this principle and also pass functions which are invoked during `render()` within a component. This way, data can be passed from one component into the next. The principle is similar to that of **Higher Order Components**, but offers a little more flexibility. We do not need to connect our component with a Higher Order Component but can simply be included within JSX in our current component. Thinking back to our `withFormatting` HOC from the previous chapter, a similar approach could look like the following using a **Function as a Child \(FaaC\)**:

```jsx
const bold = (string) => {
  return <strong>{string}</strong>;
};

const italic = (string) => {
  return <em>{string}</em>;
};

const Formatter = (props) => {
  if (typeof props.children !== 'function') {
    console.warn('children prop must be a function!');
    return null;
  }

  return props.children({ bold, italic });
};
```

We have defined two functions: the `bold` function and the `italic` function. `props.children` can then be called in a formatter function after checking whether the `children` props are actually a function. The function takes in an object with two properties: `bold` with the `bold` function as its value and `italic` with the `italic` function as its value. The invoked function is returned by the component.

Using this **Function as Children** component, a function in JSX is passed to the child element:

```jsx
<div>
  <p>This text does not know about the Formatter function</p>
  <Formatter>{({ bold }) => <p>This text {bold('does though')}</p>}</Formatter>
</div>
```

This increases flexibility as components no longer need wrapped by a Higher Order Function just to reuse functionality. In contrast to Higher Order Components, it is also possible to pass parameters directly from JSX into a function as a Child component and communicate with it.

Let's look at our second example again which we talked about in the chapter on Higher Order Components. This is what the rendered list of cryptocurrencies looks like as Function as a Child:

```jsx
class CryptoPrices extends React.Component {
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

    const { limit } = this.props;

    try {
      const cryptoTicker = await fetch(
        `https://api.coingecko.com/api/v3/coins/markets?vs_currency=eur&per_page=${limit ||
          10}`
      );
      const cryptoTickerResponse = await cryptoTicker.json();

      this.setState(() => ({
        isLoading: false,
        items: cryptoTickerResponse,
      }));
    } catch (err) {
      this.setState(() => ({
        isLoading: false,
      }));
    }
  };

  render() {
    const { isLoading, items } = this.state;
    const { children } = this.props;

    if (typeof children !== 'function') {
      return null;
    }

    return children({
      isLoading,
      items,
      loadData: this.loadData,
    });
  }
}
```

At first glance, the example does not look much different to the one we have introduced in the previous chapter using Higher Order Components. But if you pay attention, you will notice some differences:

- No new component is generated and we can work directly with our current component
- The `loadData` method can access `this.props` to read the `limit` prop. This can then be used as a parameter in the API call
- The `render()` method does not return any component that was passed in anymore and calls the `children` function instead which it receives from its own props
- The `children` function receives the `isLoading` state and returns the items.

Using this component is similar to that from our previous example, with the exception that we can also pass an optional `limit` prop in this case:

```jsx
<div>
  <h1>Current Crypto Currency Prices</h1>
  <CryptoPrices limit={5}>
    {({ isLoading, items }) =>
      isLoading ? (
        <p>Loading prices. Please be patient.</p>
      ) : (
        <ul>
          {items.map((item) => (
            <li>
              {item.name} ({item.symbol}): EUR {item.current_price}
            </li>
          ))}
        </ul>
      )
    }
  </CryptoPrices>
</div>
```

We can now combine the `PriceTable` component which expects three props with the `CryptoPrices` component \(that returns the values needed in the `PriceTable`\).

```jsx
<CryptoPrices limit={5}>
  {({ isLoading, items, loadData }) => (
    <PriceTable isLoading={isLoading} items={items} loadData={loadData} />
  )}
</CryptoPrices>
```

Or even more succinct using spread syntax:

```jsx
<CryptoPrices limit={5}>{(props) => <PriceTable {...props} />}</CryptoPrices>
```

We have allowed for a great deal of flexibility in this example and do not explicitly need to tie a component to a HOC to enable logic \(which saves us valuable time and effort\).

But be careful: **Functions as a Child Components have limitiations that Higher Order Components do not have**. The data that is received by a **FaaC** component can also be used **within JSX**. If we wanted to include highly abstract methods in logic components higher up in the component hierarchy, it would not be possible using **FaaCs**.

## Render Props

But wait — what are **Render Props** and how to they differ from **Function as Children** components?

Put simply, they differ in the name of the prop. Some popular libraries eventually started using `render` as a name for a prop which expects functions as their values. In our `CryptoPrices` component, we would then use `render` instead of `children`:

```jsx
<CryptoPrices limit={5} render={(props) => <PriceTable {...props} />} />
```

Within the `CryptoPrices` component, we can write:

```jsx
render() {
  const { isLoading, items } = this.state;
  const { render } = this.props;

  if (typeof render !== "function") {
    return null;
  }

  // Careful: render() does not have anything to do with the component with
  // the same name and gets injected via this.props.render
  return render({
    isLoading,
    items,
    loadData: this.loadData
  });
}
```

It is personal preference to a degree. You do not need to give this prop the name of `render` and could theoretically choose any valid name. Any passed function will eventually turn the component into a "Render Prop".

It is possible to have an arbitrary number of props in such a component. If you were to implement a component which returns a table which includes a table head and a body, both receiving data from the data component, that would be no problem at all.

<div class="force-break-before"></div>

## Render Props and FaaCs in combination with Higher Order Components

Here is a neat little trick: If you ever need a **Higher Order Component** but you only have a **FaaC** or **Render Prop** component, you can turn these into an HOC like this:

```jsx
function withCryptoPrices(WrappedComponent) {
  return class extends React.Component {
    render() {
      return (
        <CryptoPrices>
          {(cryptoPriceProps) => (
            <WrappedComponent {...this.props} {...cryptoPriceProps} />
          )}
        </CryptoPrices>
      );
    }
  };
}
```

However, you will not need to do this much in practice.

{% hint style="info" %}
The **Function as a Child** pattern and the **Render-Props** pattern are both used to separate business logic and layout in different components. They are a more lightweight alternative to **Higher Order Components**, which cater to a similar use case.

In contrast to a HOC, they can be easily used within the `render()` method of a component and do not need "linked" with another additional component making them more flexible and readable than **Higher Order Components**.
{% endhint %}
