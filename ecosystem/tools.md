# Tools and frameworks

A whole ecosystem of tools has now formed around React. From the Static Site Generator to create simple to medium-complex static websites based on React components, from prototyping tools to tools to clearly present the components of your own application in a kind of style guide, a lot is represented here. In this subchapter I would like to give a _short_ overview of the most popular tools and frameworks from the React world. 

### Storybook

> Storybook is a development environment for UI components. It allows you to browse a component library, view the different states of each component, and interactively develop and test components.

**Storybook** is a tool for creating isolated UI components for React, Vue.js and Angular. **Storybook** serves as a kind of sandbox environment in which components can be developed for themselves in so-called **Stories** and presents them in a clear and easy to use interface. The isolation enables a very high degree of abstraction, which also allows edge cases to be displayed and tested.

**License:** MIT \(Open Source\)  
**URL:** [https://storybook.js.org/](https://storybook.js.org/)  
**GitHub:** [https://github.com/storybooks/storybook](https://github.com/storybooks/storybook)

### React Styleguidist

> React Styleguidist is a component development environment with hot reloaded dev server and a living style guide that you can share with your team.

**Styleguidist is going in the same direction as Storybook. It is also possible with Styleguidist to create a styleguide from the UI components of your own application. However, **Styleguidist** is somewhat more implicit than **Storybook** here and so the styleguide is already enriched to a quite high degree from the **PropTypes** of components as well as from **JSDoc** comments. Markdown files in the folder of the respective component are then used for everything else.

**License:** MIT \(Open Source\)  
**URL:** [https://react-styleguidist.js.org/](https://react-styleguidist.js.org/)  
**GitHub:** [https://github.com/styleguidist/react-styleguidist](https://github.com/styleguidist/react-styleguidist)

### Docz

> It has never been so easy to document your things!

**Docz** is a tool which, as its name suggests, is used to document components. Also **Docz** is a kind of style guide, like the first two tools. Nevertheless, it stands out somewhere, because it is completely **MDX**-based. MDX is a version of the Markdown format extended by React components. Components are described in `.mdx` files and can be imported and used just like in JavaScript itself.

**License:** MIT \(Open Source\)  
**URL:** [https://docz.site/](https://docz.site/)  
**GitHub:** [https://github.com/pedronauck/docz](https://github.com/pedronauck/docz)

### React Cosmos

> Dev tool for creating reusable React components

A little further is React Cosmos. While the first three tools focus on encapsulation and the representation of pure UI components, **React Cosmos** deliberately breaks down this encapsulation and also allows external dependencies such as React Router or Redux to be represented and tested through the concept of _Fixtures_ and _Proxies_.

**License:** MIT \(Open Source\)  
**GitHub:** [https://github.com/react-cosmos/react-cosmos](https://github.com/react-cosmos/react-cosmos)

### Gatsby

> Gatsby is a free and open source framework based on React that helps developers build blazing fast websites and apps

**Gatsby** belongs to the category of static site generators, i.e. a generator for static websites. These are created in **Gatsby** from React components and GraphQL and consist of static HTML files. In the process **Gatsby** also creates a JavaScript bundle, which is loaded when your page is opened. Once the bundle is loaded, **Gatsby** makes use of client-side rendering, which has the consequence that the generated pages are displayed incredibly fast \("blazing fast"\), because from this point on the HTTP overhead is eliminated. **Gatsby** was launched as an open source project \(and still is today\) and as such received impressive funding of USD 3.8 million in May 2018.

**License:** MIT \(Open Source\)  
**URL:** [https://www.gatsbyjs.org/](https://www.gatsbyjs.org/)  
**GitHub:** [https://github.com/gatsbyjs/gatsby](https://github.com/gatsbyjs/gatsby)

### React static

> React-Static is a fast, lightweight, and powerful progressive static site generator based on React and its ecosystem.

**React static is an alternative toatsby. Also **React-Static** is a **Static Site Generator,** with which static websites based on React can be generated. Due to its high financing round, **Gatsby** is certainly the better known of the two tools, but **React-Static** is also a serious alternative in this area with a large and very active community.

**License:** MIT \(Open Source\)  
**GitHub:** [https://github.com/nozzle/react-static](https://github.com/nozzle/react-static)

### Next.js

Next.js confidently refers to itself as "The React Framework" for various applications such as static and dynamic websites, large and small businesses, mobile or classic websites and much more. In fact, with over 36,000 stars on GitHub in the React community, this framework is a recognized tool for developing applications with React and a real alternative to Create React App. Unlike CRA, Next.js comes with support for server-side rendered pages, which makes it especially interesting for SEO-relevant topics.

**License:** MIT \(Open Source\)  
**URL:** [https://nextjs.org/](https://nextjs.org/)  
**GitHub:** [https://github.com/zeit/next.js](https://github.com/zeit/next.js)

