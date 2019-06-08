# JSX - an introduction

## JSX as an important component in React development

Before we go any deeper into the development of components, I'd like to look at **JSX** first, because JSX is an essential part of working with React. As I mentioned at the beginning, JSX is a very basic part of most React components and I think it's one of the reasons why React has been accepted so quickly and positively by so many developers. Meanwhile other frameworks like Vue.js also offer the possibility to use JSX for component-driven development.

At first glance, JSX doesn't look much different from HTML, or rather XML, because in JSX, as in XML and XHTML, every opened element must have a closing element \(`</div>`\) or be self-closing \(`<img />`\). With the basic difference that JSX can rely on **JavaScript expressions** and thus becomes very powerful.

Under the hood, elements used in JSX are converted into nested `React.createElement()` calls in a later build process. We remember back to the introduction. There I mentioned briefly that React creates a tree structure of elements, which itself consists of nested `React.createElement()` calls.

It all sounds terribly complicated now, but it's very simple. Let's have a look at the following short HTML snippet as an example:

```jsx
<div id="app">
  <p>A paragraph in JSX</p>
  <p>Another paragraph</p>
</div>
```

If this HTML is used in this form in **JSX**, these elements will later be transposed by Babel into the following executable JavaScript:

```javascript
React.createElement(
  div,
  { id: 'app' },
  React.createElement('p', null, 'A paragraph in JSX'),
  React.createElement('p', null, 'Another paragraph')
);
```

The first function argument for `createElement()` is the tag name of a DOM element as string representation or another JSX element, but then as function reference.

The second argument represents the **props** of an element, i.e. roughly comparable to HTML attributes, whereby the props in React are much more flexible and, unlike conventional HTML attributes, are not limited to strings, but can also contain arrays, objects or even other React components as values.

All other arguments represent the child elements \(_"children"_\) of the element. In the above example, our `div` has two paragraphs \(`<p>`\) as child elements, which have no props of their own \(`null`\) and only one text string \(`One paragraph in JSX` or `Another paragraph`\) as child element.

If that sounds too complicated to you now, I can reassure you: In practice, this happens afterwards as if by itself. Almost like writing HTML markup. Nevertheless I consider it important to have read the backgrounds at least once to be able to understand later examples better.

## Expressions in JavaScript

What does this mean for our JavaScript expressions, which we can now also use in JSX?

First of all, an expression in JavaScript is, in short, a piece of code that generates a "value" or a "result" at the end. Put simply: everything you can write on the **right** side of the equals sign \(=\) when assigning variables.

```javascript
1 + 5;
```

... is such an expression whose value is `6`.

```javascript
Hal + lo;
```

... is another expression that combines the two strings `Hal` and `lo` by **String Concatenation** to a value `Hallo`.

But instead we could just write the same:

```javascript
6
Hello.
[1,2,3,4]
{a: 1, b: 2, c: 3}
true
nil
```

... because **JavaScript data types** can all be used as expressions.

The ES2015 **Template String Syntax** that uses Backticks \(\`\`\`\) is also an expression. Sure, they're nothing but a string after all:

```javascript
Hello ${name};
```

What's **not** an expression, on the other hand:

```javascript
if (active && visibility == 'visible') { ... }
```

I couldn't write, for example:

```javascript
const isVisible = if (active && visibility === 'visible') { ... }
```

Any JavaScript interpreter would bomb me with this because of invalid syntax.

But if I omit the `if` here, I have a **Logical AND Operator**, which is an expression and has a value to the result \(in this case `true` or `false`\):

```javascript
const isVisible = active && visibility === 'visible';
```

Likewise, the Ternary operator \( ? : \) is an expression:

```javascript
Condition ? true : untrue;
```

Expressions are not limited to boolean values, numbers and strings, but can also be objects, arrays, function calls and even arrow functions, which will also be introduced in ES2015 and will not be the last time we encounter them here.

Example of an Arrow function:

```javascript
(number) => {
  return number * 2;
};
```

All this will become important later. In order to put the crown on the whole thing, expressions can contain JSX themselves and so you can continue the game endlessly. Whew.

Since this will be important later, here are some more examples of JSX that contains valid expressions:

### Simple mathematics

```jsx
<span>5 + 1 = {5 + 1}</span>
```

### Ternary Operator

```jsx
<span>Today is {new Date().getDay() === 1 ? 'Monday' : 'not Monday'}</span>
```

### Ternary operator as value of a prop

```jsx
<div className={user.isAdmin ? is-admin' : null}>...</div>
```

### Array.map\(\) with JSX as return value, which in turn contains an expression

```jsx
<ul>
  {['Tim', 'Struppi'].map((name) => (
    <li>{name})</li>
  ))}
</ul>
```

### numerical values in props

```jsx
<input type="range" min={0} max={100} />
```

All these are first examples of how expressions can be used to make JSX more than just simple HTML.

## What you also need to know

If you have studied the examples carefully, you may have noticed some things depending on your JavaScript knowledge. First of all, the examples seem to have arbitrary brackets. This has the background that JSX must always be written in brackets, i.e. "`(`"" and "`)`", if the JSX extends over more than one line \(i.e. not arbitrarily\).

In principle, it doesn't hurt to always write your complete JSX in brackets, even if it's only a single line. Many people prefer this even for reasons of uniformity, but it is really necessary only for multi-line JSX.

If we want to use a **expression** within the props instead of a **string**, as in the example _"Ternary Operator as value of a prop"_, we use the curly braces instead of single or double attribute quotes to tell React: there is an expression inside.

{% hint style="warning" %}
For objects as a value, **two** opening and closing brackets must be written. The outer brackets introduce the expression \(or end it\) and the inner brackets are those of the actual object:

`<User data={ name: 'Manuel', location: 'Berlin' }} />`

This also applies in a similar way to array literals, with the difference, of course, that the inner brackets are the square brackets that identify the array literals:

`<List items={[1, 2, 3, 4, 5]} />`
{% endhint %}

Furthermore some people might have noticed that in the same example the prop `className` is used. If you have ever worked with the DOM Element API in your browser, you may have remembered that you can access the `class` attributes of an element using `Element.className`. This is exactly the same in React, which uses the properties of the DOM `Element` class.

If you want to set certain HTML attributes in JSX, you have to use the JavaScript equivalent. `class` is a protected keyword in JavaScript to identify a class, so we use `className` instead. The same is true for `for`, which is used as a keyword in JavaScript to initiate loops, but in HTML to tell `<label>` elements which input field they describe. Instead of `for` we write in JSX based on the DOM [HTMLLabelElement Interface](https://developer.mozilla.org/en-US/docs/Web/API/HTMLLabelElement/htmlFor) `htmlFor`:

```jsx
<fieldset>
  <input type="text" id="name" />
  <label htmlFor="name">Name</label>
