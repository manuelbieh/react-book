# Code Splitting

When developing projects with React, most people tend to also use a **bundler** such as **Webpack**, **Browserify** or **Rollup**. These tools ensure that all files and all imports are later bundled into a single big file which can be deployed in a relatively simple fashion without having to worry about relative links. This process itself is called **Bundling**. A **Bundle** can easily grow and reach a size of a megabyte or more especially if many third party libraries are used. Large Bundles are a big performance problem as bigger bundles take longer to be processed and downloaded by the browser as well as executing.

To combat large bundles, a technique called **Code Splitting** is used to counteract it. **Code Splitting** defines the process in which we separate our application into many smaller bundles which are all able to run on their own and load further bundles if necessary. A common separation is either splitting by dependencies \(React, ReactDOM, ...\) or having a bundle per route.

One of the simplest ways to make use of code splitting is to use **Dynamic Import Syntax**. It's currently in discussion at **TC39** and thus in the process of being standardized. But **Babel** and **Webpack** enable us to make use of Code Splitting today. It's necessary to install the babel plugin `@babel/plugin-syntax-dynamic-import` to make use of code splitting. **Create React App** as well as **next.js** and **Gatsby** support **Code Splitting** out of the box and do not need to be configured to allow it.

### Using dynamic imports

We have briefly touched on import syntax in the chapter on ES2015+. Dynamic Import Syntax is an extension of this syntax and allows us to dynamically lazy load them. Dynamic imports are similar to a promise:

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

When Webpack finds a dynamic import, it will automatically perform code splitting and put this file into its own so-called _Chunk_. These Chunks are loaded independently once they are needed within the application - thus the naming of **Lazy Loading.**

### Lazy Loading of components with React.lazy\(\)

Let's talk about **lazy loading in React**. To make the experience of performing **lazy loading** more enjoyable, React offers its own method from version **16.6** onward to dynamically lazy load components. It is combined with **Dynamic Import Syntax** and allows the developer to easily load certain components only when the application has started running thus further reducing the size of the bundle.

Even though a component might have been loaded via `React.lazy()`, it can be used in React just as a regular component. It can also receive props as well as refs, contain further elements or be self-contained. The `React.lazy()` method expects a function as its first parameter which will return a dynamic import. This import has to import a component which has been exported using default exports before.

```jsx
// LazyLoaded.js
import React from 'react';

const LazyLoaded = () => (
  <p>This component is only loaded by the server once it is in use.</p>
);
```

```jsx
// app.js
import React, { Suspense } from 'react';
import ReactDOM from 'react-dom';

const LazyLoaded = React.lazy(() => import('./LazyLoaded.js'));

const App = () => (
  <Suspense fallback={<div>Application is loading</div>}>
    <LazyLoaded />
  </Suspense>
);

ReactDOM.render(<App />, document.getElementById("root"));
```

This method allows us to easily optimize for the size of our JavaScript bundle and only load certain files from the server when they are actually requested by the user. During the time it takes loading and receiving the data from the server to being executed, we will see information informing us that `<div>Application is loading</div>`. This is only possible because we are using a feature which has also been added to React in version **16.6**: `React.Suspense.` 

### Display fallbacks with React.Suspense

Back in the day, the Suspense component on the React object was named **Placeholder**. This is a very accurate description of the task it fulfills: acting as a placeholder for components which have not yet been rendered and displaying alternative content in the meantime. These fallbacks can take many forms: they can be a message that parts of the application are still being loaded or take the form of a loading animation. The placeholder to display is passed to **Suspense** in the `fallback` prop and _has_ to be defined. Any valid React Element can be used and passed as a prop. Strings such as `<Suspense fallback="Loading ...">[…]</Suspense>` are also a valid.

As long as the component which you want to lazy load has not fully loaded, all children of the Suspense element will be replaced with the indicated fallback. No limit concerning the number of React.lazy\(\) component imports have been enforced. The fallback placeholder will be shown until all components have loaded and can be displayed.

Nesting components is also possible and can be great idea in certain scenarios. When there are parts of the site which are slightly less important and might interfere with the rendering of the primary user interface, it is recommended to wrap these parts of the application / the component tree in their own `Suspense` element. This will boost performance and drive the important parts of the application to load first.

A possible scenario to use `Suspense` in practice is image editing. In these type of cases, it can be useful to display the image to edit to the user already to give visual clues. The rest of the user interface containing the actual editing functionality will be loaded in a further step if loading the actual component is taking longer.

```jsx
import React, { Suspense } from "react";
import ReactDOM from "react-dom";

const ImageCanvas = React.lazy(() => import("./ImageCanvas"));
const ImageToolbar = React.lazy(() => import("./ImageToobar"));

function App() {
  return (
    <Suspense fallback={<div>Application loading</div>}>
      <ImageCanvas url="https://via.placeholder.com/350x240"/>
      <Suspense fallback={<div>Image editing tools are being loaded</div>}>
        <ImageToolbar />
      </Suspense>
    </Suspense>
  );
}

ReactDOM.render(<App />, document.getElementById("root"));
```



