# CSS and Styling

Styling in React is a topic of its own in React. React does not offer its own in-house solution to make styling easier, however the introduction of CSS-in-JS has shaken up the scene a little bit. Adopted widely and loved by some but hotly debated by others. With CSS-in-JS, the styling of components also moves into JavaScript to not break with the paradigm of component-based development. But let's start with the basics and explore the topic bit by bit.

### Styling with the style attribute

The simplest way to style components in React is using the `style` attribute on regular HTML elements. It differs from regular HTML though and React components expect an **object** in the form of `property: value`. The property itself needs to be declared in JavaScript \(not its regular CSS counterpart\) form meaning `zIndex` instead of `z-index`,  `backgroundColor` instead of `background-color` or `marginTop` instead of `margin-top`. If the values accept declarations in pixels, it is optional to explicitly define `px` as the corresponding unit:



```jsx
<div style={{border: '1px solid #ccc', marginBottom: 10}}>
  A div with a grey border and a 10 pixel margin at the bottom
</div>
```

Values that are not used with a unit \(for example `z-index`, `flex` or `fontWeight`\) are not affected by this and can be used just like we are used to. React does not add `px` to these.

By using an object instead of a string, React keeps a consistent approach to dealing with styles. The regular `style` property of DOM elements `document.getElementById('root').style` is also an object and also ensures that we secure these by preventing XSS.

While using inline styling is not exactly recommended, it can be useful at times if the styling of an element depends on particular values in state.

### Using CSS classes in JSX

It is much cleaner and nicer to use real CSS classes in JSX, just like we do in regular HTML with the difference being that we declare classes by using `className` instead of `class`:

```jsx
<div className="item">...</div>
```

React renders normal syntax with our HTML equivalent:

```markup
<div class="item">...</div>
```

It is also quite common to concatenate values in the `className` prop dynamically:

```jsx
render() {
  let className = 'item';
  if (this.state.selectedItem === this.props.itemId) {
    className += ' item--selected';
  }
  return (
    <div className={className}>
      ...
    </div>
  );
}
```

In this example the value for `className` is `item` in each case. If the selected item is the current item, it also gets the class `item item--selected`.

The package `classnames` has become the de-facto standard to let classes be defined based on a condition. It can be installed via the CLI:

```bash
npm install classnames
```

```bash
yarn add classnames
```

Now, we only need to import the package in those components where we want to use it:

```javascript
import classNames from 'classnames';
```

This way, we are importing the function and give it the name of `classNames.` The function expects an arbitrary amount of parameters which can be a string or an object in the form of `{ class: true | false }`. It works similar to what you might already expect: if the value in the condition is `true`, `classNames` employs a class with the same property name:

```jsx
render() {
  return (
    <div className={classNames('item', {
      'item--selected': this.state.selectedId === this.props.itemId
    })}>
      ...
    </div>
  );
}
```

Objects can also be used to work with multiple properties. Multiple classes can be set if the condition is `true`:

```jsx
render() {
  return (
    <div className={classNames({
      'item': true,
      'item--selected': this.state.selectedId === this.props.itemId
    })}>
      ...
    </div>
  );
}
```

If regular CSS classes are used, we should also ensure that the corresponding stylesheet is also linked in the HTML document. React does not usually take care of this on its own.

### Modular CSS with CSS Modules

**CSS Modules** are some sort of predecessor to **CSS-in-JS** and combine a number of properties from CSS and JavaScript modules. As the name already suggests, the CSS is found in their own importable **modules**, which however contain pure CSS and are tied to one particular component. If we were to develop a component to display a profile picture in a file called `ProfileImage.js`, the **CSS Modules** approach often introduces another file with the name `ProfileImage.module.css`.When these CSS modules are imported, a cryptic classname is generated to ensure that the classname is only given one and is only used in the single component. This aims to prevent that classnames are accidentally used by two different components.

**CSS Modules** are only a concept first of all, and do not dictate a particular approach or implementation. The implementation is achieved by tools such as `css-loader` for Webpack.

