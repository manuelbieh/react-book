# Introduction to ES2015+

### The "new" JavaScript

**ES2015** is a modern, up-to-date version of JavaScript with lots of new functionality and syntax. **ES2015** is the successor to **ECMAScript** version 5 \(or **ES5**\) and is thus often referred to as **ES6** in many articles and blogs. So if you come across **ES6** anywhere in an article or elsewhere, you know now that it applies to **ES2015**. In most of the book, I will refer to **ES2015+** which means any changes in the JavaScript language since 2015. This includes ES2016 \(ES7\), ES2017 \(ES8\), ES2018 \(ES9\).

{% hint style="info" %}
The **ES** in **ES2015** and **ES6** stands for **ECMAScript**. The organization ECMA International is responsible for the standardization of the **ECMA-262** specification. Since 2015, a new specification has been published each year. Historically, they attributed each version with an increasing number, however to increase clarity and understanding the versions now include the year in which they were published. This way, **ES6** becomes **ES2015**, **ES7** is **ES2016** and so on. 
{% endhint %}

People that work with React most definitely also use **Babel** as their **transpiler** to change **JSX** into its respective `createElement()`, but this is not the only thing **Babel** can do. Previously named **6to5**, it did exactly what its name suggests: transpile **ES6** syntax JavaScript into **ES5**. This unlocks the ability to use newer, or soon-to-be-supported, features and syntax extensions today whilst still supporting older browsers that do not natively support these features.

In this chapter I want to show and explain the most important and useful functionality and opportunities that **ES2015** and the following versions have brought about. I will focus on those features that you would commonly come across when developing with React and those that make life easiest for you.

**If you already have experience working with ES2015 and its subsequent versions, feel free to skip this chapter!**

## **Variable declarations with `let` and `const`**

For a long while, one could only use `var` to declare a variable in JavaScript. Since 2015 however, JavaScript has gained two new keywords which we can use to declare variables: `let` and `const`. Using `var` for variable declarations has become somewhat superfluous and in almost all cases `let` and `const` are the better choices. But what's the difference?

As opposed to `var`, the new variable declarations, `let` and  `const`, only exist **inside of the scope in which they were defined**. These scopes can be a function, as was the case with `var`, but it can also be a loop or an `if` statement.

**Tip:** Whenever you find an open curly bracket in your code, you are opening a new scope. Similarly, a closing curly bracket closes the scope again. Using these new variable declarations, we have encapsulated our variables to a greater degree and limited their usage which is usually considered a good thing.

On the one hand, if you want to override the value of a variable \(e.g. in a loop\), you have to use `let` to achieve this. On the other hand, if the reference of the variable should stay the same, i.e. constant, one should use `const.` 

But be careful: As opposed to other languages, `const` does not disable every mutation on the variable. If you have declared an object or an array with `const`, the actual content of the variable can still be changed. The reference of the variable however is fixed and cannot be changed anymore.

### The difference between `let/const` and `var`

Let us look at an example to further demonstrate the differences between `let` and `const` versus `var` and see how the former are only visible in the scope which they are defined in:

```
for (var i = 0; i < 10; i++) { }
console.log(i);
```

**Result:**

{% hint style="info" %}
10
{% endhint %}

 An now let us look at the same example with `let`

```
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

```
let myNumber = 1234;
myNumber = 5678;
console.log(myNumber);
```

**Result:**

{% hint style="info" %}
5678
{% endhint %}

If we try and run the same code with `const` :

```
const myNumber = 1234;
myNumber = 5678;
console.log(myNumber);
```

**Result:**

{% hint style="danger" %}
Uncaught TypeError: Assignment to constant variable.
{% endhint %}

If we try to directly override a previously declared variable with `const`, the JavaScript interpreter will give us a warning. But what about just changing a property _inside_ of an object declared with `const`?

```
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

In this instance, we do not encounter any problems as we are changing the object itself but not the reference to `myObject`. Arrays, similarly, follow the same rules. The elements in the array can be changed, but the value of the variable cannot. 

**Allowed:**

```
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

```
const myArray = [];
myArray = Array.concat(1, 2);
```

{% hint style="danger" %}
Uncaught TypeError: Assignment to constant variable.
{% endhint %}

If we want to ensure that `myArray` can easily be overridden, we have to use `let` instead or we accept that we can only change the elements in the array but not the actual value itself while declaring it with `const`.

## Arrow functions

**Arrow functions** are yet another great addition in terms of simplicity since the introduction of ES2015. Previously, function declarations were written in the following fashion: the keyword `function`, followed by an optional function name, parentheses in which the function arguments were given as well as the **function body**. The function body comprises the actual content of the function:

```
function(arg1, arg2) {}
```

**Arrow functions** simplify this greatly by making the `function` keyword redundant:

```
(arg1, arg2) => {}
```

If we only pass one parameter into the function, even the parentheses for the arguments is optional. Our pre-ES2015 function: 

```
function(arg) {}
```

would be transformed into this **arrow function**:

```
arg => {}
```

Yep, this is a valid function in ES2015!

And it's getting wilder. If our function should only return an expression in the `return` value, the parentheses become optional as well. Let's compare a function that takes a number as its single argument, doubles it and `return`s it. This is ES5:

```
function double(number) {
  return number * 2;
}
```

... and this is the same code as a ES2015 **arrow function**:

```
const double = number => number * 2;
```

In both cases, the declared functions yields the same result. For example, if we called the function with `double(5)` , the result would be `10`.

But there is another even greater advantage when it comes to arrow functions. This will be especially useful once we fully start to work with React. Arrow functions do not have their own constructor meaning they cannot be instantiated using the `new MyArrowFunction()` form. On top of this, they do not bind their own `this` but inherit `this` from their **parent scope**. The latter will become useful in the future.

This sounds more complicated than it actually is and can be explained quickly using an example. Let's assume we define a button which should write the current time into a `div` once it is being pressed. A typical function in ES5:

```
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

