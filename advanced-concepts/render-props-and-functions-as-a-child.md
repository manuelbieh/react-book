# Functions as a Child and Render Props

**Functions as a Child** \(short: _FaaC_\) and **Render Props** are described separately in the official React documentation, whereby **Functions as Children** are also mentioned in the chapter on **Render Props**. Since both work quite identically in principle, I would like to describe the two concepts in one chapter. But first of all: What is it all about?

**Functions as a Child** \(also called **Function as Children** in the official documentation\\) and **Render Props** are patterns that bundle business or application logic in a kind of higher-level component similar to **Higher Order Components**. Unlike **Higher Order Components**, however, no new component is returned and the corresponding data is passed to it as props. Instead, a function is called that receives the corresponding data as a parameter. With the **Function as Children-**Pattern this function is a child element of the component \(so: `this.props.children`\), with the **Render Props-**Pattern it is a prop which in most cases has the name `render` \(so: `this.props.render`\), but could also have any other name.

We already know that the value of a prop in JSX can be any valid expression in JavaScript. Also called functions can return expressions and so we can use not only strings, booleans, arrays, objects, other react elements or `null` as value for our props, but also the return value of a called function. We also learned that `children` in React are only a kind of special form of a prop and so the following two lines each result in the same rendering result:

```jsx
<MyComponent>I am a child element</MyComponent>
<MyComponent children="I am a child element" />
```

In the `MyComponent` component the _I am a child element_ text can be accessed with `props.children`.

We can take advantage of this and pass functions that are then called in the `render()` method of a component. The idea behind this is that any data can be passed from one component to the next in this way. Similar to **Higher Order Components**, but with a little more flexibility. For example, we don't have to "connect" the whole component with a **Higher Order Component**, but can do this right in the middle of the JSX of our component. Let's think back to the `withFormatting` HOC from the previous chapter. A corresponding component implemented as **Function as a Child \(FaaC\)** could look something like this:

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
    return zero;
  }

  return props.children({ bold, italic });
}
```

So we define again a `bold`- and a `italic`-function, check in the `Formatter`-component, if the passed `children`-prop is a function and call this function. Further we pass an object with the properties `bold` with the `bold` function as value and `italic` with the `italic` function as value as the only parameter. At the same time, we return the called function from the component.

When this **Function as Children** component is used, a function is passed in the JSX as a child element. This works as follows:

```jsx
<div>
  <p>This text has no knowledge of the formatter functions </p>
  <Formatter>
  {(bold }) => (
    <p>This text however {bold('very well')}</p>
  )}
  </Formatter>
</div>
```

The benefit of this approach is the said flexibility, since we no longer have to combine the whole component itself with a higher order function, only to perhaps have reusable functionality at a single point. Unlike Higher Order Components, it is also possible in this way to pass parameters directly from the JSX to the Function as a Child component and thus communicate with it.

Let's look again at our second example from the chapter on Higher Order Components, the price list of crypto currencies, and implement it as a Function as a Child:

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
      return zero;
    }

    return children({
      isLoading, 
      items, 
      loadData: this.loadData 
    });
  }
}
```

At first glance, the component does not look so different from the Higher Order Component from the previous chapter. But if you look closely:

* No further component is created and returned, but the component is worked with directly.
* The `loadData` method accesses `this.props` to read the `limit` prop. This is used as a parameter for the API call.
* The `render()` method now does not return a component entered into the component, but instead calls the `children` function, which it gets from its own props.
* The `children` function gets the load status \(`isLoading`\) and finally the items back.

The use of this component is then similar to the one in the first example with the small difference that we can optionally pass a `limit` prop to the component:

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

At this point the `PriceTable` component also comes into play again. This expected exactly the three props we returned from the `CryptoPrices` component. What a coincidence! Let's take a look at how we can connect the two of them:

```jsx
<CryptoPrices limit={5}>
  {({ isLoading, items, loadData }) => (
    <PriceTable isLoading={isLoading} items={items} loadData={loadData} />
  )}
</CryptoPrices>
```

Or to make it short using spread syntax:

```jsx
<CryptoPrices limit={5}>
  {(props) => <PriceTable {...props} />}
</CryptoPrices>
```

In this way, we have ensured a very high degree of flexibility, but do not have to rigidly connect components with the logic via a HOC, which saves us a lot of "organizational effort". 

But be careful: **Functions as a Child components also have a restriction that Higher Order Components** do not have. Namely, the data we get from a **FaaC** component **can only be used within JSX**! So if we want to provide relatively abstract methods via a logic component that is higher in the component hierarchy, this is not possible with a **FaaC** component at first, or only inconveniently possible!

### Render props

But wait a minute, how was that now actually with the **Render Props** and what is that now exactly and how do these differ from **Function as Children** components?

Put simply: only by the name of Prop. Some popular libraries from the React world had at some point started using `render` as the name for props that expect functions as value. And so our `CryptoPrices` component, using a `render` prop instead of `children`, would look like this:

```jsx
<CryptoPrices limit={5} render={(props) => <PriceTable {...props} />} />
```

Within the `CryptoPrices` component it must of course be called:

```jsx
render() {
  const { isLoading, items } = this.state;
  const { render } = this.props;

  if (typeof render !== "function") {
    return zero;
  }

  // Attention: this render() has nothing to do with the same named components method
  // but gets into the component via this.props.render!
  return render({
    isLoading, 
    items, 
    loadData: this.loadData 
  }); 
}
```

So it's also a matter of taste to a certain extent. Of course you are not limited to the name `render`, but can simply pass a function to any prop and make it a _"Render Prop"_. 

It is also possible to have any number of such props in one component. If, for example, you want to implement a component that outputs a table to you that has both a table header and a body that both get data from the component, this is no problem either!

### Render props and FaaCs in conjunction with Higher Order Components

Finally, a little trick. If you actually need a **Higher Order Component** and you already have a **FaaC-** or **Render Prop** component, you can also make it a HOC:

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

In practice, experience has shown that this case will be rather rare.

{% hint style="info" %}
The **Function as a Child** pattern and the nearly identical **Render-Props** pattern are used to separate business logic from presentation components. They are a very lightweight alternative to **Higher Order Components**, which serve pretty much the same application. 

However, unlike HOCs, they can also be used within a `render()` method of components and do not have to be "linked" rigidly to a component. This makes them even more flexible in their applications than **Higher Order Components,** without sacrificing clarity.
{% endhint %}