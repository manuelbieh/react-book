# Internationalization

Apart from **Routing** and **State Management**, **internalization** is the last important topic left to discuss. Internalization deals aims to solve the problem of effectively displaying different interfaces to users who speak another language. A German user will be prompted to interact with a German interface whereas an English user will be given an English interface to interact with.

I do not want to dive deeply into the general topic of **internalization** \(or **i18n** for short\) but rather focus on **internalization** in **React**. I assume that you have a very rough understanding of internalization in user interfaces and thus focus on how internalization can be implemented in **React**.

I have shown you a very simple example on how internalization can be achieved using the **Context API** in the respective chapter for the **Context API**. As complexity of our applications grows though, the complexity of issues regarding internalization grow as well. Using the plural in different languages or meaningful placeholders suddenly become important parts of the applications we're building. In order to be prepared well to deal with such cases, it is recommended to use one of the well-known internalization packages which have been developed to cater specifically to these use cases.

The **React ecosystem** boasts with a number of options to choose from. The most notable options form **Lingui**, **Polyglot**, **i18next** or **react-intl**. Facebook has developed its own framework for internationalization called **FBT**. In this chapter we will mainly look at `i18next` and its react bindings: `react-i18next`.

You might wonder why. I have worked with both `react-intl` and `react-i18next` extensively in the past and have also evaluated the alternatives I mentioned above. Through these evaluations and my extensive use of `react-intl` and `react-i18next`, I have found **i18next** to be the best package for a number of reasons. It supports a number of different frameworks, libraries and platforms \(apart from **React**\), and offers a very big and active community. Moreover, it works seamlessly client-side as well as server-side in Node.js. Most of the bigger translation services offer **i18next** as one of their export formats.

**i18next** was also one of the first packages to implement support and even implement optimizations for React Hooks, without comprising backwards compatibility. The API has remained simple and easy to use in that process and still offers the greatest amount of flexibility. The package is complete and caters to most use cases a developer could possibly imagine making it a very solid choice for us.

### Setup of i18next

In order to install this package, we enter the following lines into the terminal window:

```bash
npm install i18next react-i18next
```

 Or if you are using yarn:

```bash
yarn add i18next react-i18next
```

We install `i18next`, the **internationalization framework**, as well as `react-i18next`, the associated **React bindings**. These offer a number of component and functions which will simplify the work with i18next in React. Similar to the principle which we have already seen in the chapter on state management with `redux` and `react-redux`.

Let's start by creating two objects containing our translations. One will hold texts in German while the other will hold texts in English:

```javascript
const de = {
  greeting: 'Hallo Welt!',
  headline: 'Heute lernen wir Internationalisierung mit React',
  messageCount: '{{count}} neue Nachricht',
  messageCount_plural: '{{count}} neue Nachrichten',
};

const en = {
  greeting: 'Hello world!',
  headline: 'Today we learn Internationalization with React',
  messageCount: '{{count}} new message',
  messageCount_plural: '{{count}} new messages',
};
```

To use **i18next** in our application, we have to import it, initialize it and pass the React plugin. Ideally, all these steps should happen somewhere in the beginning of our application meaning before our app component is given over to `ReactDOM.render()`.

We start by importing the `i18next` package as well as the named export `initReactI18next` from the `react-i18next` package:

```javascript
import i18next from 'i18next';
import { initReactI18next } from 'react-i18next';
```

Consequently, we can make use of the `.use()` method to pass the React plugin to **i18next** as well as using the `.init()` method to initialize **i18next**.

```javascript
i18next
  .use(initReactI18next)
  .init({ ... });
```

 The `init()` function expects a config object which should at least contain a `lng` property as well as a `resources` property. `lng` indicates the chosen language whilst `resources` will contain the actual translations. It is also useful to define a `fallbackLng` which can be used if one of the translations chosen is not available in the chosen language. **i18next** offers up to 30 different configuration options, however the three I have mentioned should be enough for the moment:

```javascript
i18next
  .use(initReactI18next)
  .init({
    lng: 'en',
    fallbackLng: 'en',
    resources: {
      en: {
        translation: en,
      },
      de: {
        translation: de,
      }
    },
  });
```

The main language of the application as well as the fallback language is initially set to English. The `resources` object which follows needs a little more explanation. It follows the following pattern:

```text
[Language].[Namespace].[Translation Key] = Translation
```

It should be clear what we are referring to with language. This can either be `en` for English, `de` for German or even `de-AT` for German with a focus on the Austrian region. The property has object value which contains at least one up to an indefinite amount of **namespaces**.