As the function supplied as an **event listener** it does not have access to its **parent scope** — the **TimeButton** in this case — we have to make amends with saving `this` in the variable `self`. This is not an uncommon pattern for ES5 JavaScript. Alternatively, the scope of the function could be **explicitly** bound to `this` and thus teach the **event listener** which scope to execute the code in:

```
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

```
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

```
string.includes(value);
string.startsWith(value);
string.endsWith(value);
```

In each of these a boolean value is returned by the function, meaning either `true` or `false`. If I want to test whether the string `Example` contains the word `egg`, I can check for it like this:

```
'Example'.includes('egg')
```

Similarly, we can test for `startsWith`:

```
'Example'.startsWith('Ex')
```

... and for `endsWith`:

```
'Example'.endsWith('ample')
```

All of these are case-sensitive, meaning that differences in capitalization are taken into account.

Another two useful methods added in ES2015 are `String.prototype.padStart()` and `String.prototype.padEnd()`. Both of these can be used to extend a string to a specific length by inserting a number of characters at the beginning or end. The first parameter denotes the desired length of the string whereas the optional second parameter describes the character with which you want to fill up the string to the given length. If no second parameter is provided, spaces are used instead.

A common use case is to prepend numbers with leading zeros to give them all the same length:

```
  '7'.padStart(3, '0'); // 007
 '72'.padStart(3, '0'); // 072
'132'.padStart(3, '0'); // 132
```

The same applies to `String.prototype.padEnd()` although the string is filled up at the end \(not at the start\).

### Arrays

The Array has also gained new static and prototype methods. What do I even mean when I talk about prototype methods? Prototype methods work "with the actual array" — an already existing **array instance**. Static methods on the other hand can be thought of as helper methods which "do things" that are related to arrays.

**Static array methods**

```
Array.of(3); // [3]
Array.of(1, 2, 3); // [1, 2 ,3]
Array.from('Example'); // ['E', 'x', 'a', 'm', 'p', 'l', 'e']
```

`Array.of()` creates a new array instance from any given number of parameters. It does not matter if their types are different. `Array.from()` also creates an array instance but from an "array-like" iterable object. The most common examples are probably an `HTMLCollection` or a `NodeList`. These are created by DOM methods such as `getElementsByClassName()` or the more modern `querySelectorAll()`. `HTMLCollection` and `NodeList` do not own methods such as `.map()` or `.filter()`. If you want to iterate over any of the two, you need to convert them to an array first. It's easily done by using `Array.from()` though. 

```
const links = Array.from(document.querySelectorAll('a'));
Array.isArray(links); // true
```

**Methods on the array prototype**

The methods on the array prototype can be **directly executed on the array instance**. The most applicable methods for React and Redux are:

```
Array.find(func);
Array.findIndex(func);
Array.includes(value);
```

`Array.find()` finds the **first** element in an array that fulfills the given criteria \(as can be inferred from its name\). It uses a function which is supplied as the first parameter to check for this value. 

```
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

```
[1,2,3,4,5].includes(4); // true
[1,2,3,4,5].includes(6); // false
```

But be careful: `.includes()` is case sensitive. If you try to check for `['a', 'b'].includes('A')`, it will return `false`.

### Objects

**Static object methods**

Arrays and strings are not the only data structures which have gained new functionality. Objects have received a lot of new methods and improvements. Let's look at the most important ones briefly and one after the other:

```
Object.assign(target, source[, source[,...]]);
Object.entries(Object)
Object.keys(Object)
Object.values(Object)
Object.freeze(Object)
```

In my opinion the most useful addition has been `Object.assign()`. This method enables the merge of one or more objects into an existing object and returns the result as an object. But beware: the existing object is **mutated**. Hence, you should use this method sparingly. Another example to illustrate the point:

```
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

```
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

```
const user = { id: 1, name: 'Manuel' };
const modifiedUser = Object.assign(
  {},
  user,
  { role: 'Admin' },
  { name: 'Not Manuel', job: 'Developer' }
);
console.log(modifiedUser); 
// -> { id: 1, name: 'Not Manuel', role: 'Admin', job: 'Developer' }
```

`Object.entries()`, `Object.keys()` and `Object.values()` offer similar functionality. They return the properties \(`keys`\), the values \(`values`\)  or the entries \(`entries()`\) as an **array**. **Entries** however returns a nested array of the following form:

`[[key, value], [key2, values2], …]`

Applying these methods to our example above, we receive the following results:

```
Object.keys({ id: 1, name: 'Manuel'}); 
// -> ['id', 'name']
Object.values({ id: 1, name: 'Manuel'}); 
// -> [1, 'Manuel']
Object.entries({id: 1, name: 'Manuel'}); 
// -> [['id', 1], ['name', 'Manuel']]
```

Lastly, let us look at `Object.freeze()` which does just what you would expect. It freezes an object and prohibits any further mutations, be it the adding of new properties, deleting old properties or even just changing values. Following React's principle of immutable objects, this is very useful.

```
const user = Object.freeze({ id: 1, name: 'Manuel' });
user.id = 2;
delete user.name;
user.role = 'Admin';
console.log(user);
// -> { id: 1, name: 'Manuel' }
```

If an object created with `Object.freeze()` is attempted to be changed, for example using `Object.assign()`, it will throw a `TypeError`, thus preventing unwanted and accidental mutations on the object.

**Syntax extensions and simplifications**

The rest of the changes that were added to objects are not methods but syntax extensions.

First, **computed properties** were introduced to have the possibility of using expressions as object properties. In the past, one had to manually create the object \(for example using an **object literal** `{}` or using `Object.create()`\), assign it to a variable and then add it as a new property to the object:

```
const nationality = 'german';
const user = {
  name: 'Manuel',
};
user[nationality] = true;
console.log(user);
// -> { name: 'Manuel', german: true };
```

With the addition of **ES2015**, expressions can now be directly used as object properties by surrounding them with brackets `[]` avoiding the clunky detour of adding properties to the already existing object:

```
const nationality = 'german';
const user = {
  name: 'Manuel',
  [nationality]: true,
};
console.log(user);
// -> { name: 'Manuel', german: true };
```

 So far so good. I have tried to keep the example very simple at this point but its implications are evident. In more complex situations, this technique will allow us to write clean and easily readable code especially with regard to **JSX**.

Another noteworthy addition for objects has been the introduction of **shorthand property names**. Previously, code had to be written in the following form:

```
const name = 'Manuel';
const job = 'Developer';
const role = 'Author';

