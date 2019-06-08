# Principles and rules of Hooks

The use of **Hooks** is linked to some principles and rules, which have to be observed, so that there are no unpleasant surprises and unexpected bugs, of which **Hooks** are supposed to have less. We are assisted by **ESLint** and the ESLint plugin `eslint-plugin-react-hooks` developed by the React team itself to comply with these rules. I had already recommended the use of ESLint in the chapter about **Tools and Setup** and this recommendation has not changed since then.

To install the plugin in the project, the following command line call helps:

```bash
npm install --save-dev eslint-plugin-react-hooks
```

or with yarn:

```bash
yarn add --dev eslint-plugin-react-hooks
```

Then the `.eslintrc` configuration must be modified as follows:

```javascript
{
  "plugins": [
    // ...
    "react-hooks"
  ],
  "rules": {
    // ...
    "react-hooks/rules-of-hooks": "error",
    "react-hooks/exhaustive-deps": "warn"
  }
}
```

For those of you who are using **Create React App**, have good news: You don't need to do anything else, the two rules come standard in the current versions of **Create React App** automatically into your setup!

### The rules of hooks

We would have clarified the formalities now, however, now first of all one after the other. So what are these legendary rules that I mentioned at the beginning and for which there are even specially developed ESLint rules?

#### Hooks can only be used in React Function Components!

Not in **class components** and nowhere else: **Hooks** can only be used within React **Function Components**! This means that a function using hooks must be a react component, i.e. it must inevitably also have a return value that is a valid react element. So JSX, arrays, strings or zero.

{% hint style="danger" %}
**Not allowed because a class component is used: **
{% endhint %}

```jsx
class MyComponent extends React.Component {
  render() {
    const [value, setValue ] = useState();
    return (
      <input type="text" onChange={(e) => setValue(e.target.value)} />
    );
  }
}
```

{% hint style="success" %}
**Allowed, because a Function Component is used:**
{% endhint %}

```jsx
const MyComponent = () => {
  const [value, setValue ] = useState();
  return (
    <input type="text" onChange={(e) => setValue(e.target.value)} />
  );
}
```

#### Hooks may only be used on top level within their Function Component!

In concrete terms, this means that it is **not possible** to use hooks within **loops, conditions, or nested functions**. This is related to how React processes **Hooks** internally. Here it is important that the order in which **Hooks** are executed must be identical for each rendering of a component. For this reason, for example, it is not possible to call a **Hook** only if a certain condition is fulfilled. Depending on whether the condition is fulfilled, this would lead to a different order when processing the **Hooks**. However, it is possible to use conditions _within_ **Hooks**.

{% hint style="danger" %}
**Not allowed, because the hook is within a condition:**
{% endhint %}

```javascript
if (title) {
  useEffect(() => {
    document.title = title;
  }, [title])
}
```

{% hint style="success" %}
**Allowed, because the condition is inside the hook:**
{% endhint %}

```javascript
useEffect(() => {
  if (title) {
    document.title = title;
  }
}, [title])
```

If you install the ESLint plugin and activate the rules, you have nothing to worry about. ESLint then displays a warning if these rules are violated and alerts the developer to his error.

