# Tools and Frameworks

As React has been around for a while, an ecosystem has grown around it that boasts an abundance of tools. Ranging from static site generators (in order to realize simple to medium-sized and complex static website on the basis of React) to prototyping tools to tools which allow us to display our components in some sort of styleguide, the React ecosystem has much to offer. In this small sub-chapter I want to provide a _short_ overview of the most commonly used tools and frameworks.

### Storybook

> Storybook is a development environment for UI components. It allows you to browse through a component library, view the different states of each component, and interactively develop and test components.

**Storybook** is a tool that allows us to create isolated UI components for React, Vue.js and Angular. **Storybook** bundles our components in some form of Sandbox environment in which components can be independently developed in so-called **stories** and can then be displayed and viewed in a tidy and easy-to-use interface. By isolating these components, we allow for a great degree of abstraction and can also easily test and display for edge cases.

**License:** MIT \(Open Source\)  
**URL:** [https://storybook.js.org/](https://storybook.js.org/)  
**GitHub:** [https://github.com/storybooks/storybook](https://github.com/storybooks/storybook)

### React Styleguidist

> React Styleguidist is a component development environment with a hot reloaded dev server and a living style guide that you can share with your team.

**Styleguidist** is similar to Storybook. It also enables us to create a styleguide from our React components which are currently used in an application. However, **Styleguidist** is more implicit than **Storybook** and can be extended with **PropTypes** and **JSDoc** comments. Markdown files in the directory of the components can also be used to further add information on the component.

 **License:** MIT \(Open Source\)  
**URL:** [https://react-styleguidist.js.org/](https://react-styleguidist.js.org/)  
**GitHub:** [https://github.com/styleguidist/react-styleguidist](https://github.com/styleguidist/react-styleguidist)

### Docz

> It has never been so easy to document your things!

**Docz** is a tool that is centered around documentation as the name might suggest. It can also be seen as some form of styleguide, but it is completely **MDX** based. **MDX** is an extended version of the Markdown format which can also include React components. This way, our components can be described in `mdx` format and can be imported and used just like regular JavaScript files.

**License:** MIT \(Open Source\)  
**URL:** [https://docz.site/](https://docz.site/)  
**GitHub:** [https://github.com/pedronauck/docz](https://github.com/pedronauck/docz)

### React Cosmos

> Dev tool for creating reusable React components

**React Cosmos** takes things a little bit further and allows us to include external dependencies, such as React Router or Redux in our component overview, by using the concepts of _fixtures_ and _proxies_. This allows us to display and test such components more simply. React Cosmos knowingly breaks up the encapsulation of components which we found in the three previously mentioned tools and allows us to move beyond _just_ UI components.

**License:** MIT \(Open Source\)  
**GitHub:** [https://github.com/react-cosmos/react-cosmos](https://github.com/react-cosmos/react-cosmos)

### Gatsby

> Gatsby is a free and open source framework based on React that helps developers build blazing fast websites and apps

**Gatsby** is part of the so-called static site generator category, meaning it is a generator for static websites. **Gatsby** allows us to create components using React and GraphQL and then transforms them into static HTML files. **Gatsby** also creates a JavaScript bundle which is loaded as soon as your page loads. Once the bundle has loaded, **Gatsby** makes use of client-side rendering which means that sites are rendered to the user in "blazing fast" fashion as the HTTP overhead is drastically reduced. Gatsby has been started as an Open Source project and still maintains that status today. In May 2018 it received an impressive investment of 3.8 million US dollars. 

**License:** MIT \(Open Source\)  
**URL:** [https://www.gatsbyjs.org/](https://www.gatsbyjs.org/)  
**GitHub:** [https://github.com/gatsbyjs/gatsby](https://github.com/gatsbyjs/gatsby)

### React Static

> React-Static is a fast, lightweight, and powerful progressive static site generator based on React and its ecosystem.

**React-Static** is an alternative to **Gatsby** and is also a **static site generator**. Gatsby might be the more well-known tool, especially since its last investment. However, **React  static** is a serious contender and also boasts with a great and active community.

**License:** MIT \(Open Source\)  
**GitHub:** [https://github.com/nozzle/react-static](https://github.com/nozzle/react-static)

### Next.js

Next.js describes itself as "The React Framework" for a number of different use cases, ranging from static to dynamic websites, small to large companies, and mobile to classic websites. Next.js lives up to its claim, as 36,000 stars on GitHub indicate; it is extremely popular within the community. For many, Next.js is a serious alternative to Create React App. As opposed to CRA, Next.js supports server-side rendering out of the box, which is especially important for developers optimizing for SEO.

**License:** MIT \(Open Source\)  
**URL:** [https://nextjs.org/](https://nextjs.org/)  
**GitHub:** [https://github.com/zeit/next.js](https://github.com/zeit/next.js)

