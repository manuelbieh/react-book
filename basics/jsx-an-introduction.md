# JSX – an Introduction

### JSX as an important building block in React development

Before diving deeper into the development of components, I want to first talk about **JSX**. As mentioned earlier, JSX makes up a large portion of the code written in most React components and thus a large amount in React in general. In my opinion, JSX is one of the reasons why React has been so widely and happily adopted by the developer community. Nowadays, even other frameworks or libraries like Vue.js offer the possibility to use JSX to supercharge their components.

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