const user = {
  name: name,
  job: job,
  role: role,
};
```

We are using a lot of repetition here. **Shorthand property name syntax** in **ES2015** prevents this and allow us to use only the variable name if the name of the object property is the same. This reduces our code to:

```
const name = 'Manuel';
const job = 'Developer';
const role = 'Author';

const user = {
  name, job, role
};
```

Yep, since **ES2015** both of these methods of defining an object will lead to the same result. Of course, shorthand property syntax can be mixed with the previous syntax all the same:

```
const name = 'Manuel';
const job = 'Developer';

const user = {
  name,
  job,
  role: 'Author'
};
```

## Classes

**Classes** are a concept most will know from object oriented languages such as Java. However, with **ES2015** classes have been added to JavaScript. In the past, object orientation could only be mocked by using the prototype property of a function and defining its methods and properties. Compared to many other object oriented languages though, it felt overly complicated and wordy in JavaScript.

Since **ES2015**, classes can be defined by using the keyword `class`. While React equally uses principles of **functional programming**, it also depends heavily on ES2015 classes to instantiate **React Class components**. Before ES2015 it was possible to define React components using `createClass()` but it has since been deprecated and should no longer be used.

A class consists of a name, an optional **constructor** that gets called at creation of a class instance as well as an unlimited number of class methods.

```
class Customer {
  constructor(firstName, lastName) {
    this.firstName = firstName;
    this.lastName = lastName;
  }

  getFullName() {
    return this.firstName + ' ' + this.lastName;
  }
}

const firstCustomer = new Customer('Max', 'Mustermann');
console.log(firstCustomer.getFullName());
```

**Result:**

{% hint style="info" %}
Max Mustermann
{% endhint %}

Additionally, classes can be extended using `extends:`

```
class Customer extends Person {}
```

Or:

```
class MyComponent extends React.Component {}
```

Using the inbuilt `super()` function the component can call the **constructor** of its parent class. In terms of React, the use of `super()` is only necessary if new and own classes have been added and if a constructor is present. If this is the case, `super()` is called and its `props` are passed to the constructor of the `React.Component`.

```
class MyComponent extends React.Component {
  constructor(props) {
    super(props);
  }
}
```

If we omit this, `this.props` would be undefined and the props within the component could not be accessed. In most cases we do not need to define our own constructors though, as React provides **lifecycle methods** which are preferable over the use of a constructor.

## Rest and spread operators and destructuring

Another great simplification has been the introduction of the so-called rest and spread operators for objects and arrays. Strictly speaking, the use of these is not yet supported for objects in ES2015 because the specifics are still under discussion. However, this will change with ES2018 when they get added to the ECMAScript specification. First, the rest and spread operators were added to arrays in ES2015. Using Babel though, we can harness the power of these operators in our work with objects today. As you will see, many React projects will make heavy use of these.

But enough of this. What are they exactly? Let's start with the spread operator.

### Spread operator

The spread operator allows us to "unpack" values. Pre-ES2015 one had to extract arguments from an array to pass them to a function via `Function.prototype.apply()`:

```
function sumAll(number1, number2, number3) {
  return number1 + number2 + number3;
}