The **namespace** is a central feature of **i18next** which can be used if we so please. It allows for splitting large translation files into different parts which could be dynamically lazy loaded. While this feature is not necessary for smaller applications, it can be game-changer for larger and more complex applications. It will help us to contain the size of the translations and to aid the readability of the translations, for example by using an own **namespace** for each page. Translation files for these different pages can then be cared for independently and will only be loaded if they are actually needed.

We always have to use at least **one** **namespace** in **i18next**. By default, it will be `translation`, but it can be changed in the `defaultNS` option in the configuration object of the `.init()` method. The **namespace** itself ****is also an object which contains the translations in the form of `translationKey: value` or `greeting: 'Hello world!'` to be concrete.

The value _could_ also be an object:

```javascript
{
  greeting: {
    morning: 'Good morning!',
    evening: 'Good evening!',
  }
}
```

Or for short:

```javascript
{
  'greeting.morning': 'Good morning!',
  'greeting.evening': 'Good evening!',
}
```

It is entirely up to you which form you like best.

### Using translations in React components

Once **i18next** is set up correctly and the translations have been set up as well, we can start to translate our components. **i18next** offers full flexibility: we can work with a `withTranslation` HOC in **class components** or a `useTranslation` hook in **function components**. In the rare case of having to use components inside of translations, `react-i18next` boasts with a so-called `Trans` component.

#### Using the withTranslation\(\) HOC in class components

The `withTranslation()` function will create a **HOC** which we can pass to the component that is supposed to be translated. The `t` and `i18n` props will be passed to this component. To do this, we import the function as a named export from the `react-i18next` package:

```javascript
import { withTranslation } from 'react-i18next';
```

Once imported, we call the function and obtain an HOC which we pass the component to in which we want to access the translated values. If we are using **namespaces**, we can also pass the **namespace** we want to use  to the `withTranslation()` function:

```javascript
// Without namespaces (using the default value):
const TranslatedComponent = withTranslation()(TranslatedComponent);

// Using a single namespace:
const TranslatedComponent = withTranslation('namespace')(Component);

// Using multiple namespaces:
const TranslatedComponent = withTranslation([
  'namespace1', 
  'namespace2'
])(TranslatedComponent);
```

For myself it has proven useful, to extract components into their own files and wrap them with the `withTranslation()` HOC when they are exported:

```jsx
// Greeting.js
class Greeting extends React.Component {
  render() {
    const { t } = this.props;
    return (
      <h1>{t('greeting')}</h1>
    );
  }
}

export default withTranslation()(Greeting);
```

We can then immediately import the translated component:

```javascript
import Greeting from './Greeting.js';
```

Not only have we imported the actual component, but we have also gained access the extended props `t`, `i18n` and `tReady` via i18next.

The `t` function forms the central function for everything related to translations. It is passed a **translation** **key** and will return the translated value to us, based on the chosen language.

```jsx
<h1>{t('greeting')</h1> 
// -> <h1>Hello world!</h1>
```

If plurals or placeholders are used in the translations, they can be defined in the second parameter:

```javascript
<p>{t('messageCount', { count: 3 })}</p>
// -> <p>3 new messages</p>
```

The `i18n` prop contains the initialized **i18next** instance. It offers a number of properties and methods that can be relevant for our translations. The most notable are:

* `i18n.language` to read the currently selected language
* `i18n.changeLanguage()` to switch the currently selected language

In order to switch the language from `en` to `de`, one can call `i18n.changeLanguage('de')`.

#### Using the useTranslation\(\) hook in function components

The use of the `withTranslation()` HOC is not constrained to **class components** but can also be used in **function components**. However, the use of the `useTranslation()` hook often simplifies the component and makes it much more readable. The hook can be imported similarly to the HOC:

```javascript
import { useTranslation } from 'react-i18next';
```

This **hook** allows us to extract the `t` and `i18n` properties by using **destructuring assignment** from ES2015+.

```jsx
const Greeting = () => {
  const { t, i18n } = useTranslation();
  return (
    <div>
      <h1>{t('greeting')}</h1>
      <button onClick={() => i18n.changeLanguage('de')}>de</button>
      <button onClick={() => i18n.changeLanguage('en')}>en</button>
    </div>
  );
};
```

As was already the case in the `withTranslation()` HOC, `t` refers to the function which allows us to display translations based on their **translation key**. `i18n` refers to the respective **i18next** instance. The `useTranslation()` hook offers the same set of functionality as the `withTranslation()` HOC, however it is much more explicit and thus more readable. In order to use different **namespaces**, we can pass a string or an array of strings containing the namespaces to the hook:

```javascript
const { t } = useTranslation('namespace');
const { t } = useTranslation(['namespace1', 'namespace2']);
```

If no namespace has been provided, the default settings will be set.

