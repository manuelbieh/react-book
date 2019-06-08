# CSS and styling

Styling in React is a relatively separate topic. React does not offer any home remedies to facilitate the styling of applications using CSS, but CSS-in-JS has created its own, sometimes very controversially discussed movement on the subject. The styling part is also implemented in JavaScript in order not to break with the paradigm of component-based development. But one by one, we start with the basics and work our way up to the topic bit by bit.

### Styling with style attribute

Probably the easiest way to style components in React is by using the `style` attribute on HTML elements. However, this works a bit differently than in HTML and so React expects a **object** in the form `Property: Value`, where the JavaScript notation of a CSS property is expected as the property. So `zIndex` instead of `z-index`, `backgroundColor` instead of `background-color` or `marginTop` instead of `margin-top`. For values that accept pixel specifications, the use of `px` as a unit is optional:

```jsx
<div style={border: '1px solid #ccc', marginBottom: 10}}>
  A div with a gray frame and 10 pixels spacing down
</div>
```

Values that are used without a unit \(e.g. `z-index`, `flex` or `fontWeight`\) are not affected, they can be used without hesitation without React adding `px` to them. 

By using an object instead of a string, React is consistent with the `style` property of DOM elements \(`document.getElementById('root').style` is also an object!\) and also ensures that no XSS vulnerabilities can occur.

Now, the use of inline styles is not necessarily desirable, but it can be helpful in some situations, such as when the styling of an element depends on certain variable values in the state.

### Using CSS classes in JSX

Much more pleasant and cleaner is the use of real CSS classes in JSX, as it is already known from the use of HTML. The crucial difference here is that unlike HTML, the name of the corresponding prop in JSX is not `class`, but `className`:

```jsx
<div className="item">...</div>
```

React finally renders the usual syntax with the respective HTML equivalent:

```markup
<div class="item">...</div>
```

As usual in JSX, the value for the `className` prop can also be assembled dynamically via string concatenation:

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

Here the value for `className` would in any case be `item` and in case the selected item corresponds to the current item `item item--selected`.

The `classnames` package has established itself as the de facto standard for such cases. With this it is possible to make classes dependent on a condition. It can be installed via the CLI, using :

```bash
npm install class names
```

```bash
yarn add classnames
```

Then we only have to import it into the components we want to use it in. This happens quite uncomplicated via:

```javascript
import classNames from 'classnames';
```

This will import the function and give it the name `classNames`. The function then expects any number of parameters which can either be a string or an object in the form `{ class: true|false }`. The functionality is also quite simple: If the value for a property is `true`, `classNames` sets a class with the same property name. In the above example, that would be:

```jsx
render() {
  return (
    <div className={classNames('item', {
      &lt;font color="#ffff00"&gt;-==- proudly presents
    })}>
      ...
    </div>
  );
}
```

It is also possible to use an object with several properties, here all properties whose value is `true` are set as a class:

```jsx
render() {
  return (
    <div className={classNames({
      item': true,
      &lt;font color="#ffff00"&gt;-==- proudly presents
    })}>
      ...
    </div>
  );
}
```

When using classes, it should of course be ensured that the stylesheet with the respective classes is also included in the HTML document, since React does not care about this in principle.

### Modular CSS with CSS Modules

**CSS Modules** are a kind of precursor to **CSS-in-JS** and combine some features of CSS and JavaScript modules. As the name suggests, the CSS is located here in its own importable **modules**, which consist of pure CSS and usually belong directly to a component. For example, if we develop a component to display a profile image in a file called `ProfileImage.js`, the **CSS Modules** approach often has a file called `ProfileImage.module.css` as well. When importing such a CSS module, it is then ensured that a unique, often somewhat cryptic class name is generated to ensure that this class is only used in the respective component. This prevents conflicts with other components that use the same class names.

However, **CSS Modules** themselves are initially only a **concept** and not yet a concrete implementation. Tools like `css-loader` for Webpack take over the implementation.

The extension `.module.css` is by no means mandatory and a mere `.css` would also be possible, but `.module.css` has become a convention for some well-known tools with a wide range and is also supported by **Create React App** without the need to make any further configuration settings. CRA uses the `css-loader` implementation for Webpack mentioned above.

The CSS files contain regular, **standard-compliant CSS** in its usual spelling. This can then be imported into a component like a JavaScript module:

```javascript
import css from './ProfileImage.module.css';
```

This is where the special feature of **CSS Modules** comes into play: the variable `css` now contains an object with properties that correspond to the class names from our CSS module. If a CSS class named `image` exists in the CSS file, we now have a property named `image` in the `css` object as well. The value is a unique, generated string such as `ProfileImage_image_2cvf73`. 

We can now assign this generated class name as `className` in our component:

```jsx
<img src={props.imageUrl} className={css.image} />
```

And the result of the rendering would be:

```markup
<img src="..." className="ProfileImage_image_2cvf73" />
```

If we now use the class `image` in another component and also import a CSS file with an `image` class there would be no conflict here, unlike in ordinary CSS, **not**, because the _generated_ class name would be a different one. 

Everything is allowed that is also allowed in CSS and the cascade \(i.e. the **C** in **Cascading Style Sheets**\) remains intact:

