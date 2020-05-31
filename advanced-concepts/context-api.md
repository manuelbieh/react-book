# Context API

For a long time, the React **Context API** has been treated as somewhat of an afterthought. First only implemented as a prototype and treated experimentally, but later added to React in version **16.3**.

The Context API has been designed to distribute data from a component to so-called data _consumers_ without explicitly passing props through the whole component tree. This is immensely useful for language settings as well as a global styling schema \("Theme"\).

The Context API consists of two main actors: The **Context Provider** as well as the **Context Consumer**. The **Provider** acts as a central instance for the corresponding data structure whereas the **Consumer** can _consume_ this data at any point in the app. It forms some sort of "semi-global" data instance which is only valid in certain parts of the component hierarchy.

This does not mean that the data structure cannot be complex. It is not limited to strings or arrays but can consist of complex data. An application can have an unlimited amount of Contexts \(for example one for the user-chosen language, one for the styling schema etc.\) and Providers can be reused with different values. But let's take it one step at a time.

## API

In order to create a new **Context**, React provides the `createContext` method:

```jsx
const LanguageContext = React.createContext(defaultValue);
```

We have just created a new **Context** using this line of code. The Context now consists of a **Provider** and a **Consumer** component: `LanguageContext.Provider` as well as `LanguageContext.Consumer`.

The **Context** can now be used in the application by wrapping the contents of the tree within a Provider:

```jsx
// LanguageContext.js
import React from 'react';
const LanguageContext = React.createContext('de');
export default LanguageContext;
```

```jsx
// index.js
import React from 'react';
import ReactDOM from 'react-dom';
import LanguageContext from './LanguageContext';

const App = () => (
  <LanguageContext.Provider value={'en'}>
    {/* inside of this tree, the value of 'en' is available to other components  */}
  </LanguageContext.Provider>
);

ReactDOM.render(<App />, document.getElementById('#root'));
```

The `SelectedLanguage` component can now be used at any point within the application. If the value within the Provider changes, all **Consumer components** encompassed within the Provider will render again using the updated value.

A complete if a little artificial example is this:

```jsx
import React from 'react';
import ReactDOM from 'react-dom';
import LanguageContext from './LanguageContext';
import DisplaySelectedLanguage from './DisplaySelectedLanguage';

const App = () => (
  <LanguageContext.Provider value="en">
    <header>Welcome!</header>
    <div className="content">
      <div className="sidebar" />
      <div className="mainContent">
        <DisplaySelectedLanguage />
      </div>
    </div>
    <footer>© 2019</footer>
  </LanguageContext.Provider>
);

ReactDOM.render(<App />, document.getElementById('#root'));
```

Although no **props** have been passed to the `DisplaySelectedLanguage` component, it still has knowledge of the currently selected language and will also demonstrate this accurately:

```markup
<p>The chosen language is en</p>
```

If the `value` within the Provider component changes, all Consumer components will re-render — if they are located within the current Provider's Context.

If we extend the example a little bit, we can easily add a little more structure to make a simple multilingual service.

The following example contains an object with translations to which we pass a rather complex object with multiple data types \(consisting of an array, a string, a function to change the language as well as object with the original translations\):

```jsx
import React from 'react';
import ReactDOM from 'react-dom';

const translationStore = {
  de: {
    greeting: 'Guten Tag!',
    headline: 'Heute lernen wir, wie Context funktioniert.',
  },
  en: {
    greeting: 'Hello!',
    headline: 'Today we will learn how Context works.',
  },
};

const defaultLanguage = 'de';

const defaultLanguageContextValue = {
  availableLanguages: Object.keys(translationStore),
  changeLanguage: () => {
    console.warn('Funktion changeLanguage() nicht implementiert!');
  },
  language: defaultLanguage,
  translations: translationStore[defaultLanguage],
};

const LanguageContext = React.createContext(defaultLanguageContextValue);

class Localized extends React.Component {
  changeLanguage = (newLanguage) => {
    this.setState((state) => ({
      translations: translationStore[newLanguage],
      language: newLanguage,
    }));
  };

  state = {
    ...defaultLanguageContextValue,
    changeLanguage: this.changeLanguage,
  };

  render() {
    return (
      <LanguageContext.Provider value={this.state}>
        {this.props.children}
      </LanguageContext.Provider>
    );
  }
}

const Greeting = () => (
  <LanguageContext.Consumer>
    {(contextValue) => contextValue.translations.greeting}
  </LanguageContext.Consumer>
);

const Headline = () => (
  <LanguageContext.Consumer>
    {(contextValue) => contextValue.translations.headline}
  </LanguageContext.Consumer>
);

const LanguageSelector = () => {
  return (
    <LanguageContext.Consumer>
      {(contextValue) => (
        <select
          onChange={(event) => {
            contextValue.changeLanguage(event.target.value);
          }}
        >
          {contextValue.availableLanguages.map((language) => (
            <option value={language}>{language}</option>
          ))}
        </select>
      )}
    </LanguageContext.Consumer>
  );
};

const App = () => (
  <Localized>
    <LanguageSelector />
    <p>
      <Greeting />
    </p>
    <p>
      <Headline />
    </p>
  </Localized>
);

ReactDOM.render(<App />, document.getElementById('root'));
```