It's not actually necessary to enforce the file ending of `.module.css` \(just regular `.css` would also be okay\) but it has become convention in many well-known tools. **Create React App** also supports this convention out of the box without the need for further configuration by using the previously mentioned `css-loader` in the background \(implemented in Webpack\).

 In these CSS files, regular and **standardized CSS** can be found. It can then be imported into a JavaScript module with ease:

```javascript
import css from './ProfileImage.module.css';
```

We've just entered new territory: the variable `css` now containts an object with properties that relate to the classnames in our CSS modules. If there is a CSS class in the CSS file that has the name `image`, we can now access the property `image` on the CSS object. The resulting value in the rendered markup is a unique string such as `ProfileImage_image_2cvf73`.

We can pass these generated classnames to our components like this:

```jsx
<img src={props.imageUrl} className={css.image} />
```

This code would result in the following rendered markup:

```markup
<img src="..." className="ProfileImage_image_2cvf73" />
```

If we used the class `image` in another component and also imported the css file with an `image` class, we'd not run into conflict as we'd usually do in regular CSS. The _generated_ classname would be different.

Everything that's allowed in regular CSS is also allowed in CSS Modules. The Cascade will remain intact:

```css
.imageWrapper img {
  border: 1px solid red;
}

.imageWrapper:hover img {
  border-color: green;
}
```

This CSS snippet would be used in the following JSX structure:

```jsx
const ProfileImage = () => {
  return (
    <div className={css.imageWrapper}>
      <img src="..." />
    </div>
  );
};
```

**CSS Modules** aim to avoid conflicts in very isolated components while still allowing us to write regular CSS in CSS files without any JavaScript knowledge. They are a great solution for teams in which development is more separated into JavaScript and CSS and in which there are experts for both.

### CSS-in-JS - Moving styles into JavaScript

I've already mentioned in the introduction that **CSS-in-JS** is a bit of a hotly debated topic. Opponents say that users of **CSS-in-JS** do simply not understand the cascade to write scalable CSS. Proponents on the other hand explain that the cascade is not the main reason for choosing CSS-in-JS but argue that a safer way is needed to write highly isolated components. I'm a bit more diplomatic myself  and think that there's room and reasons for both. **CSS-in-JS** certainly has reason to exist. But what even is **CSS-in-JS**?

The **CSS-in-JS** approach mandates that the styles which are commonly found in CSS files are now moved into JavaScript. As was already the case in **CSS modules**, the primary goal is to create highly isolated components which are free of conflict making them easy to reuse throughout the application. However, as opposed to **CSS modules**, the styling in **CSS-in-JS** happens entirely in JavaScript. The syntax is very similar and it mostly feels like you're writing regular CSS.

**CSS-in-JS** is the umbrella term for the concept as such whereas the actual implementation is achieved by a number of different libaries. If you are looking for a great overview, I suggest you checkout out [http://michelebertoli.github.io/css-in-js/](https://michelebertoli.github.io/css-in-js/) - this site covers over 60 different libraries that can help you to implement your own **CSS-in-JS** solution.

In the remaining parts of the chapter, I want to focus on **styled components** which is the most popular library with around 23,000 starts on GitHub. In order to use it, we need to first install it via the command line:

```bash
npm install styled-components
```

```bash
yarn add styled-components
```

Once installed, we are ready to import it and write our first **styled component**. Using the `styled` function, we define the HTML element that we want to style. **Template literal** syntax from ES2015 is used to define the CSS for our **styled component.**

In order to create a button which is black and yellow, we would need to write the following:

```jsx
import React from 'react';
import ReactDOM from 'react-dom';
import styled from 'styled-components';

const Button = styled.button`
  background: yellow;
  border: 2px solid black;
  color: black;
  padding: 8px;
`;

const App = () => {
  return (
    <div>
      <h1>An example of a styled component</h1>
      <Button>A black and yellow button without any function</Button>
    </div>
  );
};

ReactDOM.render(<App />, document.getElementById('root'));
```