#### Complex translations using the Trans component

In a few select cases, it might be necessary to use React component in the translations. For example, one might want to use the `Link` component from **React Router** to link to a different URL within a translation. The `t()` function does not support this out of the box. The solution comes in the form of the `Trans` component from `react-18next`. It is not always easy to understand how it should be used but it can be powerful tool.

Let's assume that we want to use a `Link` component inside of our translations. The code might resemble something like this:

```jsx
<p>
  <label>
    <input type="checkbox" name="terms" />
    Ich akzeptiere die <Link to="/terms">AGB</Link>
  </label>
</p>
```

Using the Link component in such a translation would not work as translations are strings and would thus not be able to define that the `<Link>` component does in fact refer to the Link component from the React Router package.

The `Trans` component offers the solution to our problem. Translations using this component can include numbered placeholders. These placeholders will later be replaced with components which will appear in the same place as they are used as in the `Trans` component.

Let's look at the above example using the `Trans` component:

```jsx
// Translations:
const de = {
  terms: 'Ich akzeptiere die <1>AGB</1>.',
};

const en = {
  terms: 'I accept the <1>Terms and Conditions</1>.',
};

// JSX:
<p>
  <label>
    <input type="checkbox" />
    <Trans i18nKey="terms">
      I accept the <Link to="/terms">Terms and Conditions</Link>.
    </Trans>
  </label>
</p>
```

Our text is wrapped by a `Trans` element. The text itself is only a placeholder that is used if the `i18nKey` prop does not contain a value which corresponds to a translation. Let's look at the numbering now. Where the text is placed and which text in particular is decided by the index value of the child element, similarly to `React.createElement()`.

Using the above example, we have the following counting result:

```text
0: I accept the
1: <Link to="/terms">Terms and Conditions</Link>
2: .
```

The `<1>Terms and Conditions</1>` will be replaced by the `<Link>` element as it corresponds to the index value of 1 in the array of children.

In complex structures where we might even deal with a number of links, this procedure can become a little bit cumbersome. Luckily, `react-i18next` offers the opportunity to automatically generate placeholders. These can be set as an option when **i18next** is initialized.

The first option to change is `saveMissing` which should now be set to `true`. Additionally, a `missingKeyHandler` function should be set. In the example to follow, a simple `console.log()` is used to log missing translations to the browser console:

```javascript
i18next
  .use(initReactI18next)
  .init({
    saveMissing: true,
    missingKeyHandler: (language, namespace, key, fallbackValue) => {
      console.log('Missing translation:', fallbackValue);
    },
    // other options
  });
```

If an `i18nKey` not currently in existence is used, the fallback value, meaning the value in the Trans element, will be written to the console instead:

{% hint style="info" %}
Missing translation: I accept the &lt;1&gt;Terms and Conditions&lt;/1&gt;.
{% endhint %}

We can now set up a translation for this case. **React Router's** `Link` has already been replaced by the corresponding placeholder \(**1**\) in the output. 

As a second option, we could also replace the `missingKeyJandler` function with a `debug` option set to `true`.

```javascript
i18next
  .use(initReactI18next)
  .init({
    debug: true,
    // other options
  });
```

We now get detailed debugging information, including a hint concerning our missing translations. The output would resemble the following:

{% hint style="info" %}
i18next::translator: missingKey de translation terms I accept the &lt;1&gt;Terms and Conditions&lt;/1&gt;.
{% endhint %}

The output generated by the `debug` option is a little more extensive than the one provided by the `missingKeyHandler` variant. However, it is possible to set this option by only setting its value to `true`.

By the way: You can still use the `t()` function as you are used to if you find yourself in a `<Trans>` element. This means, the example to follow is completely valid:

```jsx
<Trans i18nKey="terms">
  I accept the <Link to="/terms" title={t('termsTitle')}>Terms and Conditions</Link>.
</Trans>
```

### Summary

Using the examples provided in this chapter, it is relatively simple to provide internalization in our applications. For my personal and professional use, i18next has proven itself to be a very universal and complete tool to build international applications. Integrating with different libraries and frameworks has been a breeze. It offers all sorts of functionality that should be supported by a i18n framework and is easy to understand a learn with only a small set of functions: `i18n.changeLanguage()`, `t()`, `<Trans>`.

Once set up correctly \(I would advise you to check out the  [complete options of i18next](https://www.i18next.com/overview/configuration-options)\), internalization in React applications is easy to achieve without having to deal with a great deal of effort. One could even integrate with online services such as [Locize.com](https://locize.com/) or [Lokalise.co](https://www.lokalise.co) \(to name a few\) which help to create and manage translations or even outsource them automatically.