First of all, we define an object `defaultLanguageContextValue` which holds the default value of the newly generated Context object. This consists of:

- an object called `translationStore` which contains all available translations
- a standard language — German in this case \(`de`\) which is saved to a property called `language`
- an array named `availableLanguages` that lists all available languages of the `translationStore` object which we dynamically generate with `Object.keys()` — in our case: `["de", "en"]`
- a placeholder function \(`changeLanguage()`\) which is later replaced with an actual implementation in the `Localized` component. This helps us to avoid the case of incorrectly calling a function which is not yet implemented. Otherwise the warning: _"Function changeLanguage\(\) is not implemented"_.

The `changeLanguage()` function can only be implemented in the component itself as React cannot control the state \(in this case languages and their translations\) anywhere else other than **inside of a component**. We could save the current language settings inside of a global variable, however React would not re-render the component if something changed in this global variable as this value would neither be state nor props.

The `Localized` component serves as a wrapper component for our newly generated Context. We can save and modify the user's selected language here by changing the state accordingly. The `defaultLanguageContextValue` object is saved in the state of the component and the `changeLanguage()` method is also implemented here. This function receives a language \(`de` or `en`\) and modifies the state accordingly, and then it fetches the translations for the chosen language from the `translationStore` object and writes it to the state as new `translations`. If the user changes their language setting from German \(default\) to English, the function overrides all German translations currently in state with the English translations. By calling `this.setState()` a re-render is triggered and all Context Consumers within the component tree will be rendered with the updated value \(which we pass to the Context Provider through the render\(\) method in the component\).

If this all sounds a little complicated so far, do not worry, it will become more intuitive once it is used in practice. I strongly advise you to try the above example yourself and play with the code.

However, there is a little gotcha in the above example: usually the state of a class component is defined first and only then properties and methods of this class will follow. In this example however, we did not follow this convention and implemented the `changeLanguage()` method first. Why? By defining `changeLanguage()` first, we ensure that `this.changeLanguage` will not be `undefined`. Only then, we construct the `state` property of our class.

The code snippet is still relatively complex — and unnecessarily complicated — too. We built our own component for both the headline and the greeting, to provide it with access to a Context Consumer which has access to the object containing the translations. However, we can optimize the code by constructing a generic component with which we can directly access the translations in the `translations` object. This component will be called `Translated` and receives a single prop: the property which we want to access in the `translations` object. In our example, this could either be `greeting` or `headline`.

```jsx
const Translated = ({ translationKey }) => (
  <LanguageContext.Consumer>
    {(contextValue) => contextValue.translations[translationKey]}
  </LanguageContext.Consumer>
);
```

Our `App` component will now look like this:

```jsx
const App = () => (
  <Localized>
    <LanguageSelector />
    <p>
      <Translated translationKey="greeting" />
    </p>
    <p>
      <Translated translationKey="headline" />
    </p>
  </Localized>
);
```

We can then safely eliminate the `Headline` and `Greeting` component from our example.

**Attention:** Especially when dealing with translations, it is not uncommon to denote the key for the translations with "key" . This would surely be a nice addition for our `Translated` component:

```jsx
<Translated key="greeting" />
```

