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

