# Code Splitting

If you develop a project with React, in most cases you also use a **Bundler** like **Webpack,** **Browserify** or **Rollup**. These ensure that all individual files, all imports, are later bundled into a single large file, which can then be deployed relatively easily without a developer having to worry too much about relative links. This process is accordingly called **bundling**. In very large and complex projects such a **Bundle** can quickly grow one megabyte in size or even larger, especially if many third-party libraries are in use. This is a problem in many ways, because large bundles take longer to download from the browser, and running unnecessarily large bundles inevitably results in performance degradation.

To counter the problem of large bundles, there is the so-called **Code Splitting**. Code splitting splits the application into several smaller bundles, all of which run on their own and load more bundles if they are needed later. The division into a bundle with the most frequently used dependencies \(e.g. React, React DOM, ...\) and one bundle per route is a quite common method for **Code Splitting**.

The simplest method is to use the **Dynamic Import Syntax**. This is currently a proposal for the **TC39**, so it is currently in the standardization process. But thanks to Webpack and Babel, it is still possible to use it today. You need the Babel plugin `@babel/plugin-syntax-dynamic-import`. **Create React App** and other tools such as **next.js** or **Gatsby** come with support for dynamic imports and do not need to be configured specifically to use **Code Splitting**.

## Using dynamic imports

The import syntax has already been mentioned in the chapter on ES2015+. The **Dynamic Import Syntax** is an extension of the **Dynamic Import Syntax** and allows, therefore, the name to load imports dynamically within an application. A dynamic import does not work any differently than a Promise:

```jsx
// greeter.js
export sayHi = (name) => `Hi ${name}!`;
```

```jsx
// app.js
import('./greeter').then((greeter) => {
  console.log(greeter.sayHi('Manuel'); // "Hi Manuel!"
});
```

If Webpack finds a dynamic import, it automatically uses its code splitting function at this point and stores the corresponding file in its own so-called _Chunk_ when creating the bundle, i.e. a section, which it then loads independently as soon as it is needed in the application. This behaviour is generally referred to as **Lazy Loading**, i.e. "delayed loading".

## Delayed loading of components with React.lazy\(\)

And that brings us to the next topic: **Lazy Loading with React. In order to make the developer experience with** Lazy Loading **as pleasant as possible, React offers since version** 16.6. **an in-house method to dynamically reload components. This is combined with the** Dynamic Import Syntax\*\* and allows the developer to load certain React components at runtime to further reduce the size of the bundles.

An import loaded via `React.lazy()` can be used in React as an ordinary component. Props can be handed over to her as well as refs. It can contain its own child elements or be self-contained. The method expects a function as a parameter that returns a dynamic import. This import must import a component that has a default export, which in turn must be a React component:

```jsx
// LazyLoaded.js
import React from 'react';

const LazyLoaded = () => (
  <p>This component is not loaded from the server until it is used </p>
);
```

```jsx
// app.js
import React, { Suspense } from 'react';
import ReactDOM from 'react-dom';

const LazyLoaded = React.lazy(() => import('./LazyLoaded.js')));

const App = () => (
  <Suspense fallback={<div>Application is being loaded</div>}>
    <LazyLoaded />
  </Suspense>
);

ReactDOM.render(<App />, document.getElementById("root"));
```

In complex components and especially with growing applications you can optimize the size of the JavaScript bundle and load relevant files from the server only when they are really needed. While the file is being loaded from the server until it is finally executed, the component will display the message `<div>Application is being loaded</div>`. This is due to another innovation that found its way into React in version **16.6.**:

## Display of placeholders with React.Suspense

The `Suspense` component on the React object was originally called **Placeholder**  in its original version, and that pretty much describes what it does: it acts as a placeholder for components that haven't been loaded yet, and renders an alternative. As in the example above, this can be a message that parts of the application are loaded or a classic load animation. The placeholder is specified as `fallback` prop and must be defined. Any valid React element can be used as the valid value of the prop. This also includes strings. `<Suspense fallback="Will be loaded">[...]</Suspense>` would therefore also be a valid placeholder.

As long as the component to be loaded has not yet been fully loaded, all child elements of the `Suspense' element will be replaced by the specified placeholder and only replaced by the actual content after the component has been fully loaded. Any number of components loaded via`React.lazy\(\)`can be used within a`Suspense`element. The`fallback\` placeholder is then displayed until **all** components are fully loaded and can be displayed!

Nesting is also possible and sometimes even useful. For example, if there are parts in the page that are unimportant and should not delay rendering the user interface if other, more important parts are already loaded, the corresponding page tree can be enclosed by its own `Suspense` element. This results in the other, possibly more important parts of the interface being displayed as soon as they are loaded, while for the other, less important parts a placeholder is displayed again.

A conceivable scenario could be an application for image processing. Here it may be useful to display the image to be processed as soon as the component responsible for displaying the image has been loaded. The user interface with the actual functions for processing is then only displayed in the next step if loading the corresponding component takes longer. And only then.

```jsx
import React, { Suspense } from "react";
import ReactDOM from "react-dom";

const ImageCanvas = React.lazy(() => import("./ImageCanvas"));
const ImageToolbar = React.lazy(() => import("./ImageToobar")));

function App() {
  return (
    <Suspense fallback={<div>Application is being loaded</div>}>
      <ImageCanvas url="https://via.placeholder.com/350x240"/>
      <Suspense fallback={<div>Editing functions are loaded</div>}>
        <ImageToolbar />
      </Suspense>
    </Suspense>
  );
}

ReactDOM.render(<App />, document.getElementById("root"));
```

What happens here is that the `ImageCanvas` \(for displaying the image\) and the `ImageToolbar` \(for the editing functions\) are in a `Suspense` element. This displays the wildcard message: _"The application will load"_ until the `ImageCanvas` component is loaded from the server.

If this happens **before** loading the `ImageToolbar`, it will already be displayed and the second, inner `Suspense` element will be used. This ensures that the message _"Editing functions are loaded"_ is displayed until they have actually been loaded.

Once the `ImageCanvas` component is loaded, **after** the `ImageToolbar` component has been loaded, the inner `Suspense` element is resolved, but the outer Suspense element prevents the toolbar from displaying it and only displays it once the `ImageCanvas` component is loaded. Then, however, without further delay.

So our user interface knows three possible representations:

* `ImageCanvas` and `ImageToolbar` have been successfully loaded and are both displayed
* `ImageCanvas` has not been loaded yet and only the "Application is loading" message appears, regardless of the load status of the `ImageToolbar`.
* `ImageCanvas` was loaded, `ImageToolbar` not yet. Then the `ImageCanvas` would already be displayed, but instead of the toolbar there would be the note "Editing functions will be loaded" until they have actually been loaded.

So we deliberately exclude the possibility that a user already sees the editing functions for an image, but not the drawing area on which the image to be edited is displayed. A clever nesting of `Suspense` gives us all the flexibility we need to fine-tune when which parts of the application should already be displayed and where we want to temporarily display a placeholder.

Currently **Suspense** is only officially supported as a placeholder for loading components using `React.lazy()`. In the future, the asynchronous loading of data of various types  \(such as API queries\) will also be supported by **Suspense**.

{% hint style="warning" %}
**Caution:** Currently, **Lazy** and **Suspense** are supported only when used in **client side** applications. Support for **server-side** rendering is currently not available for this feature and is currently in the works.
{% endhint %}

