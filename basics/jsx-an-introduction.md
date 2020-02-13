# JSX – an Introduction

### JSX as an important building block in React development

Before diving deeper into the development of components, I want to first talk about **JSX**. As mentioned earlier, JSX makes up a large portion of the code written in most React components and thus a large amount in React in general. In my opinion, JSX is one of the reasons why React has been so widely and readily adopted by the developer community. Nowadays, even other frameworks or libraries like Vue.js offer the possibility to use JSX to supercharge their components.

At first glance JSX does not look very different from HTML or XML. As in XML or in XHTML, every opening element also needs to have a matching closing element `</div>` or has to be self-closing \(`<img/>`\). In contrast to XML and XTHML though, JSX can include **JavaScript expressions** making it extremely powerful.

JSX elements are transformed into nested `React.createElement()` calls at a later step in your build process. Remember that I mentioned in the introduction that React creates a tree-like structure of elements - this happens here.

This all sounds a lot more complicated than it actually is though. Let us look at an example with the following HTML snippet:

```jsx
<div id="app">
  <p>A paragraph in JSX</p>
  <p>Another paragraph</p>
</div>
```

If we use this snippet of HTML in **JSX**, Babel will later transpile it into the following executable JavaScript:

```javascript
React.createElement(
  'div',
  { id: 'app' },
  React.createElement('p', null, 'A paragraph in JSX'),
  React.createElement('p', null, 'Another paragraph')
);
```

The first argument for the `createElement()` call denotes the tag name of the DOM element's string representation or another JSX element \(in this case this would only be a function reference\).

The second argument of the `createElement()` call are the **props** of an element. Props are comparable to HTML attributes but are much more flexible than regular HTML attributes. They are not limited to strings but can also include arrays, objects or even other React components as their value.

All other arguments are the so-called _"children"_ of the element. In our example above, the `div` contains two children: two paragraphs \(`<p>`\) . These in turn do not have any other props though \(`null`\)  and only contain a text string \(`A paragraph in JSX`  and `Another paragraph`\) as their children.

If anyone is a little confused at this point or finds JSX a little intimidating, I can assure you that writing JSX will become second nature to you in no time. It will feel like writing HTML markup. I still deem it very important to understand how JSX works under the hood though especially to grasp slightly more complex examples in the future.

## Expressions in JavaScript

What exactly does it mean for us to be able to use JavaScript expressions in JSX?

A JavaScript expression is not much more than a piece of code that will generate a "value" or returns a "result" as a result of its operation. Put simply, anything that you can write on the **right** hand side of a variable declaration \(so after the = sign\):

```javascript
1 + 5;
```

... is an expression whose value is `6`.

```javascript
'Hel' + 'lo';
```

... is another expression that **concatenates** the two strings `Hel` and `lo`  into a single value `Hello`.

Instead, we could use any of the following as all **JavaScript data types** can be used as an expression:

```javascript
6
'Hello'
[1,2,3,4]
{a: 1, b: 2, c: 3}
true
null
```

The ES2015 **Template String Syntax** is yet another expression. Even though we are using backticks \(\`\`\) they are just a plain string in the end:

```javascript
`Hello ${name}`;
```

However, this is **not** an expression:

```javascript
if (active && visibility === 'visible') { … }
```

This is because I could **not** simply write code like this:

```javascript
const isVisible = if (active && visibility === 'visible') { … }
```

Every JavaScript interpreter would rightly complain because of invalid syntax.

If we omit the `if` in this snippet, we are left with a **Logical AND operator** which is indeed an expression and returns a value \(in this case`true` or `false`\):

```javascript
const isVisible = active && visibility === 'visible';
```

The ternary operator \( ? : \) is also an expression:

```javascript
Condition ? true : false;
```

Expressions are not limited to boolean values, numbers or strings but can also include objects, arrays or even function and arrow function calls. You will certainly see them again in the future.

Example for an arrow function:

```javascript
(number) => {
  return number * 2;
};
```

All of this will become important very soon. To put the cherry on top, I will tell you now that expressions can in turn include other JSX. You can play this game until infinity.

Let's look at a few other examples of JSX that contain valid JavaScript expressions to really drive home these concepts:

### Simple mathematics

```jsx
<span>5 + 1 = {5 + 1}</span>
```

### Ternary operator

```jsx
<span>Today is {new Date().getDay() === 1 ? 'Monday' : 'not Monday'}</span>
```

### Ternary operator as a value of a prop

```text
<div className={user.isAdmin ? 'is-admin' : null}>…</div>
```

### Array.map\(\) with JSX as its return value \(which in turn contains an expression\) <a id="array-map-mit-jsx-als-rueckgabewert-das-wiederum-einen-ausdruck-enthaelt"></a>

```jsx
<ul>
  {['Tintin', 'Milou'].map((name) => (
    <li>{name})</li>
  ))}
