# Code Splitting

When developing projects with React, most people tend to also use a **bundler** such as **Webpack**, **Browserify** or **Rollup**. These tools ensure that all files and all imports are later bundled into a single big file which can be deployed in a relatively simple fashion without having to worry about relative links. This process itself is called **Bundling**. A **Bundle** can easily grow and reach a size of a megabyte or more especially if many third party libraries are used. Large Bundles are a big performance problem as bigger bundles take longer to be processed and downloaded by the browser as well as executing.

To combat large bundles, a technique called **Code Splitting** is used to counteract it. **Code Splitting** defines the process in which we separate our application into many smaller bundles which are all able to run on their own and load further bundles if necessary. A common separation is either splitting by dependencies \(React, ReactDOM, ...\) or having a bundle per route.

One of the simplest ways to make use of code splitting is to use **Dynamic Import Syntax**. It's currently in discussion at **TC39** and thus in the process of being standardized. But **Babel** and **Webpack** enable us to make use of Code Splitting today. It's necessary to install the babel plugin `@babel/plugin-syntax-dynamic-import` to make use of code splitting. **Create React App** as well as **next.js** and **Gatsby** support **Code Splitting** out of the box and do not need to be configured to allow it.

