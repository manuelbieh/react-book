# Multilingualism

One of the topics that comes up again and again besides **routing** and **state management** is **multilingualism**. How do I get my application translated so that a German user sees a German interface while other users see an English interface?

In this chapter I don't want to go into the depths of **Internationalization** \(mostly abbreviated: **i18n**\), because this is about **React**. I therefore assume a certain \(low\) basic understanding of the internationalization of user interfaces and concentrate on how to implement it in React.

How multilingualism can be implemented in a very simple form for simple apps using the **Context API** has already been demonstrated in the chapter about the **Context API**, where we have implemented a very rudimentary form of multilingualism using this API as an example. As the complexity of an application increases, so does the complexity of multilingualism. Suddenly things become important like the use of wildcards or the use of plural. At some point, it will make sense to fall back on packages that have been developed precisely for this purpose.

Here are some proven options in the **React ecosystem,** which we can fall back on. Among the better known are **Lingui**, **Polyglot**, **i18next** or **react-intl** and Facebook also has its own framework for the internationalization of React applications, **FBT**. This chapter is mainly about `i18next` and its react bindings `react-i18next`.

What's the matter with you? Well, I have worked in different projects with `react-intl` and `react-i18next`, evaluated the other alternatives in detail and am convinced that **i18next** is in many ways the best of the mentioned alternatives. There is a very large and active community around **i18next**, besides **React** many other frameworks, libraries and platforms are supported, it runs without much effort both on the client side in the browser and on the server side in Node.js and the translation format of **i18next** is offered by pretty much all large online translation services as an export format.

In addition, it was the first package to offer support for hooks a few days after their release, and was even optimized for use without sacrificing backward compatibility. The API remains mostly simple and easy to use and offers maximum flexibility. In other words, the whole package is very complete and leaves little to no wish unfulfilled.

## Setup of i18next

To install it, we call up our terminal again and type:

```bash
npm install i18next react-i18next
```

or with yarn:

```bash
yarn add i18next react-i18next
```

We install here with `i18next` the **Internationalization-Framework** itself, as well as with `react-i18next` its **React Bindings**, so to speak a set of components and functions, which make our work with i18next in React much easier. According to the principle we already got to know in the chapter on State Management with `redux` and `react-redux`.

Let's start by first creating two objects that contain our translations. One with texts in German, one with texts in English:

```javascript
const de = {
  greeting: 'Hello world!',
  headline: 'Today we learn internationalisation with React',
  messageCount: '{{count}} New message',
  messageCount_plural: '{{count}} new messages',
};

const en = {
  greeting: 'Hello world!',
  headline: 'Today we learn Internationalization with React',
  messageCount: '{count}} new message',
  messageCount_plural: '{count}} new messages',
};
```

To use **i18next** in our application now, we have to import it, initialize it and pass the React plugin. This should ideally happen at the beginning of our application. If possible, before we pass our app component to the `ReactDOM.render()` call.

We import the `i18next` package itself and the named export `initReactI18next` from the `react-i18next` package:

```javascript
import i18next from 'i18next';
import { initReactI18next } from 'react-i18next';
```

Then we use the `.use()` method to pass the React plugin to **i18next**, and the `.init()` method to initialize **i18next**:

```javascript
i18next
  .use(initReactI18next)
  .init({ ... });
```

The `init()` method expects a config object that should contain at least the two properties `lng` \(for the selected language\) and `resources` \(for the translations themselves\). Furthermore, it is often useful to create a `fallbackLng`, i.e. a fallback language that is used when a translation is not available in the selected language. Altogether **i18next** offers more than 30 different configuration options, but the three mentioned are the ones that interest us for the moment:

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
      {
        translation: de,
      }
    },
  });