var myArray = [1, 2, 3];
sumAll.apply(null, myArray);
```

**Result:**

{% hint style="info" %}
6
{% endhint %}

Using the spread operator which consists of three dots \(...\) we can "unpack" these arguments or "spread" them.

```
function sumAll(number1, number2, number3) {
  return number1 + number2 + number3;
}
var myArray = [1, 2, 3];
sumAll(...myArray);
```

As we can see, `apply()` is not necessary anymore. However, spreading arguments is not only useful and limited to function arguments. It can also be used to easily combine two arrays into one:

```
const greenFruits = ['kiwi', 'apple', 'pear'];
const redFruits = ['strawberry', 'cherry', 'raspberry'];
const allFruits = [...greenFruits, ...redFruits];
```

**Result:**

`['kiwi', 'apple', 'pear', 'strawberry', 'cherry', 'raspberry']`

A new array is created which not only contains all values from `greenFruits` but also all values from `redFruits`. But there's more: a new array is also created for us — not just a reference to the old arrays. Thinking back to the previously mentioned **read-only** mentality in React, this will prove useful for the work with props. The spread operator can also be used to create a simple copy of an array:

```
const users = ['Manuel', 'Chris', 'Ben'];
const selectedUsers = [...users];
```

`selectedUsers` is a copy of `users`and all of its values. If we change the `users`array, no complications for `selectedUsers` occur.

Let's shift our focus to objects. Using the spread operator with objects is very similar to using it with arrays. However, instead of using every single value, every property that is enumerable in the object \(roughly those that are used during a `for(… in …)` loop\) will be used. 

The spread operator is a great choice to create objects:

```
const globalSettings = { language: 'en-US', timezone: 'Berlin/Germany' };
const userSettings = { mutedUsers: ['Manuel'] };
const allSettings = {...globalSettings, ...userSettings};
console.log(allSettings);
```

**Result:**

```
{
  language: 'en-US',
  timezone: 'Berlin/Germany',
  mutedUsers: ['Manuel'],
}
```

The properties of each object can be found in the `allSettings` object. However, the spread operator is not limited to two objects. We can combine any number of objects into a new object. Even the combination of single properties is possible:

```
const settings = {
  ...userSettings,
  showWarnings: true,
}
```

If there are object properties with the same name in any two objects, the property in the object declared last will be used:

```
const globalSettings = { language: 'en-US', timezone: 'Berlin/Germany' };
const userSettings = { language: 'de-DE' };
const allSettings = {...globalSettings, ...userSettings};
console.log(allSettings);
```

**Result:**

```
{
  language: 'de-DE',
  timezone: 'Berlin/Germany',
}
```

The `userSettings` object which has been declared after the `globalSettings` object overrides the `language` property by providing a key which is identical to that in the `globalSettings`object. The spread operator works in a similar fashion to the newly introduced `Object.assign()` method in ES2015 which is also used in in ES2015+ applications from time to time.

However, it is important to make a distinction here: `Object.assign()` mutates an existing object whereas the spread operator creates a new object. In terms of writing React components and their props, we want to avoid creating mutations. But for the sake of completion, let us look at a brief example anyway.

**Combining objects with `Object.assign()`**

`Object.assign()`  takes any number of arguments and combines them into a single object:

```
const a = { a: 1 };
const b = { b: 2 };
const c = { c: 3 };
console.log(Object.assign(a, b, c));
```

**Result:**

```
{a: 1, b: 2, c: 3}
```

The function returns a new object in which all three of the objects that have been passed to `Object.assign()` have been combined. But is this really a new object? **No!** In order to prove this, let's print `a`, `b`, and `c`  to the console:

```
console.log(a);
console.log(b);
console.log(c);
```

 **Result:**

```
{a: 1, b: 2, c: 3}
{b: 2}
{c: 3}
```

As can be seen from the example above, `Object.assign()` did not create a new object for us.  It merely added the properties of the second and the third object to the first. In terms of **pure functions** and **immutable** **objects** this is far from ideal and should be avoided.

There's a trick though to ensure that objects can be combined via `Object.assign()` but also be created in a new object. Passing an empty object literal `{}` as the first argument to the function like this:

```
Object.assign({}, a, b, c);
```

... will achieve just what we want. It passes all the object properties of `a`, `b` and `c` while keeping the previous objects `a`, `b` and `c` intact.

### Destructuring assignments

Before introducing the **rest operator** to you which is closely related to the **spread operator**, I want to talk about **destructuring assignments** \(or **destructuring** for short\).

**Destructuring** allows the extraction of one or more elements from any object or array and to assign it to a new variable. It is another great syntax extension that has been gifted to us with the introduction of ES2015.

**Destructuring Arrays**

Let's imagine that we want to extract the medalists of a 100m run and write them to a new variable. ES5 allows us to express this in the following:

```
const athletes = [
  'Usain Bolt',
  'Andre De Grasse ',
  'Christophe Lemaitre ',
  'Adam Gemili',
  'Churandy Martina',
  'LaShawn Merritt',
  'Alonso Edward',
  'Ramil Guliyev',
];

const gold = athletes[0];
const silver = athletes[1];
const bronze = athletes[2];
```

Thanks to **destructuring**, we can simplify this greatly and reduce the expression to a single statement:

```
const [gold, silver, bronze] = athletes;
```

The array elements at index `0`, `1` and `2` are now all to be found in the variables for `gold`, `silver` and `bronze`. The result is the same as in the previous example, however the second version is much shorter and a lot more succinct.

We can use array destructuring anywhere where the array has been initialized  on the right side, even if it contains a `return` value from the function.

```
const getAllAthletes = () => {
  return [
    'Usain Bolt',
    'Andre De Grasse ',
    'Christophe Lemaitre ',
    'Adam Gemili',
    'Churandy Martina',
    'LaShawn Merritt',
    'Alonso Edward',
    'Ramil Guliyev',
  ] 
}