</ul>
```

### Number values in props

```jsx
<input type="range" min={0} max={100} />
```

All of these are great examples of using expressions to ensure JSX is much more than just HTML.

## What else you should know

For those of you who have paid close attention, you may have noticed a few things depending on the level of your JavaScript knowledge. In some examples, parentheses appear in seemingly odd places. The parentheses, "\(" and "\)", need to be wrapped around any JSX which spans more than one line — so not quite so random anymore.

You can usually just put your JSX in parentheses without problem. Many people actually prefer this practice as all JSX is uniformly treated the same way, but only multi-line JSX actually requires parentheses.

If an **expression** instead of a **string** should be used inside our props \(as in the example "Ternary Operator as a value of a prop"\), we should use curly braces \("{" and "}"\). These indicate to React that an expression is contained within them as opposed to a plain string which would be indicated by single or double quotes.

{% hint style="warning" %}
For each object, **two** opening and closing curly braces need to be used. The outer braces introduce the expression whereas the inner ones represent the curly braces of the object contained within.

`<User data={{ name: 'Manuel', location: 'Berlin' }} />`

Similarly, array literals also need to be included within a set of curly braces. The outer one representing the expression and the inner brackets denoting the array.

`<List items={[1, 2, 3, 4, 5]} />`
{% endhint %}

Some of you may have noticed that the prop `className` was used in the example. The DOM Element API of the browser lets us access the `class` attribute of an element by using `Element.className`. React does just the same and borrows from the properties of the DOM `Element` class.

Some attribute names deviate from the regular JavaScript equivalents in JSX as they use reserved words. In the example, `class` is a reserved word which is why we use `className` instead. Same applies to `for`. The `for` is usually the JavaScript keyword for starting a loop but can also be used to inform a `<label>` which input field it belongs to. Instead of using `for`, JSX code can employ the  [HTMLLabelElement Interface](https://developer.mozilla.org/en-US/docs/Web/API/HTMLLabelElement/htmlFor) `htmlFor`.

```jsx
<fieldset>
  <input type="text" id="name" />
  <label htmlFor="name">Name</label>
</fieldset>
```

This pattern applies to all major HTML attributes. All HTML attributes that should be set need to adhere to the following rules `tabindex` is `tabIndex`, `readonly` to `readOnly`, `maxlength` becomes `maxLength`.

The development mode of the browser usually reminds everyone that the regular HTML attributes should be renamed though and thus makes it easy to avoid these mistakes before they happen. 

{% hint style="danger" %}
Warning: Invalid DOM property `class`. Did you mean `className`?
{% endhint %}

For those of you who want to really understand which HTML attributes are supported in React, this list will help \(fasten your seat belts - it's going to be long\):

> `accept acceptCharset accessKey action allowFullScreen alt async autoComplete autoFocus autoPlay capture cellPadding cellSpacing challenge charSet checked cite classID className colSpan cols content contentEditable contextMenu controls controlsList coords crossOrigin data dateTime default defer dir disabled download draggable encType form formAction formEncType formMethod formNoValidate formTarget frameBorder headers height hidden high href hrefLang htmlFor httpEquiv icon id inputMode integrity is keyParams keyType kind label lang list loop low manifest marginHeight marginWidth max maxLength media mediaGroup method min minLength multiple muted name noValidate nonce open optimum pattern placeholder poster preload profile radioGroup readOnly rel required reversed role rowSpan rows sandbox scope scoped scrolling seamless selected shape size sizes span spellCheck src srcDoc srcLang srcSet start step style summary tabIndex target title type useMap value width wmode wrap`

The same applies to SVG elements. You can also use these in JSX as SVGs are valid XML. The documentation for SVG attributes and which of these are supported is easily at least three times as long than the one for plain HTML modules. Take a look if you want to know more:  [https://reactjs.org/docs/dom-elements.html\#all-supported-html-attributes](https://reactjs.org/docs/dom-elements.html#all-supported-html-attributes)

## Inline styles

Of course there are exceptions: the style attribute is one of them. While inline styles in plain HTML are written as strings with their original CSS property names, React uses JavaScript properties and an object instead of a string.

For comparison - this is regular HTML:

```markup
<div style="margin-left: 12px; border-color: red; padding: 8px"></div>
```

And this is the equivalent in JSX:

```jsx
<div style={{ marginLeft: '12px', borderColor: 'red', padding: '8px' }} />
```

Events form another exception. Events are an extensive topic and thus will be dealt with in an own chapter later.

## Comments in JSX

JSX also allows the use of comments if however they look a bit different from their HTML counterparts:

```markup
<!-- This is an example for an HTML comment -->
```

Instead the comments are contained within curly braces and will be opened in the form of a JavaScript multi-line comment:

```jsx
{
  /* This is a JSX comment */
}
```

Comments can easily span multiple lines of course. In contrast, the usual one-liner JavaScript comments introduced with the double slash \(`//`\) cannot be used in JSX meaning even short one-line comments need to be written with the above \(`/* */`\). 

And this is it! All these examples and explanations should have laid the groundwork for understanding and working with JSX enabling you to follow along in the following chapters.

## Summary

{% hint style="info" %}
* Multi-line JSX has to be surrounded by parentheses
* JSX include JavaScript expressions. These have to be contained in curly braces and can then be used in props
* To use HTML elements, the DOM Element Interface writing standard has to be used \(`htmlFor` instead of `for`, `className` instead of `class`\)
* CSS inline styles have to be written as a JavaScript object
* Comments are put within curly braces and use multi-line comment Syntax: `{/* */}`
{% endhint %}