```

So we first set the language to English, as well as the fallback language. Then follows a `resources` object, which needs some explanation. The object has the form:

```text
[Language].[Namespace].[Translation Key] = Translation
```

The language should be clear. This can be `de` for German, `en` for English or `de-AT` for German with Austrian regionalization. The property has as value an object consisting of at least one up to theoretically unlimited **Namespaces**.

The **Namespace** is a central feature in **i18next,** that can be used, but does not have to be. It allows the developer to split larger translation files into multiple parts that can be dynamically reloaded as needed. While this feature is not necessarily useful for smaller applications, it can be used in larger and more complex applications to keep translations small and clear, for example by giving each larger page area its own **namespace**. Translation files for this page area can then be maintained separately and loaded only when they are needed.

In **i18next** you must always use at least _one_ **Namespace**. By default this is called `translation`, but this can be changed by the `defaultNS` option in the configuration object of the `.init()` method. The **Namespace** itself is an object which contains the actual translations in the form `translationKey: value`, i.e. `greeting: 'Hallo Welt!'`.

The value itself _can_ again be an object:

```javascript
{
  greeting: {
    morning: 'Good morning!',
    evening: 'Good evening!',
  }
}
```

Or in abbreviated form:

```javascript
{
  Greeting.morning. Good morning,
  Greeting.evening. Good evening,
}
```

However, this is also at the discretion of the developer.

## Using translations in React components

Once **i18next** has been set up correctly and the translations have been created, we can continue translating our components. Again, **i18next** gives us full flexibility: We get a `withTranslation` HOC for use with **class components**, a `useTranslation` hook for use in **Function Components**, and for cases where we want to use components within translations, `react-i18next` also gives us a corresponding `Trans` component.

### Use with class components and withTranslation\(\)-HOC

We go through them one by one and start with the `withTranslation()` function. This creates a **HOC** to which we pass the component to be translated and get the props `t` and `i18n` into it. First we import the function as a named export from the `react-i18next` package:

```javascript
import { withTranslation } from 'react-i18next';
```

Then we call the function, get an HOC and pass the component in which we want to access the translated values to it. If we use **Namespaces,** we can also pass the desired **Namespaces** to the `withTranslation()` function:

```javascript
// Without namespaces (here the default value is used):
const TranslatedComponent = withTranslation()(TranslatedComponent);

// With only one namespace:
const TranslatedComponent = withTranslation('namespace')(Component);

