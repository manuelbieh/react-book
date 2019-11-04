# Introduction to ES2015+

### The "new" JavaScript

**ES2015** is a modern, up-to-date version of JavaScript with lots of new functionality and syntax. **ES2015** is the successor to **ECMAScript** version 5 \(or **ES5**\) and is thus often referred to as **ES6** in many articles and blogs. So if you come across **ES6** anywhere in an article or elsewhere, you know now that it applies to **ES2015**. In most of the book, I will refer to **ES2015+** which means any changes in the JavaScript language since 2015. This includes ES2016 \(ES7\), ES2017 \(ES8\), ES2018 \(ES9\).

{% hint style="info" %}
The **ES** in **ES2015** and **ES6** stands for **ECMAScript**. The organization ECMA International is responsible for the standardization of the **ECMA-262** specification. Since 2015, a new specification has been published each year. Historically, they attributed each version with an increasing number, however to increase clarity and understanding the versions now include the year in which they were published. This way, **ES6** becomes **ES2015**, **ES7** is **ES2016** and so on. 
{% endhint %}

People that work with React most definitely also use **Babel** as their **transpiler** in 99% of the cases as to change **JSX** into their respective `createElement()` calls. But this is not the only thing **Babel** can do. Previously named **6to5**, it achieved just this: transpile **ES6** syntax JavaScript into **ES5**. This unlocks the ability to use newer or soon-to-be-supported features and syntax extensions today while still supporting older browsers that do not natively support these.

In this chapter I want to show and explain the most important and useful functionality and opportunities that **ES2015** and the following versions have brought about. I will focus on those features that you would commonly come across when developing with React and those that make life easiest for you.

**If you already have experience working with ES2015 and its subsequent versions, feel free to skip this chapter!**

## **Variable declarations with let and const**

For a long while, one could only use `var` to declare a variable in JavaScript. Since 2015 however, JavaScript has gained two new keywords which we can use to declare variables: `let` and `const`. Using `var` for variable declarations has become somewhat superfluous and in almost all cases `let` and `const` are the better choices. But what's the difference?

As opposed to `var`, the new variable declarations, `let` and  `const`, only exist **inside of the scope in which they were defined**. These scopes can be a function as it was he case with `var` as well, but it can also be a loop or an `if` statement.

**Tip:** Whenever you find an open curly bracket in your code, you are opeining a new scope. Similarly, a closing curly bracket closes the scope again. Using these new variable declarations, we have encapsulated our variables to a greater degree and limited their usage which is usually considered a good thing.

On the one hand, if you want to override the value of a variable \(e.g. in a loop\), you have to use `let` to achieve this. On the other hand, if the reference of the variable should stay the same, i.e. constant, one should use `const.` 

But be careful: As opposed to other languages, `const` does not disable every mutation on the variable. If you have declared an object or an array with `const`, the actual content of the variable can still be changed. The reference of the variable however is fixed and cannot be changed anymore.

### The difference between `let/const` and `var`

Let us look at an example to further demonstrate the differences between `let` and `const` versus `var` and see how the former are only visible in the scope which they are defined in:

```javascript
for (var i = 0; i < 10; i++) { }
console.log(i);
```

**Result:**

{% hint style="info" %}
10
{% endhint %}

 An now let us look at the same example with `let`

```javascript
for (let j = 0; j < 10; j++) { }
console.log(j);
```

**Result:**

{% hint style="danger" %}
Uncaught ReferenceError: `j` is not defined
{% endhint %}

While the variable `var i` is accessible outside of the `for` loop, the variable defined with `let` is limited to the scope in which it is defined, the `for` loop in this case. Remember, a loop creates a new scope.

This information forms part of one of the building blocks which will help us to write encapsulated components without any side effects.

**The differences between `let` and `const`**

The following code is valid and works for as long as the variable is declared with `let` \(or `var` \):

```javascript
let myNumber = 1234;
myNumber = 5678;
console.log(myNumber);
```

**Result:**

{% hint style="info" %}
5678
{% endhint %}

If we try and run the same code with `const` :

