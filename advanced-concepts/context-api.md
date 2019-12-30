# Context API

For a long time, the React **Context API** has been treated as somewhat of an afterthought. First only implemented as a prototype and treated experimentally, but later added to React in version **16.3**. 

The Context API has been designed to distribute data from a component to so-called data _consumers_ without explicitly passing props through the whole component tree. This is immensely useful for language settings as well as a global styling schema \("Theme"\).

The Context API consists of two main actors: The **Context Provider** as well as the **Context Consumer**.The **Provider** acts as a central instance for the corresponding data structure whereas the **Consumer** can _consume_ this data at any point in the app. It forms some sort of "semi-global" data instance which is only valid in certain parts of the component hierarchy.

This does not mean that the data structure cannot be complex. It is not limited to Strings or Arrays but can consist of complex data. An application can have an unlimited amount of contexts \(for example one for the user-chosen language, one for the styling schema etc.\) and providers can be reused with different values. But let's take it one by one.

## API

In order to create a new **context**, React provides the method of `createContext`:

```jsx
const LanguageContext = React.createContext(defaultValue);
```

We've just created a new **context** using this line of code. The context now consists of a **Provider** and a **Consumer** component: `LanguageContext.Provider` as well as `LanguageContext.Consumer`.

The **context** can now be used in the application by wrapping the contents of the tree within a Provider:

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

The `SelectedLanguage`component can now be used at any point within the application and the value made available by the **Provider**. If the value within the provider changes, all **consumer components** encompassed within the current provider will render again using the updated value.

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

Although no **props** have been passed to the `DisplaySelectedLanguage` component, it still has knowledge of the currently selected language and will also demonstrate this accurately:

```markup
<p>The chosen language is en</p>
```

If the `value` within the Provider component changes, all consumer components will re-render if they are located within the current Provider's context.

If we extend the example a little bit, we can easily add a little more component structure to make multilingual services a reality.

The following example contains an object with translations to which we pass a rather complex object with multiple data types \(consisting of an array, a string, a function to change the language as well as object with the original translations:

```jsx
import React from "react";
import ReactDOM from "react-dom";

const translationStore = {
  de: {
    greeting: "Guten Tag!",
    headline: "Heute lernen wir, wie Context funktioniert.",
  },
  en: {
    greeting: "Good day!",
    headline: "Today we learn how context works.",
  },
};

const defaultLanguage = "de";

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
    <p><Greeting /></p>
    <p><Headline /></p>
  </Localized>
);

ReactDOM.render(<App />, document.getElementById("root"));
```

