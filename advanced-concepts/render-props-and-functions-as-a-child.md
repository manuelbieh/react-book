# Functions as a Child and Render Props

**Functions as a Child** \(FaaC for short\) and **Render Props** are treated separately in the official React documentation. However, **Functions as Children** are already mentioned in the **Render Props** hinting at the fact that they are similar if a little different. Thus, I'd like to discuss them both at the same time. But what even are they?

**Functions as a Child** \(also called **Functions as Children**\) ****and **Render Props** are patterns which allow you to bundle business or application logic in some sort of overarching component - similar to a **Higher Order Component**. In contrast to a HOC though, a function is called which is passed the relevant data as a parameter \(as opposed to returning a new component which receives data as props\). The function takes the form of a child element of the relevant component in the **Function as Children** pattern \(so `this.props.cildren`\). The **Render Props** pattern on the other hand introduces the function as a Prop with the name of `render` \(or any other name\).

We've learned that the value of a prop in JSX can be any valid JavaScript expression. As invoked functions can also return expressions, we can also use the return value of said function as a prop. Strings, Booleans, Arrays, Objects, other React elements and `null` can also be passed as props. `children` are a special form of props, meaning that both of these lines will result in the same output when rendered:

```jsx
<MyComponent>I am a child element</MyComponent>
<MyComponent children="I am a child element" />
```

`props.children` can be used to access _I am a child element_ in `MyComponent`.

We can use this principle and also pass functions which are invoked during `render()` within a component. This way, data can be passed from one component into the next. The principle is similar to that in **Higher Order Components**, but offers a little more flexibility. We do not need to connect our component with a Higher Order Component but can simply be included within JSX in our current component. Thinking back to our `withFormatting` HOC from the previous chapter, a similar approach could look like the following using **Function as a Child \(FaaC\)**:

```jsx
const bold = (string) => {
  return <strong>{string}</strong>
}

const italic = (string) => {
  return <em>{string}</em>
}

const Formatter = (props) => {
  if (typeof props.children !== 'function') {
    console.warn('children prop must be a function!');
    return null;
  }

  return props.children({ bold, italic });
}
```

We have defined two functions: the `bold` function and the `italic` function. `props.children` can then be called in a Formatter function after checking whether the `children` props is actually function. The function takes in an object with two properties: `bold` with the `bold` function as its value and `italic` with the `italic` function as its value.The invoked function is returned by the component.

Using this **Function as Children** component, a function in JSX is passed to the child element:

```jsx
<div>
  <p>This text does not know about the Formatter function</p>
  <Formatter>
  {({ bold }) => (
    <p>This text is {bold('bold though and does')}</p>
  )}
  </Formatter>
</div>
```

This increases flexibility as components no longer need wrapped by a Higher Order Function just to reuse functionality. In contrast to Higher Order Components, it is also possible to pass parameters directly from JSX into a function as a Child component and communicate with it.

Let us look at our second example again which we talked about in the chapter on Higher Order Components. This is what the rendered list pf cryptocurrencies looks like as Function as a Child:

```jsx
class CryptoPrices extends React.Component {
  state = {
    isLoading: true,
    items: []
  };

  componentDidMount() {
    this.loadData();
  }

  loadData = async () => {
    this.setState(() => ({
      isLoading: true
    }));

    const { limit } = this.props;

    try {
      const cryptoTicker = await fetch(
        `https://api.coinmarketcap.com/v2/ticker/?limit=${limit || 10}&convert=EUR`
      );
      const cryptoTickerResponse = await cryptoTicker.json();

      this.setState(() => ({
        isLoading: false,
        items: this.convertResponseToArray(cryptoTickerResponse)
      }));
    } catch (err) {
      this.setState(() => ({
        isLoading: false
      }));
    }
  };

  convertResponseToArray = (response) => {
    return Object.entries(response.data).map(([id, item]) => item);
  };

  render() {
    const { isLoading, items } = this.state;
    const { children } = this.props;

    if (typeof children !== "function") {
      return null;
    }

    return children({
      isLoading, 
      items, 
      loadData: this.loadData 
    });
  }
}
```

At first glance, the example does not look much different to the one we've introduced in the previous chapter using Higher Order Components. But if you pay attention, you will notice some differences:

* No new component is generated and we can work directly with our current component
* The `loadData` method can access `this.props` to read the `limit` prop. This can then be used as a parameter in the API call
* The `render()` method does not return any component that was passed in anymore and calls the `children` function instead which it receives from its own props
* The `children` function receives the `isLoading` state and returns the items.

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
              {item.name} ({item.symbol}): EUR {item.quotes.EUR.price}
            </li>
          ))}
        </ul>
      )
    }
  </CryptoPrices>
</div>
```

