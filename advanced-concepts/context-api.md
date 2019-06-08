# Context API

The **Context API** was neglected in React for a very long time, at first only prototypically implemented and described as experimental, before it became an official part of React in React **16.3.** in a fundamentally revised form.

It was designed to distribute data within a component hierarchy to so-called _consumers_ without having to explicitly pass props to each individual component. In some cases, this can be very tedious if the data is shared by many components within the tree. These include, for example, language settings or a global style scheme \("Theme"\).

In such cases, the **Context API\*\***, which consists of one **Context** _**Provider**_ and any number of **Context** _**Consumers**_, is the best choice. The **Provider** serves here as a kind of central instance for the respective data structure, the **Consumer** can then _consume_ the corresponding data provided by the provider. A "semi-global" data instance, so to speak, that only applies to a specific tree within the component hierarchy.

The data structure can be very complex and is not limited to simple data types like strings or arrays. An application can have any number of contexts \(e.g. one for the language set by the user, one for the style scheme, etc.\) and even a provider itself can be used multiple times with changing values. But one thing at a time.

## API

To create a new **Context**, React provides the `createContext` method:

```jsx
const LanguageContext = React.createContext(defaultValue);
```

With only this one line we have already created a new **Context**. The **Context** now consists of a **Provider-** and a **Consumer-** component: `LanguageContext.Provider` and `LanguageContext.Consumer`.

Within our application, the **Context** can now be used by enclosing a specific tree with a provider:

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
    {/* within this tree we now have the value 'en' at our disposal */}
  </LanguageContext.Provider>
);

ReactDOM.render(<App />, document.getElementById('#root'));
```

If we now want to access the value of the **Context** in a component, we enclose a component and make use of the **Function as a Child** principle we looked at in the previous chapter:

```jsx
// DisplaySelectedLanguage.js
import React from 'react';
import LanguageContext from './LanguageContext';

const DisplaySelectedLanguage = () => (
  <LanguageContext.Consumer>
    {(value) => (<p>The selected language is {value}</p>)}
  </LanguageContext.Consumer>
);

export default DisplaySelectedLanguage;
```

We can now use the `SelectedLanguage` component anywhere within our application and always have the value provided by the provider available there. If the value in the **Provider,** changes, all **Consumer-** components below the corresponding provider will also be re-rendered with the updated value!

A complete example, even if it is quite constructed, can look like the following:

```jsx
import React from 'react';
import ReactDOM from 'react-dom';
import LanguageContext from './LanguageContext';
import DisplaySelectedLanguage from './DisplaySelectedLanguage';

const App = () => (
  <LanguageContext.Provider value="en">
    <header>Hearly welcome</header>
    <div className="content">
      <div className="sidebar"></div>
      <div className="mainContent">
        <DisplaySelectedLanguage />
      </div>
    </div>
    <footer>© 2019</footer>
  </LanguageContext.Provider>
);

ReactDOM.render(<App />, document.getElementById('#root'));
```

Although we don't pass any **props** to the DisplaySelectedLanguage component, it has knowledge of the currently selected language and displays correctly:

```markup
<p>The selected language is en</p>
```

If the `value` of a provider component changes, all consumer components within this provider will be re-rendered!

If we extend the example, we can build a relatively simple small component structure to realize multilingualism in an application, for example.

In the following example we create an object with translations and enter a relatively large object with different data types \(consisting of an array, a string, a function to change the language and an object with the actual translations\) into the context:

```jsx
import React from "react";
import ReactDOM from "react-dom";

const translationStore = {
  {
    greeting: "Hello!",
    headline: "Today we learn how context works."
  },
  en: {
    greeting: "Good day!",
    headline: "Today we learn how context works."
  },
};

const defaultLanguage = "de";