// With several namespaces:
const TranslatedComponent = withTranslation([
  namespace1, 
  namespace2
)(TranslatedComponent);
```

In practice, it has proven to be a good idea for me to outsource individual components into my own files and to place the `withTranslation()`-HOC around the component during export:

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

This allows us to import the translated component immediately during the import:

```javascript
import Greeting from './Greeting.js';
```

Here we import the component with the props `t`, `i18n` and `tReady` extended by i18next.

The `t` function is the central function for everything that has to do with translations. She gets a **Translation Key** and returns the translated value based on our current language:

```jsx
<h1>{t('greeting')</h1> 
// -> <h1>Hello world!</h1>
```

If we use placeholders or plurals in our translations, these can be passed as a second parameter:

```javascript
<p>{t('messageCount', { count: 3 })}</p>
// -> <p>3 new messages</p>
```

The `i18n` prop contains the previously initialized **i18next** instance. It provides us with some features and methods that may be relevant to our translations. The two most important of these are certainly:

* `i18n.language` to read the currently selected language
* `i18n.changeLanguage()` to change the selected language

For example, to change the language of a component from `en` to `de`, the call to `i18n.changeLanguage('de')` can be used here.

### Use with Function Components and useTranslation\(\)-Hook

Of course, not only **class components** can be used with the `withTranslation()`-HOC, but also **Function Components**. However, we can also use the `useTranslation()` hook in the latter, which usually makes the component a bit more manageable. Therefore we import the hook as well as the HOC from the `react-i18next` package:

```javascript
import { useTranslation } from 'react-i18next';
```

The **Hook** then lets us extract the two properties `t` and `i18n` from ES2015+ via **Destructuring Assignment Syntax**:

```jsx
const Greeting = () => {
  const { t, i18n } = useTranslation();
  return (
    <div>
      <h1>{t('greeting')}</h1>
      <button onClick={() => i18n.changeLanguage('de')}>en</button>
      <button onClick={() => i18n.changeLanguage('en')}>en</button>
    </div>
  );
};
```

As in the `withTranslation()`-HOC, `t` is the function to output translations via your **Translation Key** and `i18n` is the **i18next** instance. The hook provides exactly the same functionality as the HOC, but in **Function Components** it is much clearer because it is more explicit. To use certain **Namespaces**, you can pass a string or an array of strings to the hook like you did with the `withTranslation()` function:

```javascript
const { t } = useTranslation('namespace');
const { t } = useTranslation(['namespace1', 'namespace2']);
```

If no namespace is passed, the default setting also applies here.

## Complex translations with the Trans component

In some cases it may be necessary to use React components in translations. This is the case, for example, if you want to use the `Link` component of the **React Router** to link to another URL within a translation. This is only not possible by using the `t()` function. For this purpose `react-i18next` provides us with the `Trans` component. Its use is not always easy to understand, but it is a very powerful tool.

So let's say we want to use the `Link` component within our translation. The code looks like this before translation:

```jsx
<p>
  <label>
    <input type="checkbox" name="terms" />
    I accept the <Link to="/terms">AGB</Link>
  </label>
</p>
```

Using the link component in a translation would not work here because translations are strings and we would have no way of specifying that `<Link>` is the link component from the React router package.

The solution here is to use the `Trans` component. Translations used with this component may have numbered placeholders. Instead, components are used that are used at the same position in the `Trans` component.

Let's have a look at the example above using `Trans`:

```jsx
// Translations:
const de = {
  terms: 'I accept the <1>AGB</1>.',
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

Here we enclose our text with a `Trans` element. The text serves only as a placeholder, which is used if there is no corresponding translation for the value specified in the `i18nKey` prop. Now it's time to count. The index value of the child elements determines where which text is replaced and how, and where a component occurs. Similarly, how `React.createElement()` works.

In the above example, the following counting result is obtained:

```text
0: I accept the
1: <Link to="/terms">Terms and Conditions</Link>
2: .
```

So `<1>Terms and Conditions</1>` is replaced by the `<Link>` element, since it has the index value 1 in the children array.

Now, especially in more complex structures, where there may even be several links, it can be quite cumbersome to count that. Here `react-i18next` offers two possibilities to generate the placeholders automatically. Both are set as options when initializing **i18next**.

The first option is to set `saveMissing` to `true` and use a `missingKeyHandler` function. In the following example we use a simple `console.log()` to log missing translations into the browser console:

```javascript
i18next
  .use(initReactI18next)
  .init({
    saveMissing: true,
    missingKeyHandler: (language, namespace, key, fallbackValue) => {
      console.log('Missing translation:', fallbackValue);
    },
    // further options
  });
```

If a `i18nKey` is used that does not exist, the fallback value, i.e. the value from the trans element, is written to the browser console instead:

{% hint style="info" %}
Missing translation: I accept the &lt;1&gt;Terms and Conditions&lt;/1&gt;.
{% endhint %}

A corresponding translation can now be created for these. The **React Router** `Link` has already been replaced in the output by the appropriate placeholder index, in this case **1**.

As a second option we can set the `debug` option to `true` instead of the `missingKeyHandler`.

```javascript
i18next
  .use(initReactI18next)
  .init({
    debug: true,
    // further options
  });
```

We will then receive a set of debugging information, as well as an indication of missing translations. The output is then approximately:

{% hint style="info" %}
i18next::translator: missingKey de translation terms I accept the &lt;1&gt;Terms and Conditions&lt;/ i&gt;1&gt;.
{% endhint %}

The output here is a bit more insignificant than with the own `missingKeyHandler` variant, but here it is sufficient to set only one option to `true`.

Within `<Trans>` elements, the `t` function can also be used as usual. For example, the following would also be possible:

```jsx
<Trans i18nKey="terms">
  I accept the <Link to="/terms" title={t('termsTitle')}>Terms and Conditions</Link>.
</Trans>
```

## Conclusion

With the examples shown in this chapter, it is relatively easy to develop your application in multiple languages. In practice, i18next has proven to be a very universal and complete tool for me personally. The integration into different libraries and frameworks works flawlessly, it offers all functions required by such an i18n framework and with only a few functions and components \(`i18n.changeLanguage()`, `t`, `<Trans>`\) it is relatively easy to learn.

Once correctly set up and configured, here I recommend a look at the [full options of i18next](https://www.i18next.com/overview/configuration-options), realizing multilingualism in your React application without it feeling like extra work. Online services such as [Locize.com](https://locize.com/) or [Lokalise.co](https://www.lokalise.co), to name but a few examples, also make sure of this, with which the creation of translations can be managed and in some cases even outsourced automatically.

