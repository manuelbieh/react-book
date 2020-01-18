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