const [gold, silver, bronze] = getAllAthletes()
```

As can be seen above, the array function returns an array with all the athletes. We can use destructuring directly once it is called and do not need to save the `return` value in a temporary variable.

If we want to omit elements from the array, their values can literally be omitted.

```
const [, silver, bronze] = athletes;
```

In this example, we do not declare a variable gold and only save the silver and bronze medalists to our variables.

But **array destructuring** is not limited to assigning variables with `let` and `const`. There are many other cases that prove useful such as passing function arguments in the form of an array.

```
const logWinners = (athletes) => {
  const gold = athletes[0];
  const silver = athletes[1];
  const bronze = athletes[2];
  console.log(
    'Winners of Gold, Silver and Bronze are', 
    gold, 
    silver, 
    bronze
  );
}
```

But this can be simplfied:

```
const logWinners = ([gold, silver, bronze]) => {
  console.log(
    'Winners of Gold, Silver and Bronze are', 
    gold, 
    silver, 
    bronze
  );
}
```

We are passing the array to our `logWinners()` function and instead of defining a variable to each and every medalist winner we can combine our efforts and simply use the destructuring method.

**Destructuring objects**

The concept of destructuring is not only limited to arrays. Objects can also be assigned to variables if they share the same naming.

The semantics are very similar to those of arrays, with the difference that values are not being assigned based on position but based on their property name. Moreover, curly braces are used instead of brackets.

```javascript
const user = {
  firstName: 'Manuel',
  lastName: 'Bieh',
  job: 'JavaScript Developer',
  image: 'manuel.jpg',
};
const { firstName } = user;
```

`firstName` now contains the value of `user.firstname`.

Object destructuring is among the most widely used features in React components. It allows for writing single props into new variables and to use them in JSX in an uncomplicated fashion.

Let's look at the following example of a stateless functional component ****\(SFC\):

```javascript
const UserPersona = (props) => {
  return (
    <div>
      <img src={props.image} alt="User Image" />
      {props.firstName} {props.lastName}<br />
      <strong>{props.job}</strong>
    </div>
  );
};
```

Having to repeat `props` in front of every property hinders readability of the component. Instead, we can use object destructuring and create a variable for each property of the props once.

```javascript
const UserPersona = (props) => {
  const { firstName, lastName, image, job } = props;
  return (
    <div>
      <img src={image} alt="User Image" />
      {firstName} {lastName}<br />
      <strong>{job}</strong>
    </div>
  );
};
```

That already looks tidier and improves readability of the code, but we can simplify this further. Remember that it was possible to destructure arrays when they were being passed as a function argument. The same applies for objects. Instead of using the `props` argument, we can directly use **destructuring assignment**:

```javascript
const UserPersona = ({ firstName, lastName, image, job }) => (
  <div>
    <img src={image} alt="User Image" />
    {firstName} {lastName}<br />
    <strong>{job}</strong>
  </div>
);
```

As a nice side effect, we can simplify this further as the implicit return from the **arrow function** allows us to omit the curly braces and the explicit `return`. We are left with just 5 lines of **JSX.**

This syntax is very common in SFCs but you can find a similar destructuring assignments in the beginning of the `render()` method of **class components**:

```javascript
render() {
  const { firstName, lastName, image, job } = this.props;
  // more Code
}
```

Of course it is entirely up to you if you make use of destructuring or whether you continue to use `this.props.firstname` in your function. However, the former has developed into some sort of best practice and can be found in most React projects as it aids readability and makes it easier to understand it.

**Renaming properties when destructuring**

Sometimes it is necessary to rename properties. Either because a variable with the same name has already been declared or because a variable with the current property name would be invalid. Both of these are valid concerns and ES2015 offers a solution for these.

```javascript
const passenger = {
  name: 'Manuel Bieh',
  class: 'economy',
}
```

The above `passenger` object contains a property named class which is a valid property name. However, it is not valid as a variable name. Direct destructuring is not possible in this case and would lead to an error:

```javascript
const { name, class } = passenger;
```

{% hint style="danger" %}
Uncaught SyntaxError: Unexpected token }
{% endhint %}

In order to rename the variable, the invalid property name needs to be followed by a colon and the new name. This would pass as a valid **destructuring assignment**:

```javascript
const { name, class: ticketClass } = passenger;
```

The value of the `class` property is written into the new variable `ticketClass` which, as opposed to `class`, is a valid name for a variable. The name of the passenger can easly be destructured into a variable with the name `name`.

**Defaults in destructuring assignments**

It is also possible to use defaults with **destructuring**. If you are trying to destructure a property in an object in which it does not exist, you can define a default to use instead. Instead of using a colon as we did for reserved names, we use the equality sign followed by the default value we want to pass:

```javascript
const { name = 'Unknown passenger' } = passenger;
```

If no property called `name` was present in the object or its value was `undefined`, `name` would result in `Unknown passenger` in this example. If however an empty string is passed or `null`, the default would **not** be used.

**Combining renames and defaults**

Now it's getting interesting because combining renaming and default naming is possible. It might not be the simplest code to follow first but syntactically it is correct. Let use our previous `passenger` object again to illustrate this point. First, we define that the `name` property to be renamed to `passengerName` and then pass it the default value of `Unknown passenger` if no property `name` is found on the object. We also want to keep using `ticketClass` instead of the reserved `class` and assign the value `Economy` to it should it not exist as a property on the object.

```javascript
const {
  name: passengerName = 'Unknown passenger',
  class: ticketClass = 'economy',
} = passenger;
```

If the newly named variables `passengerName` and `ticketClass` are not present in the object to be destructured, the values `Unknown passenger` and `Economy` are assigned. We do however need to ensure that the object itself is not null, otherwise the JavaScript interpreter will throw an error:

```javascript
const {
  name: passengerName = 'Unknown passenger',
  class: ticketClass = 'economy',
} = null;
```

{% hint style="danger" %}
Uncaught TypeError: Cannot destructure property \`name\` of 'undefined' or 'null'.
{% endhint %}

There is an untidy yet highly practical trick to ensure that the object we're working with is not `null` or `undefined`. The **logical OR operator** can be used to declare a fallback object of our actual object is `null` or `undefined.`

```javascript
const {
  name: passengerName = 'Unknown passenger',
  class: ticketClass = 'economy',
} = passenger || {};
```

With the addition of `|| {}` we express that if the `passenger` object is **falsy**, an empty object should be used instead. Arguably, it would be "cleaner" to check if `passenger` is an object first and only then to continue and destructure it. The **Logical OR operator** provides a handy and concise alternative for our tool set though and should be sufficient in most cases.

**Destructuring** and the **spread operator** can be used together too:

```javascript
const globalSettings = { language: 'en-US' };
const userSettings = { timezone: 'Berlin/Germany' };
const { language, timezone } = { ...globalSettings, ...userSettings }
```

In this example, the **spread operator** is revolved first and a new object with all the properties of `globalSettings` and `userSettings` is created. Then, using **destructuring** variables are being assigned.

### Rest operator

The rest operator takes care of the remaining elements in an array or object after **destructuring** or the remaining arguments of **function arguments**. Its name is pretty self-explanatory - it deals with the **"rest"** of the elements. Similarly to the **spread operator**, the **rest** **operator** is preceded by three dots `…`. However, it will not be found on right side of an assignment but on the **left**. Moreover, only **one** rest operator can be used in each expression \(also in stark contrast to the **spread operator**\).

Let's investigate the **rest operator** in function arguments. Assume, that we want to write a function that can take any number of arguments - it doesn't matter whether 2, 5 or 25 arguments are being passed. ES5 offers the keyword `arguments` to us with which we can access an array of all the function arguments within our function:

```javascript
function Example() {
  console.log(arguments);
}
Example(1, 2, 3, 4, 5);
```

**Result:**

{% hint style="info" %}
`Arguments(5) [1, 2, 3, 4, 5, callee: ƒ]`
{% endhint %}

**Arrow functions** however do not offer this functionality anymore and will throw an error if you attempt to access the **arguments**.

```javascript
const Example = () => {
  console.log(arguments);
}
Example(1, 2, 3, 4, 5);
```

**Result:**

{% hint style="danger" %}
Uncaught ReferenceError: arguments is not defined
{% endhint %}

The **rest operator** offers a neat solution to this problem: it writes all the remaining function arguments which we have not already extracted into their own variables into one more variable with any given name:

```javascript
const Example = (...rest) => {
  console.log(rest);
}
Example(1, 2, 3, 4, 5);
```

**Result:**

{% hint style="info" %}
`[1, 2, 3, 4, 5]`
{% endhint %}

But this does not only work for a single function argument but also if you previously defined some other parameters. The **rest operator** will quite literally take care of the **rest**:

```javascript
const Example = (first, second, third, ...rest) => {
  console.log('first:', first);
  console.log('second:', second);
  console.log('third:', third);
  console.log('rest:', rest);
}
Example(1, 2, 3, 4, 5);
```

**Result:**

{% hint style="info" %}
`first: 1    
second: 2    
third: 3    
rest: [4, 5]`
{% endhint %}

One could say that the **rest operator** "collects" the remaining elements from a **destructuring** operation and saves them to variable which is indicated just after the three dots. Even though, we have named this variable `rest` in our previous examples, you can give it any valid name.

The **rest operator** is not limited to functions but can also be used in **array destructuring**:

```javascript
const athletes = [
  'Usain Bolt',
  'Andre De Grasse',
  'Christophe Lemaitre',
  'Adam Gemili',
  'Churandy Martina',
  'LaShawn Merritt',
  'Alonso Edward',
  'Ramil Guliyev',
];
const [gold, silver, bronze, ...competitors] = athletes;
console.log(gold);
console.log(silver);
console.log(bronze);
console.log(competitors);
```

**Result:**

{% hint style="info" %}
```javascript
'Usain Bolt'
'Andre De Grasse'    
'Christophe Lemaitre'`  
[  
  'Adam Gemili',
  'Churandy Martina',
  'LaShawn Merritt',
  'Alonso Edward',
  'Ramil Guliyev'  
]
```
{% endhint %}