const defaultLanguageContextValue = {
  availableLanguages: Object.keys(translationStore),
  changeLanguage: () => {
    console.warn('Function changeLanguage() not implemented!');
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
    <p><Greeting /></p>
    <p><Headline /></p>
  </Localized>
);

ReactDOM.render(<App />, document.getElementById("root"));
```

First we define an object `defaultLanguageContextValue`, which represents the default value of our new context object. This consists of:

* an object `translationStore`, which contains all existing translations. 
* a default language English \(`de`\) stored in the `language` property
* an array \(`availableLanguages`\) with all available languages from the `translationStore` object, which we create dynamically from the properties on first level using `Object.keys()` \(in our example `['de', 'en']`\).
* a placeholder function \(`changeLanguage()`\), which is later replaced in the `Localized` component by a real implementation. This ensures that we do not run the risk of calling a function that does not yet exist if the context is used incorrectly. In this case the warning would be output _"Function changeLanguage\(\) not implemented!"_.

The `changeLanguage()`-function can only be implemented later in the component itself, because otherwise React wouldn't be able to change the state  \(i.e. in this concrete case the language and the translations\) with on-board means, because state for React only exists **within a component**. For example, we could save the current language setting in a global variable, but then React would not render the component again if it were changed, because neither props nor state have changed, but this is a condition for React to render the page tree again.

The `Localized` component now serves as a wrapper component for our newly created context. In it, \(and change!\) we store the language selected by the user by setting the state accordingly. We save the `defaultLanguageContextValue` object in the state of the component and additionally implement the `changeLanguage()` method here. This receives a language \(thus `de` or `en`\), modifies the state accordingly, gets the translations for the newly selected language from the `translationStore` object and writes them as `translations` into the state. For example, if the user changes the language from German \(default\) to English, the function overwrites all German translations in the state with the English translations. When `this.setState()` is called, a rendering is triggered and all context consumers within the component tree are re-rendered with the updated value that we pass to the context provider in the render\(\) method of the component.

The whole thing sounds complicated at first, but in practice it's actually not that difficult. At this point I would like to encourage you to try the above example live.

By the way, there is a pitfall in the above component: Usually the state is defined first in a class component and only then all other properties and methods of the class are defined. In this case, however, we deviated from the convention and implemented the `changeLanguage()` method first. This has the simple reason that `this.changeLanguage` otherwise would not be defined, i.e. `undefined`. To work around this, we define the method before constructing the `state` property of the class.

Now our code example is relatively complex and more complex than it actually should be. And so we have created a separate component for the headline and the greeting text, only to use a context consumer in each of them to have access to the object with our translations. Here the code can be optimized in such a way that we create a generic component to directly access certain translations from the `translations` object. We call this component Translated and the only prop it gets is the property we want to access in the `translations` object. In our concrete example this could be `greeting` or `headline`.

```jsx
const Translated = ({ translationKey }) => (
  <LanguageContext.Consumer>
    {(contextValue) => contextValue.translations[translationKey]}
  </LanguageContext.Consumer>
)
```

Our `App` component will look like this:

```jsx
const App = () => (
  <Localized>
    <LanguageSelector />
    <p><Translated translationKey="greeting" /></p>
    <p><Translated translationKey="headline" /></p>
  </Localized>
);
```

The `Headline` and `Greeting` components from the previous example can then simply be saved.

**Attention:** Especially for translations it is not uncommon to call the keys for the translations simply "key" and so it would have a certain charm to call the prop in the `Translated`-component `key`. That would be nice and short and easy to read:

```jsx
<Translated key="greeting" />
```

However, React does us a favor here, since `key` is a reserved name in JSX and is used to identify elements when these elements are used in an array. The exact reasons for this can be found in the chapter on "Lists, Refs, Fragments and Conditional Rendering" in "The Basics", there specifically in the section "Lists".

## Using multiple contexts

It is no problem to have multiple context providers within a component hierarchy. Nesting several provider components is therefore not a problem. Even providers of the same context type can be nested into each other. The context value of the next higher provider is always passed to the consumer components:

```jsx
<MyContext.Provider value="1">
  <MyContext.Provider value="2">
    <MyContext.Consumer>
      {(value) => <p>The value is {value}</p>}
    </MyContext.Consumer>
  </MyContext.Provider>
</MyContext.Provider>
```

The above example would therefore be possible without any problems. The issue would be here:

```markup
<p>The value is 2</p>
```

The **Consumer component** gets its data here from the next higher **Context provider**, which has the value "2" in the above example.

If the nesting of the same **Context-Providers** usually makes relatively little sense, it is still no unusual or even bad practice to nest different **Context-Providers** into each other. So an application can easily consist of a theme provider, a language provider and an account provider. The latter would then, for example, take care of the data handling of the logged in user, if necessary manage access tokens or user-specific settings.

## Abbreviation: contextType

When using class components, we can use a trick and avoid using a consumer component that further inflates our component tree.

The `contextType` is the keyword here. This can be assigned to a class component in the form of a static property of the same name, then the value of the respective context can be accessed within the component using `this.context`. The value assigned to the `contextType` property is a context that was previously created using `React.createContext()`.

However, it is only possible to assign a single **context type** to each class. If we want to access the value of two or more contexts, we have to wrap our JSX into consumer components again. When using the _Public Class Fields Syntax_ from ES2015+, it is sufficient to define a static class property `contextType` and assign a context to it.

As an example, applied to the Translated component from above, this would look something like this:

```jsx
class Translated extends React.Component {
  static contextType = LanguageContext;
  render() {
    return this.context.translations[this.props.translationKey];
  }
}
```

We assign the static `contextType` property of the component \(which is now a class component and no longer a function component\) as a value to our `LanguageContext` and already we have the value of the context available in `this.context`.

Without the use of the Public Class Fields syntax \(which I strongly recommended some chapters before because it makes life easier in some places\) the same code would look like this:

```jsx
class Translated extends React.Component {
  render() {
    return this.context.translations[this.props.translationKey];
  }
}

Translated.contextType = LanguageContext;
```

So we would define the `contextType` outside the component and not inside it anymore. Basically this is a matter of taste and has no other implications or disadvantages. It is alone another syntax variant, which is possible only in later ECMAScript versions or even by transpiling by means of Babel. It is made possible by the Babel plugin `@babel/plugin-proposal-class-properties`.

## Performance pitfall

React massively optimizes **Context** under the hood to avoid unnecessary rendering of components or even entire component hierarchies. For this purpose, the old context value of the provider is compared with the new value for each rendering and **consumer components** are then only rendered again if the value of their **context provider** has changed.

What sounds relatively simple in theory for once, but brings with it a small stumbling block. This concerns the consumer provider whose value is always generated on-the-fly within the `render()` method of a component. Therefore, it is normally recommended to generate the context value outside the render\(\) method and pass a **reference** to the value instead of a **newly generated** value.

To illustrate this, here is a negative example:

```jsx
class App extends React.Component {
  state = {
    color: 'red',
  };

  render() {
    <Provider value={{color: this.state.color}}}>
      <MoreComponents />
    </Provider>
  }
}
```

In this case we create a new object `{color: this.state.color}` each time we call the render\(\) method, which we use as a value for the context provider. Since React only checks whether the reference to the corresponding `value` in the current `render()` call is the same as in the previous `render()` call, but this is never the case here, since a new object is created on the spot, all consumer components are re-rendered here.

However, the above example can easily be rewritten to show that React's performance optimizations work here:

```jsx
class App extends React.Component {
  state = {
    color: 'red',
  };

  render() {
    <Provider value={this.state}>
      <MoreComponents />
    </Provider>
  }
}
```

In the second example, we simply pass a _reference_ to the `state` object of the component. Since this is also retained during component rendering, this procedure does not trigger any rendering as long as the content of the state of the component does not change!