```javascript
const myNumber = 1234;
myNumber = 5678;
console.log(myNumber);
```

**Result:**

{% hint style="danger" %}
Uncaught TypeError: Assignment to constant variable.
{% endhint %}

If we try to directly override a previously declared variable with `const`, the JavaScript interpreter will give us a warning. But what about just changing a property _inside_ of an object declared with `const`?

```javascript
const myObject = {
  a: 1
};
myObject.b = 2;
console.log(myObject);
```

**Result:**

{% hint style="info" %}
`{a: 1, b: 2}`
{% endhint %}

In this instance, we do not encounter any problems as we are only changing the object itself but not the reference to `myObject`. Arrays, similarly, follow the same rules. The elements in the array can be changed, but the value of the variable cannot. 

**Allowed:**

```javascript
const myArray = [];
myArray.push(1);
myArray.push(2);
console.log(myArray);
```

**Result:**

{% hint style="info" %}
`[1, 2]`
{% endhint %}

**Conversely, the following code would not be allowed as the variable would be directly overridden:**

```text
const myArray = [];
myArray = Array.concat(1, 2);
```

{% hint style="danger" %}
Uncaught TypeError: Assignment to constant variable.
{% endhint %}

If we want to ensure that `myArray` can easily be overridden, we have to use `let` instead or we accept that we can only change the elements in the array but not the actual value itself while declaring it with `const`.

## Arrow functions

**Arrow functions** are yet another great addition in terms of simplicity since the introduction of ES2015. Previously function declarations were written in the following fashion: the keyword `function`, followed by an optional function name, parentheses in which the function arguments were given as well as the **function body**. The function body comprises the actual content of the function:

```javascript
function(arg1, arg2) {}
```

**Arrow functions** simplify this greatly by making the `function` keyword redundant:

```javascript
(arg1, arg2) => {}
```

If we only pass one parameter into the function, even the parentheses for the arguments is optional. Our pre-ES2015 function: 

```javascript
function(arg) {}
```

would be transformed into this **arrow function**:

```javascript
arg => {}
```

Yep, this is a valid function in ES2015!

And it's getting wilder. If our function should only return an expression in the `return` value, the parentheses become optional as well. Let's compare a function that takes a number as its single argument, doubles it and `return`s it. This is ES5:

```javascript
function double(number) {
  return number * 2;
}
```

... and this is the same code as a ES2015 **arrow function**:

```javascript
const double = number => number * 2;
```

In both cases, the declared functions yields the same result. For example, if we called the function with `double(5)` , the result would be `10`.

But there is another even greater advantage when it comes to arrow functions. This will be especially useful once we fully start to work with React. Arrow functions do not have their own constructor meaning they cannot be instantiated using the `new MyArrowFunction()` form. On top of this, they do not bind their own `this` but inherit `this` from their **parent scope**. The latter will become useful in the future.

This sounds more complicated than it actually is and can be explained quickly using an example. Let's assume we define a button which should write the current time into a `div` once it is being pressed. A typical function in ES5:

```javascript
function TimeButton() {
  var button = document.getElementById('btn');
  var self = this;
  this.showTime = function() {
    document.getElementById('time').innerHTML = new Date();
  }
  button.addEventListener('click', function() {
    self.showTime();
  });
}
```

As the function supplied as an **event listener** does not have access to its **parent scope**, the **TimeButton** in this case, we have to make amends with saving `this` in the variable `self`. This is not an uncommon pattern for ES5 JavaScript. Alternatively, the scope of the function could be **explicitly** bound to `this` and thus teach the **event listener** which scope to execute the code in:

```javascript
function TimeButton() {
  var button = document.getElementById('btn');
  this.showTime = function() {
    document.getElementById('time').innerHTML = new Date();
  }
  button.addEventListener('click', function() {
    this.showTime();
  }.bind(this));
}
```

`self` becomes superfluous in this example. This example is entirely valid as well, however also not very elegant.

Let us look at an example using **arrow functions** which inherit `this` from their **parent scope**, in this case from the `TimeButton`. 