... as well as in **object destructuring**:

```javascript
const user = {
  firstName: 'Manuel',
  lastName: 'Bieh',
  job: 'JavaScript Developer',
  hair: 'Brown',
}
const { firstName, lastName, ...other } = user;
console.log(firstName);
console.log(lastName);
console.log(other);
```

**Result:**

{% hint style="info" %}
`Manuel`  
`Bieh`  
`{ job: 'JavaScript Developer', hair: 'Brown' }`
{% endhint %}

All the values not being explicitly written into a variable during **destructuring**, can be accessed via the **rest variable** which we have declared above.

## Template Strings

**Template strings** in ES2015 offer yet a third alternative to write a string. Strings can already be declared by using either single exclamation marks \(`'Example'`\) or double exclamation marks \(`"Example"`\). Since ES2015, backticks can also be used to declare a string: ```Example```.

There are two variations of **template strings**. There are the usual **template strings** which can contain JavaScript expressions as well as **tagged template literals** in their extended form.

**Tagged template literals** are much more powerful than regular **template strings** as their output can be modified with the help of a special function.However, for our usual work with React this is not our primary concern.

If you wanted to mix them with JavaScript expressions or values, you would normally use plain old **string concatenation** pre-ES6 times.

```javascript
var age = 7;
var text = 'My daughter is ' + age + ' years old';
```

```javascript
var firstName = 'Manuel';
var lastName = 'Bieh';
var fullName = firstName + ' ' + lastName;
```

**Template strings** allow us to include **JavaScript expressions** in this variation of a string. In order to do this, you wrap the expression you want to include in this form: `${ }`. Let us look at our previous examples but rewrite them using **template literals**:

```javascript
const age = 7;
const text = `My daughter is ${age} years old`;
```

```javascript
const firstName = 'Manuel';
const lastName = 'Bieh';
const fullName = `${firstName} ${lastName}`;
```

It's important to note that one is not limited to using variables as shown above. Equally, you can use any JavaScript expressions \(i.e. function calls\) - in this case:

```javascript
console.log(`Today's date is: ${new Date().toISOString()}`);
console.log(`${firstName.toUpperCase()} ${lastName.toUpperCase()}`);
```

## Promises and async/await

Promises are not an exclusively new concept in JavaScript, however with ES2015 they been standardized and can be used without any other additional libraries for the first time. Promises allow for asynchronous development by linearizing with callbacks. A promise takes in an **executor function** which again takes two arguments, namely `resolve` and `reject`, and can take any of three states: `pending`, `fulfilled` or `rejected`. The initial state of a promise is always `pending`. Depending on the operation of the executor function being successful or unsuccessful, meaning `resolve` or `reject`, the status of the of the promise will change to either `fulfilled` or `rejected`. You can react directly to these two states with`.then()` or `.catch()` respectively. If `resolve` is being passed to the executor function, the `then()` part will execute. If `reject` is called, **all** `then()` calls are skipped and the `catch()` block is executed instead.

An executor function **has** to execute one of the two functions it has been passed, otherwise the promise will be stuck in an _unfulfilled_ state. This can lead to unexpected and false behavior and can even cause memory leaks within an application.

To reiterate the difference between promises and callbacks, let us have a look at the following example:

```javascript
const errorHandler = (err) => {
  console.error('An error occured:', err.message);
};