</fieldset>
```

This pattern runs consistently through all known HTML attributes. If you want to set an HTML attribute in JSX, you must use the spelling of the corresponding **JavaScript DOM Element property**. So `tabindex` becomes `tabIndex`, `readonly` becomes `readOnly`, `maxlength` becomes `maxLength`, and so on.

But: Don't be afraid! In development mode, React will issue a warning in the browser console, so you should seldom not notice any of the incorrect code:

{% hint style="danger" %}
Warning: Invalid DOM property `class`. Did you mean `className`?
{% endhint %}

Who wants to know it exactly: here is the complete list of supported HTML attributes as it is written in the official React-Doku \(to hold, becomes long\):

> `accept acceptCharset accessKey action allowFullScreen alt async autoComplete autoFocus autoPlay capture cellPadding cellSpacing challenge charSet checked cite classID className colSpan cols content contentEditable contextMenu controlsList coords crossOrigin data dateTime default defer dir disabled download draggable encType form formAction formEncType formMethod formNoValidate formTarget frameBorder headers height hidden high href hrefLang htmlFor httpEquiv icon id inputMode integrity is keyParams keyType child label long list loop low manifest marginHeight marginWidth max maxLength mediaGroup method min minLength multiple muted name noValidate nonce open optimum pattern placeholder poster preload profile radioGroup readOnly rel required reversed roleSpan rows sandbox scope scoped scrolling seamless selected shape size sizes span spellCheck src srcDoc srcLang srcSet start step style summary tabIndex target title type useMap value width wmode wrap` wrap` wrap

The same is true for SVG elements, which can also be used within JSX, because SVG is valid XML! The list of supported SVG-attributes is however well and gladly 3 times as long and long not so relevant in everyday life, why I must refer you in this case however really to the appropriate place in the official documentary: [https://reactjs.org/docs/dom-elements.html\#all-supported-html-attributes](https://reactjs.org/docs/dom-elements.html#all-supported-html-attributes)

## Special case Inline-Styles

Of course, there are also special cases. The style attribute is one such attribute. While inline styles are written in HTML with the original CSS property names and as a string, React uses you suspect it: JavaScript properties and also an object instead of a simple string.

Are you writing in traditional HTML:

```markup
<div style="margin-left: 12px; border-color: red; padding: 8px"></div>
```

Does the counterpart in JSX look like this?

```jsx
<div style={ marginLeft: '12px', borderColor: 'red', padding: '8px' }} />
```

Events are another special case. Since these are very extensive, we will deal with the topic again later in a separate chapter.

## Comments in JSX

Comments are also possible in JSX, but do not work like HTML in the form:

```markup
<!-- This is an example of a comment -->
```

... but are also enclosed in curly brackets like expressions and then written in the form of a JavaScript multiline comment:

```jsx
{
  /* This is what a comment in JSX looks like */
}
```

Such comments can, of course, extend over several lines. Single-line JavaScript comments introduced with a double slash \(`//`\) are not possible in JSX. Here you have to use the `/* */` syntax for multiline comments even for a short single-line comment.

This should also give you enough knowledge about JSX to understand the following examples and descriptions better and better in the course of time

## Conclusion

{% hint style="info" %}
* Multiline JSX must always be placed in parentheses
* JSX can process JavaScript expressions. These must be placed in curly brackets and can then also be used in props.
* To set attributes for HTML elements the spelling of the DOM element interface must be used \(`htmlFor` instead of `for`, `className` instead of `class`\)
* CSS Inline-Styles are written as JavaScript Object
* Comments are also enclosed in braces and use multi-line comment syntax: `{/* */}`
{% endhint %}