```javascript
function TimeButton() {
  var button = document.getElementById('btn');
  this.showTime = function() {
    document.getElementById('time').innerHTML = new Date();
  }
  button.addEventListener('click', () => {
    this.showTime();
  });
}
```

And just like that we gained access to the outer `this` in the **event listener**. 

Gone are the days of `var self = this` and `.bind(this)`. We can pretend we are still using the scope of the `TimeButton` scope even though the code is being written inside the **event listener**. This is especially useful for working with complex React components that contain many class properties and methods themselves. It avoids confusion as new scopes are not being opened every time.

## New String, Array and Object methods

With the introduction of ES2015, a lot of static and prototype methods were added to JavaScript. Even if many of these are not directly relevant for React they still prove very powerful which is why I will explain these in  bit more detail.

### String methods

In the past, we had to use `indexOf()` or regular expressions for operations such as checking if a value is present in a string or whether the string starts or ends with a specific value. With ES2015, the string data type gets its own methods to achieve these:

```javascript
string.includes(value);
string.startsWith(value);
string.endsWith(value);
```

In each of these a boolean value is returned by the function, meaning either `true` or `false`. If I want to test whether the string `Example` contains the word `egg`, I can check for it like this:

```javascript
'Example'.includes('egg')
```

Similarly, we can test for `startsWith`:

```javascript
'Example'.startsWith('Ex')
```

... and for `endsWith`:

```javascript
'Example'.endsWith('ample')
```

All of these are case-sensitive, meaning that differences in capitalization are taken into account.

Another two useful methods added in ES2015 are `String.prototype.padStart()` and `String.prototype.padEnd()`. Both of these can be used to extend a string to a specific length by inserting a number of characters at the beginning or end. The first parameter denotes the desired length of the string whereas the optional second parameter describes the character with which you want to fill up the string to the given length. If no second parameter is provided, spaces are used instead.

A common use case is to prepend numbers with leading zeros to give them all the same length:

```javascript
  '7'.padStart(3, '0'); // 007
 '72'.padStart(3, '0'); // 072
'132'.padStart(3, '0'); // 132
```

The same applies to `String.prototype.padEnd()` although the string is filled up at the end \(not at the start\).

### Arrays

The Array has also gained new static as well as prototype methods. What do I even mean when I talk about prototype methods? Prototype methods work "with the actual array" - an already existing **array instance**. Static methods on the other hand can be thought of as helper methods which "do things" that are related to arrays.

**Static array methods**

```javascript
Array.of(3); // [3]
Array.of(1, 2, 3); // [1, 2 ,3]
Array.from('Example'); // ['E', 'x', 'a', 'm', 'p', 'l', 'e']
```

`Array.of()` creates a new array instance from any given number of parameters. It does not matter if their types are different. `Array.from()` also creates an array instance but from an "array-like" iterable object. The most common examples are probably an `HTMLCollection` or a `NodeList`. These are created by DOM methods such as `getElementsByClassName()` or the more modern `querySelectorAll()`. `HTMLCollection` and `NodeList` do not own methods such as `.map()` or `.filter()`. If you want to iterate over any of the two, you need to convert them to an array first. It's easily done by using `Array.from()` though. 

```javascript
const links = Array.from(document.querySelectorAll('a'));
Array.isArray(links); // true
```

**Methods on the array prototype**

The methods on the array prototype can be **directly executed on the array instance**. The most applicable methods for React and Redux are:

```javascript
Array.find(func);
Array.findIndex(func);
Array.includes(value);
```

`Array.find()` finds the **first** element in an array that fulfils the given criteria \(as can be inferred from its name\). It uses a function which is supplied as the first parameter to check for this value. 

```javascript
const numbers = [1, 2, 5, 9, 13, 24, 27, 39, 50];
const biggerThan10 = numbers.find((number) => number > 10); // 13

const users = [
  {id: 1, name: 'Manuel'}, 
  {id: 2, name: 'Bianca'}, 
  {id: 3, name: 'Brian'}
];

const userWithId2 = users.find((user) => user.id === 2);
// { id: 2, name: 'Bianca'}
```