getUser(id, (user) => {
  user.getFriends((friends) => {
    friends[0].getSettings((settings) => {
      if (settings.notifications === true) {
        email.send('You are my first friend!', (status) => {
          if (status === 200) {
            alert('User has been notified via email!');
          }
        }, errorHandler);
      }
    }, errorHandler);
  }, errorHandler)
}, errorHandler)
```

The asynchronous `getUser()` function is called to obtain information about a user with their `id`. Using its `getFriends()` method, we can obtain a list of all of their friends. To collect the first friend's \(`friends[0]`\) user settings, the `getSettings()` method is called. If this friend has allowed email notifications, an email is sent to him. We react asynchronously to the response of the mail server.

Albeit this example being relatively simple, the code is nested **6 layers** deep - and we are not even taking care of explicit error handling or edge cases. Working with callbacks can become cumbersome and confusing quickly, especially one callback functions will trigger further callback functions, as in our example. It comes as no surprise that this is often called **Callback Hell** or the **Pyramid of Doom**.

Let us rewrite this and assume that our fictional API methods will each return ****a promise:

```javascript
const errorHandler = (err) => {
  console.error('An error occured:', err.message);
};

getUser(id)
.then((user) => user.getFriends())
.then((friends) => friends[0].getSettings())
.then((settings) => {
  if (settings.notifications === true) {
    return email.send('You are my first friend!');
  }
})
.then((status) => {
  if (status === 200) {
    alert('User has been notified via email');
  }
})
.catch(errorHandler);
```

Each step is being followed by `then()` to react to the previous promise.  Although the same end result is reached by this piece of code, we are only nesting 2 layers deep and readability is improved immensly.

It is relatively simple to refactor code based on callbacks into one that is using promises. In order to demonstrate this, I will show you an example of the Geolocation API and its `getCurrentPosition()` method. For those of you who do not know it: the method exists on the `navigator.geolocation` object and opens a dialog in the browser asking the user for permission to use their current location. The method expects two callbacks as arguments, namely the success callback and the error callback. The first callback, the  success callback, is being passed an object with the user's location if they have previously agreed. The second callback, the error callback, is being passed an error object. This can occur because the user has given permission to use their current location or due to a location not being possible for other reasons.

```javascript
navigator.geolocation.getCurrentPosition((position) => {
  console.log(`User position is at ${position.coords.latitude}, ${position.coords.longitude}`);
}, () => {
  console.log('Unable to locate user');
});
```

The callback can be transformed into a promise:

```javascript
const getCurrentPositionPromise = () => {
  return new Promise((resolve, reject) => {
    navigator.geolocation.getCurrentPosition(resolve, reject);
  });
};
```

Yep, that's it! Now we can access the user's location like the following instead of using callback syntax:

```javascript
getCurrentPositionPromise()
.then((position) => {
  console.log(`User position is at ${position.coords.latitude}, ${position.coords.longitude}`);
})
.catch(() => {
  console.log('Unable to locate user');
});
```

A few of the newer JavaScript Browser APIs have already followed this new implementation. If you want to learn about promises and how they work in detail, I suggest you to read the [the following article on the MDN web docs](https://developer.mozilla.org/de/docs/Web/JavaScript/Reference/Global_Objects/Promise). The explanation and reasoning for promises should only serve as an introduction to a much more interesting feature though, namely:

### Asynchronous functions with `async` / `await`

Asynchronous functions with `async` and `await` can be seen as the next step in the asynchronous evolution in the JavaScript ecosystem and been added to the JavaScript specification in ES2016. While still using promises under the hood, `async/await` make them invisible and allow us to write asynchronous code that resembles synchronous code. No more callbacks and `then()` or `catch()`.

In order to use it, prepend the asynchronous function with the `await` keyword. However, to use `await` the `async` keyword needs to be applied in the function call in which`await` is used. It tells the JavaScript interpreter that we are using an asynchronous function and omitting it would throw an exception.

Bringing it back to our previous example of the user that wants to send an email to his first friend, we can rewrite it using these asynchronous functions:

```javascript
(async () => {
  try {
    const user = await getUser(id);
    const friends = await user.getFriends();
    const settings = await friends[0].getSettings();
    if (settings.notifications === true) {
      const status = await email.send('You are my first friend!');
      if (status === 200) {
          alert('User has been notified via email');
      }
    }
  } catch (err) {
    console.error('An error occured:', err.message);
  }
})();
```

For me personally asynchronous functions with `async` and `await` have been one of the most notable changes in JavaScript in the past few years. They facilitate working with asynchronous data greatly especially compared to complex and confusing callbacks. Even promises, which were already a massive improvement over callbacks, look almost complex compared to `async` / `await`. 

## Import syntax and JavaScript modules

Modules in JavaScript are a bit of special topic. They did not exist officially but there have been many attempts to introduce JavaScript modules in the past. If you have been part of the development scene for a while, you might remember **AMD** \(Asynchronous Module Definition\) and if you have worked with Node.js before you should have come across CommonJS Modules \(i.e. `module.exports` and `require('./myModule')`\). There has been a lot of debate centering on which module standard should become the de-facto standard, what the syntax is supposed to look like and how the implementation should work on the side of the interpreter. In the end, consensus was reached on using `import` and `export` keywords to let the modules communicate with each other.

Babel was first to implement a solution that was based on the previous standard of the specification. This implementation had to change in between though as updates were made to the standard. When webpack entered the scene, it implemented yet another mechanism to manage resolving and loading of JavaScript modules which is based on the now approved final standard. Just like TypeScript.

After **10** years, we have finally reached consensus for the specification and JavaScript engines are busy implementing it. This sounds rather complicated. It surely has been in the past. However, nowadays most agree and developers finally found clarity. But there are a few gotchas still which is why we should still rely on Webpack, Babel or TypeScript to work with modules. But more on that later.

We've talked a lot about the history of modules now. But how do imports work and what even are modules?

### Modules in JavaScript

The primary goal of modules is to encapsulate JavaScript scope for each module. In this case, a module is actually a single **file**. ****As long as you are not explicitly limiting their scope by using an **IIFE** \(_Immediately Invoked Function Expression_\), each function, each variable etc that you declare in JavaScript is available globally. Modules prevent this by only making the code inside it available **in the module itself**. This can avoid complications such as two libraries using the same variable. Moreover, modules encourage reusable code without being scared of using already existing variables or overriding functions that have already been declared.

Modules can **export:** functions, classes or variables that have been declared within them. Other modules can then import these exports if necessary. In order to export these functions and variables, a special `export` keyword is used. To import these, you might recognize a pattern here, there is an `import` keyword. Exporting can take two forms: **named exports** and **default exports**.

**Named Exports**

Assume that we have created a module - `calc.mjs` - that provides a number of functions for us to make more complicated calculations. For example, the module could contain the following:

```javascript
export const double = (number) => number * 2;
export const square = (number) => number * number;
export const divideBy = (number, divisor) => number / divisor;
export const divideBy5 = (number) => divideBy(number, 5);
```

We announce an **export** by using the keyword. Then, declaring a variable and assigning it an arrow function with one or more parameters and directly returning the result we could use this function elsewhere in another module. Alternatively, we can achieve the same with two separate steps:

```javascript
const double = (number) => number * 2;
export double;
```

Using the `import` keyword, we can now use these function elsewhere in our application. To do this, we use `import` followed by the exports that we want to import in curly braces as well as `from` and the path to module.

```javascript
import { double, square, divideBy5 } from './calc.mjs';