But `key` is a reserved word in React is used to identify elements in arrays and we can therefore not use it in this case. If you want to freshen up your knowledge about these, please refer to the chapter "Lists, Refs, Fragments and Conditional Rendering" in the "Basics" section and check in the section on "Lists".

## Usage of Multiple Contexts

It is entirely possible to use multiple Context Providers within the same component hierarchy. Nesting them is not a problem. Even Providers of the same Context type can be nested inside of each other. The Context value of the _above_ Provider is given to the Consumer components:

```jsx
<MyContext.Provider value="1">
  <MyContext.Provider value="2">
    <MyContext.Consumer>
      {(value) => <p>The value is {value}</p>}
    </MyContext.Consumer>
  </MyContext.Provider>
</MyContext.Provider>
```

The above example is completely valid. The output would be the following:

```markup
<p>The value is 2</p>
```

The **Consumer component** gets its data from the most adjacent **Context Provider** which is the one passing the value of "2".

Although it does not make sense to nest the _same_ **Context Providers** within each other, it is not uncommon or incorrect to use _different_ **Context Providers** within each other. An application can consist of a Theme Provider, a Language Provider and an Account Provider. The latter would take care of data handling for logged in users and manage access tokens or user-specific settings.

## Abbreviation: contextType

While using Class components, we can employ a trick which allows us to avoid building another Consumer component bloating our component tree further.

In order to do this, we can use `contextType`: it can be assigned to a class component in the form of a static property. The Context value can then be accessed within the component via `this.context`. The value of the `contextType` property is created by `React.createContext()` which you need to call beforehand.

But be careful: It is only possible to assign **a single Context type** to a class. If we want to access two or more Contexts, we have to wrap the respective JSX in a Consumer component. By using _Public Class Fields Syntax_ from ES2015+, it is sufficient to define a static class property `contextType` and to assign it a Context.

If applied to our previous `Translated` component, the result would be:

```jsx
class Translated extends React.Component {
  static contextType = LanguageContext;
  render() {
    return this.context.translations[this.props.translationKey];
  }
}
```

The value of the current `LanguageContext` is assigned to the static `contextType` property of the component \(which is no longer a Function component but a Class component\). Its value can be read by accessing `this.context`.

Without using Public Class Fields Syntax \(which I strongly encouraged in the previous chapters\), the above code would look like the following:

```jsx
class Translated extends React.Component {
  render() {
    return this.context.translations[this.props.translationKey];
  }
}

Translated.contextType = LanguageContext;
```

The `contextType` would be defined outside of the component and no longer inside of it. In the end, it all boils down to personal taste, and does not have any real implications. We are only using another form of syntax which is only allowed in later versions of ECMAScript or provided by Babel via transpiling \(by using the babel plugin `@babel/plugin-proposal-class-properties`\).

## Performance Gotchas

React massively optimizes **Context** under the hood in order to avoid unnecessary re-renders of components or avoid lengthy component hierarchies. Comparing the old value of the Context Provider with the new one, **Consumer components** are only re-rendered if the value in the **Context Provider** has actually changed.

While this sounds easy for a change, it does create a little gotcha which we need to look out for. It concerns the Context Provider whose value is recreated on-the-fly if we use it within the `render()` method. It is recommended to create the value of the Context outside of the `render()` method and pass a **reference** to the value instead of a **newly created value**.

Here is an example which you should NOT recreate:

```jsx
class App extends React.Component {
  state = {
    color: 'red',
  };

  render() {
    return (
      <Provider value={{ color: this.state.color }}>
        <MoreComponents />
      </Provider>
    );
  }
}
```

We are creating a new object `{color: this.state.color}` with each call of the `render()` method. As React only checks if the reference of the `value` in the current `render()` is the same as the reference in the previous `render()` call \(which is never the case as a new object is created on the fly each time\), all Consumer components would re-render.

However, we can transform the above example to avoid this situation and make sure that React's performance algorithm can get to work:

```jsx
class App extends React.Component {
  state = {
    color: 'red',
  };

  render() {
    return (
      <Provider value={this.state}>
        <MoreComponents />
      </Provider>
    );
  }
}
```

In this example, we are merely passing a _reference_ to the `state` object of the component. As this remains intact during the re-renders of the component, it does not trigger a re-render if the content of the state did not change.