The `Array.findIndex()` method follows a similar pattern but returns only the index of element which has been found as opposed of the `Array.find()` method which returned the value. In the above example, the first function would yield `4`, the second `1`. 

`Array.includes()`, added in ES2016, checks whether a value exists within the array it is called upon and returns a boolean. **Finally!** If you tried to reproduce the same functionality in the past, you had to make do with `Array.indexOf()`. `Array.includes()` simplifies this greatly:

```javascript
[1,2,3,4,5].includes(4); // true
[1,2,3,4,5].includes(6); // false
```

But be careful: `.includes()` is case sensitive. If you try to check for `['a', 'b'].includes('A')`, it will return `false`.

### Objects

**Static object methods**

Arrays and strings are not the only data structures which have gained new functionality. Objects have received a lot of new methods and improvements. Let's look at the most important ones briefly and one after the other:

```javascript
Object.assign(target, source[, source[,...]]);
Object.entries(Object)
Object.keys(Object)
Object.values(Object)
Object.freeze(Object)
```

In my opinion the most useful addition has been `Object.assign()`. This method enables the merge of one or more objects into an existing object and returns the result as an object. But beware: the existing object is **mutated**. Hence, you should use this method sparingly. Another example to illustrate the point:

```javascript
const user = { id: 1, name: 'Manuel' };
const modifiedUser = Object.assign(user, { role: 'Admin' });
console.log(user); 
// -> { id: 1, name: 'Manuel', role: 'Admin' }
console.log(modifiedUser); 
// -> { id: 1, name: 'Manuel', role: 'Admin' }
console.log(user === modifiedUser); 
// -> true
```

The property `role` of the object in the second parameter of the `Object.assign()` function is added to the **existing destination object**. 

React embraces the principle of **pure functions** which denote functions which are encapsulated in themselves and do not modify their entry parameters. Taking this into account, it becomes apparent that such mutations should be avoided if at all possible. We can circumvent this problem by providing an empty object literal as the first argument:

```javascript
const user = { id: 1, name: 'Manuel' };
const modifiedUser = Object.assign({}, user, { role: 'Admin' });
console.log(user); 
// -> { id: 1, name: 'Manuel' }
console.log(modifiedUser); 
// -> { id: 1, name: 'Manuel', role: 'Admin' }
console.log(user === modifiedUser); 
// -> false
```

By providing a newly created object as the destination object, the result is also a different object. In few cases, it can be advantageous to mutate the **destination** **object**, in React however this practice is best avoided. 

The method can process any given objects as parameters. If a property name appears more than once in an object, the properties added later take precedence.

```javascript
const user = { id: 1, name: 'Manuel' };
const modifiedUser = Object.assign(
  {},
  user,
  { role: 'Admin' },
  { name: 'Nicht Manuel', job: 'Developer' }
);
console.log(modifiedUser); 
// -> { id: 1, name: 'Nicht Manuel', role: 'Admin', job: 'Developer' }
```

`Object.entries()`, `Object.keys()` and `Object.values()` offer similar functionality. They return the properties \(`keys`\), the values \(`values`\)  or the entries \(`entries()`\) as an **array**. **Entries** however returns a nested array of the following form:

`[[key, value], [key2, values2], …]`

Applying these methods to our example above, we receive the following results:

```javascript
Object.keys({ id: 1, name: 'Manuel'}); 
// -> ['id', 'name']
Object.values({ id: 1, name: 'Manuel'}); 
// -> [1, 'Manuel']
Object.entries({id: 1, name: 'Manuel'}); 
// -> [['id', 1], ['name', 'Manuel']]
```

Lastly, let us look at `Object.freeze()` which does just what you would expect. It freezes an object and prohibits any further mutations, be it the adding of new properties, deleting old properties or even just changing values. Following React's principle of immutable objects, this is very useful.

```text
const user = Object.freeze({ id: 1, name: 'Manuel' });user.id = 2;delete user.name;user.role = 'Admin';console.log(user);// -> { id: 1, name: 'Manuel' }
```

If an object created with `Object.freeze()` is attempted to be changed, for example using `Object.assign()`, it will throw a `TypeError`, thus preventing unwanted and accidental mutations on the object.