const value = 5;
console.log(double(value));     // 10
console.log(square(value));     // 25
console.log(divideBy5(value));  // 1
```

Theoretically, a file can have **as many exports as you want**. Beware though, they have to have different names and an already exported name **is not allowed to be exported again**.

**Default Export**

In addition to the so-called **named exports** from the example above, there is also the singular `default export`. This is a special form of an export that is only allowed to be used within a module **once** and declared by using `default`. If a variable or function has been marked as `default` it is possible to import it without curly braces. The **default export** can alleviate the need for importing a lot of named exports by bundling them into a single default. 

```javascript
export const double = (number) => number * 2;
export const square = (number) => number * number;
export const divideBy = (number, divisor) => number / divisor;
export const divideBy5 = (number) => divideBy(number, 5);

export default {
   double, square, divideBy, divideBy5
}
```

To use any of the above functionality, our application would only need to import the module itself - meaning the **default export** - and assign it to a variable:

```javascript
import Calc from './calc.mjs';

console.log(Calc.double(value));    // 10
console.log(Calc.square(value));    // 25
console.log(Calc.divideBy5(value)); // 1
```

In most cases, it is useful for each module to have a **default export**. Especially in many component based libraries like React or Vue.js it is common to only define one export per module, which should be the **default export**. Even if this is not necessary from a syntactical point of view, it has become the standard for working with React.

```javascript
export default class MyComponent extends React.Component {
  // ...
}
```

### Inconsistencies: Browser vs Node.js

If you paid attention, you might have noticed that we imported a file named `calc.mjs` \(not `calc.js`\) above. This convention has been agreed upon in a lenghty process to ensure that JavaScript modules can be used in Node.js. 

So if you want to write universal JavaScript that can be run on the server with Node.js but also client-side in the browser, you **have to** use the `.mjs` file ending for all your files. Especially, if you want to achieve this without tools such as Babel, Wepback or TypeScript.

Loading modules works a little different in Node.js compared to the browser. While the browser does not care which file ending a module has \(as long as the server sends it back with the Content-Type `text/javascript`\), Node.js requires the `.mjs` file ending to classify JavaScript modules.

To use JavaScript modules in the browser, the `type` attribute needs to be set to `module` in the corresponding `<script></script>` element:

```markup
<script src="./myApp.mjs" type="module"></script>
```

Those browsers that support `type="module"`, also support the `nomodule` attribute to indicate a fallback for browsers that do not support modules and safely ignore it.

```markup
<script src="./myApp.mjs" type="module"></script>
<script src="./myApp.bundle.js" nomodule></script>
```

A browser supporting modules would simply load `myApp.mjs` while all other browsers would just serve a bundled `myApp.bundle.js` \(for example generated with Webpack\).

But that's not all: Node.js has its own, very special mechanism to find and load files. Modules that do not have a relative file path, i.e. not start with `./` or `../`, will be looked for in `node_modules` or `node_libraries` for example. Node.js also loads an `index.js` by default if it finds a directory with the name you specified.

```javascript
import MyModule from 'myModule';
```

Node.js would look for an export in a directory such as `./node_modules/myModule` in this case and load an `index.js` that is inside of it. Alternatively, it would look for the file in the `main` field of the `package.json`. The browser on the other hand cannot try a bunch of different file paths to find a file because each and every search would result in an expensive network request and many costly 404 responses.

On top of that the module that you want to import from, also called the **Import Specifier** \(that's the part just after the `from`\), is protected in the browser. It has to consist of either a valid URL or a valid file path.

Imports like this are not possible in browsers to date:

```javascript
import React from 'react';
```

If we want to use JavaScript modules server-side as well as client-side, we will have to keep using module bundlers like Webpack for now. The proposal for **Package Name Maps** which is supposed to solve this problem is only in the early stages of discussion and will thus only land in a later version of ECMAScript.

## Summary

ES2015 and the following versions of the ECMAScript standard have given us a lot of new functionality that did not exist in JavaScript before. It's hard to imagine working without these especially in the use with React. The most notable new features are these:

* Variable declarations with `let` and `const`
* **Arrow functions** - to create functions that do not bind their own `this`
* **Classes** - they are the basis of **React Class components** and facilitate a multitude of things
* **Rest and Spread operators** - which increase the readability as well as the writing process of data in arrays and objects immensely
* **Template strings** - to facilitate working with JavaScript expressions in Strings
* **Promises** and **Asynchronous functions with `async` / `await`** - which enhance working with asynchronous data and make it much easier
* **Import** and **Export** for encapsulating reusable JavaScript on a module basis

