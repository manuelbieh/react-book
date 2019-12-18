# Higher Order Components

**Higher Order Components** \(or **HOC** or **HOCs** for short\) were and are a central concept in React. They allow you to implement components with reusable logic and are loosely tied to **Higher Order Functions** from functional programming. These kind of functions take a function as a parameter and return another function. In terms of React, the principle is applied to components. **Higher Order Components** derive their name from those **Higher Order Functions**.

Let us look at an example to illustrate this concept better:

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

We've defined a function called `withFormatting` that takes in a React component. The function will return a new React component which in turn renders the component that was passed into the function and equips it with the props `bold` and `italic`. The component can now access these props:

```jsx
const FormattedComponent = withFormatting(({ bold, italic }) => (
  <div>
    This text is {bold('bold')} and {italic('italic')}.
  </div>
));
```

Typically, **Higher Order Components** can be used to encapsulate logic. They relate closely to the concept of **smart** and **dumb** components. Smart components \(which also encompass **Higher Order Components**\) are used to display business logic, deal with API communication or behavioral logic. _Dumb components_ in contrast mostly get passed static props and keep logic to a minimum which is only used to for display-logic. For example, it might decide whether to show a profile image or, if it is not present, show a placeholder image instead. Sometimes, we also refer to **Container Components** \(for _Smart_ components\) and **Layout Components** \(for _Dumb_ components\).