```css
.imageWrapper img {
  border: 1px solid red;
}

.imageWrapper:hover img {
  border-color: green;
}
```

The above CSS would be applied to the image with the following JSX structure:

```jsx
const ProfileImage = () => {
  return (
    <div className={css.imageWrapper}>
      <img src="..." />
    </div>
  );
};
```

The background for **CSS Modules** is that possible conflicts in the development of very isolated components should be excluded, while the actual styling itself can still be done in CSS files and completely without JavaScript knowledge. An ideal compromise, especially in teams where the development of JavaScript and CSS has so far been strictly separated and there are experts for both disciplines in the team.

### CSS-in-JS - the shift from styles to JavaScript

I mentioned this in the introduction: **CSS-in-JS** is sometimes discussed very actively and controversially in some developer circles. Opponents argue that supporters of **CSS-in-JS** simply did not understand how to use the cascade to write scalable CSS. Advocates, on the other hand, usually argue that it is not a question of the cascade, but simply that you need a safe way to ensure the highest possible isolation of components. Well, I personally am rather diplomatic and think that there are arguments for both sides, but above all I believe that **CSS-in-JS** has its raison d'être. But what exactly is it all about?

With the **CSS-in-JS** approach, the styling part, which was always the task of CSS in classic web development, is moved to the JavaScript part. As with the **CSS Modules** "Hybrid" solution, the isolation and conflict-free reusability of JavaScript components is the main focus here. Unlike **CSS Modules**, the styling in **CSS-in-JS** takes place entirely in JavaScript files, but still using CSS or at least CSS-like syntax. 

**CSS-in-JS** is also only a generic term for the concept as such, the concrete implementation then takes place through the use of one of meanwhile many libraries. This list provides an overview of the available options: [http://michelebertoli.github.io/css-in-js/](https://michelebertoli.github.io/css-in-js/) - here are listed more than 60 different libraries which can be used as **CSS-in-JS** implementation.

In this chapter, we will limit ourselves to **Styled Components** as a short example, which is the most popular option with over 23,000 stars on GitHub. For this we have to install **Styled Components** as a dependency into our project via the command line:

```bash
npm install styled-components
```

```bash
yarn add styled-components
```

Once the package is installed, we can import it and start with a first **Styled Component**. We import the `styled` function and use the HTML element we want to style as a property. Then we use the **Template Literal** syntax from ES2015 to define the CSS for our **Styled Component**. 

To create a black-yellow button, it would look like this in a complete example:

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
      <h1>Example for a styled component</h1>
      <Button>A black-yellow button with no function</Button>
    </div>
  );
};

ReactDOM.render(<App />, document.getElementById('root'));
```

By using `const Button = styled.button` **Styled Components** creates a new `Button` component and assigns the CSS properties to the component, which we define in the following **Template Literal**. Within this **Template Literals** then again conventional CSS is used. 

If we want to use pseudo-selectors or pseudo-elements, we simply use them without a prefixed selector. In the above example, this looks like this for a `:hover` status:

```jsx
const Button = styled.button`
  background: yellow;
  border: 2px solid black;
  color: black;
  padding: 8px;
  :hover {
    background: gold;
  }
`;
```

Access to the **Props** of a **Styled Component** is also possible. A function is called in the Template String, which gets all **Props** of the respective element as first parameter:

```jsx
const Button = styled.button`
  background: yellow;
  border: 2px solid black;
  color: black;
  cursor: ${(props) => props.disabled ? not-allowed' : 'pointer'};
  padding: 8px;
  :hover {
    background: gold;
  }
`;
```

Here, for example, the mouse pointer would become a `not-allowed` symbol if the button had a `disabled` property. Styled Components also offers support for themes, server-side rendering, CSS animations, and more. For a \(very\) detailed overview, I recommend taking a look at the [good and comprehensive documentation](https://www.styled-components.com/docs/basics).

The positive sides of **CSS-in-JS** in general and **Styled Components** in particular are obvious here and the documentation summarizes them quite well:

* critical CSS, i.e. what is relevant for the currently called page, is generated automatically, because **Styled Components** knows exactly which components are used on a page and which styles they require
* Automatic generation of class names reduces the risk of conflicts to a minimum
* By binding CSS directly to a component, it is very easy to find CSS that is no longer in use. If the generated **Styled Component** is no longer used in an application, the CSS is no longer needed and the component can be safely deleted along with its CSS.
* Component Logic \(JavaScript\) and Component Styling \(CSS-in-JS\) are located directly in the same place, sometimes even in the same file. The developer does not have to laboriously find the place to change when changing the style.
**Styled Components** automatically generates CSS containing vendor prefixes for all browsers without any further configuration

Now **Styled Components** is only one of many implementations of the concept **CSS-in-JS**. She is without a doubt the best known and most widely adapted, and in some of my past projects she has done her job impeccably. However, I suggest that you perhaps take a look at other available alternatives to find the optimal solution for your particular application. Some of the more well-known examples worth mentioning are also:

* [emotion](https://github.com/emotion-js/emotion)
* [styled-jsx](https://github.com/zeit/styled-jsx)
* [react-jss](https://github.com/cssinjs/jss/tree/master/packages/react-jss)
* [radium](https://github.com/FormidableLabs/radium)
* [linaria](https://github.com/callstack/linaria)

