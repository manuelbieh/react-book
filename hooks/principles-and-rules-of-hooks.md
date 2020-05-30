# Principles and Rules of Hooks

If we want to use **Hooks** and implement them in our applications, we need to follow a few rules that prevent us from running into unexpected errors or behavior. **ESLint** and the `eslint-plugin-react-hooks` package, which has been developed by the React Team itself, help us to follow these rules and indicate when you might be breaching one. I have advocated for the use of **ESLint** in the chapter on **Tools and Setup** already and recommend that you use it if you have not done so already.

To install the plugin, you can execute the following command on the command line:

```bash
npm install --save-dev eslint-plugin-react-hooks
```

or using Yarn:

```bash
yarn add --dev eslint-plugin-react-hooks
```

In addition, you need to amend the `.eslintrc` as shown:

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

I've got good news for those of you who are using **Create React App**: you do not need to do any of these previous steps as those two specific rules ship with **Create React App** out of the box!

### The rules of Hooks

We talked about the formalities and how to check that we are not violating these pre-defined Hooks rules. But what are those rules exactly?

#### Hooks can only be used in React function components

**Hooks** can only be called in React function components, not in **class components** or anywhere else. This means that a function that uses Hooks always has to be a React component, meaning it always has a return value (either JSX, Arrays, Strings or _null_).

{% hint style="danger" %}
**This is now allowed as it uses a Class component:**
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

However:

{% hint style="success" %}
**The following snippet is allowed as it uses a function component:**
{% endhint %}

```jsx
const MyComponent = () => {
  const [value, setValue ] = useState();
  return (
    <input type="text" onChange={(e) => setValue(e.target.value)} />
  );
}
```

#### Hooks are only allowed to be used on the highest layer inside of the function component

It is **not possible** to use **Hooks** inside of **loops, conditions or nested functions**. Why you might wonder? This is due to how React treats **Hooks** internally. The order in which **Hooks** are executed has to be identical for each re-render of the component and explains why it is not possible to call a **Hook** conditionally. If we did in fact executed a **Hook** based on a condition, we would change the order in which the **Hooks** are being executed. We can use conditions _inside_ of **Hooks** though!

{% hint style="danger" %}
**Not allowed: Hook is used inside of a condition**
{% endhint %}

```javascript
if (title) {
  useEffect(() => {
    document.title = title;
  }, [title])
}
```

{% hint style="success" %}
**Allowed: condition is used inside of Hook**
{% endhint %}

```javascript
useEffect(() => {
  if (title) {
    document.title = title;
  }
}, [title])
```

If you installed the ESLint plugin as outlined above and configured the `.eslintrc` to use the rules as described, you don't have to fear accidentally run into any of those errors. ESLint will prompt you with a warning as to how you violated one of the rules.

