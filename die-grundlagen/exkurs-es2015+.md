# Excursus ES2015+

## The "new" JavaScript

**ES2015** is in short a modernized, current version of JavaScript with many new functions and syntax simplifications. **ES2015**** is the successor of **ECMAScript** in version 5 \(**ES5**\), so it was originally called **ES6**** and is still called so in some blogs and articles. If you come across the term **ES6,** when reading articles about React, this means **ES2015**. I write here mostly from **ES2015+** and my changes that have been included in JavaScript since 2015. These include ES2016 \(ES7\), ES2017 \(ES8\) and ES2018 \(ES9\).

{% hint style="info" %}
The **ES**** in **ES2015** and **ES6**** stands for **ECMAScript**. ECMA International is the organization behind the standardization of the **ECMA-262-** specification on which JavaScript is based. Since 2015, new versions of the specification have been published annually, which for historical reasons had a sequential version number starting with version 1, but then adopted the year of their publication for greater clarity. Today, **ES6** is officially referred to as **ES2015**, **ES7** as **ES2016**, and so on.
{% endhint %}

Who works with React, uses in probably 99% of the cases also **Babel** as **Transpiler,** to transpose his **JSX** accordingly into `createElement()` calls. But **Babel** doesn't just transpose **JSX** into executable JavaScript, it was originally called **6to5** and did exactly that: transposing JavaScript written with **ES6** syntax into **ES5**, so that newer, future features and syntax enhancements could be used in older browsers without support for "the new" JavaScript.

The most important and useful new features and possibilities in **ES2015** and the following versions I would like to discuss in this chapter. I'll limit myself to the new features that will be more common when working with React and that will make life easier for you developers.

**If you already have experience with ES2015 and subsequent versions, you can skip this chapter!**

## Variable declarations with let and const

While previously there was only `var` to declare a variable in JavaScript, ES2015 adds two new keywords to declare variables: `let` and `const`. A variable declaration with `var` becomes superfluous in almost all cases, usually `let` or `const` are the cleaner choice. But what's the difference?

Unlike `var`, variables declared with `let` or `const` exist **only within the scope in which they were declared!** Such a scope can be a function like the one that has already created a new scope with `var`, but also loops or even `if-`statements!

**Rough rule of thumb:** Everywhere where you find an opening curly bracket, a new scope is also opened. Consequently, the closing bracket closes this scope again. This makes variables much more restricted and encapsulated, which is usually a good thing.

If you want to overwrite the value of a variable again, for example in a loop, you have to declare the variable with `let`. If you want to keep the reference of the variable unchangeable, `const` should be used.

But be careful: unlike other languages, `const` does not mean that the entire content of the variable remains constant. With objects or arrays their contents can be changed also with variables declared with `const`. Only the reference object to which the variable points can no longer be changed.

### The difference between `let`/`const` and `var`

First a short example of how the variable declaration of `let` and `const` differ from the variable declaration of `var` and what it means that the former are only visible in the scope in which they were defined:

```javascript
for (var i = 0; i < 10; i++) { }
console.log(i);
```

**output:**

{% hint style="info" %}
10
{% endhint %}

Now the same example with `let`.

```javascript
for (let j = 0; j < 10; j++) { }
console.log(j);
```

**output:**

{% hint style="danger" %}
Uncaught ReferenceError: `j` is not defined
{% endhint %}

While the variable `var i`, once defined, can also be accessed outside the `for` loop, the variable `let j` exists only inside the scope in which it was defined. And that in this case is inside the `for` loop that creates a new scope.

This is a small building block that will later help us to encapsulate our components and create them without unwanted side effects.

#### Differences between `let` and `const`

The following code is valid and works as long as the variable is declared with `let` \(or `var`\):

```javascript
let myNumber = 1234;
myNumber = 5678;
console.log(myNumber);
```

**output:**

{% hint style="info" %}
5678
{% endhint %}

The same code again, but now with `const`:

```javascript
const myNumber = 1234;
myNumber = 5678;
console.log(myNumber);
```

**output:**

{% hint style="danger" %}
Uncaught TypeError: Assignment to constant variable.
{% endhint %}

So here we try to overwrite a variable declared by `const` directly and are rightly put in the limits by the JavaScript interpreter. But what if we only want to change a property _within_ an object declared with `const` instead?

```javascript
const myObject = {
  a: 1
};
myObject.b = 2;
console.log(myObject);
```

**output:**

{% hint style="info" %}
`{a: 1, b: 2}`
{% endhint %}

In this case there are no problems, because we do not change the reference to which the `myObject` variable should refer, but the object to which it should refer. This also works with arrays that can be changed as long as the value of the variable itself is not changed!

**permitted:**

```javascript
const myArray = [];
myArray.push(1);
myArray.push(2);
console.log(myArray);
```

**output:**

{% hint style="info" %}
`[1, 2]`
{% endhint %}

**Not allowed, because we would overwrite the variable directly:**

```text
const myArray = [];
myArray = Array.concat(1, 2);
```

{% hint style="danger" %}
Uncaught TypeError: Assignment to constant variable.
{% endhint %}

So if we want to keep `myArray` overwritable, we have to use `let` instead or content ourselves with the fact that the content of the array declared with `const` can be changed, but not the variable itself.

## Arrow Functions

**Arrow Functions** are another **clear** simplification that ES2015 has brought us. Until now, a function declaration worked like this: you wrote the keyword `function`, optionally followed by a function name, parentheses in which the function arguments were described, and the **Function Body**, the actual content of the function:

```javascript
function(arg1, arg2) {}
```

**Arrow Functions** simplify this by making the `function` keyword superfluous:

```javascript
(arg1, arg2) => {}
```

If we only have one parameter, even the brackets are optional for the arguments. From our function

```javascript
function(arg) {}
```

would become the following **Arrow Function**:

```javascript
arg => {}
```

Yeah, that's a valid function in ES2015!

And it's getting wilder. If our function should only return an expression as `return` value, the brackets are optional as well. Let's compare a function, which takes a number as only argument, doubles it and returns it as `return` value from the function. Once in ES5:

```javascript
function double(number) {
  return number * 2;
}
```

... and as ES2015 **Arrow Function**:

```javascript
const double = number => number * 2;
```

In both cases the just declared function returns `10` as the result when calling e.g. `double(5)`!

But there is another important advantage that will be very useful when working with React: Arrow Functions do not have their own constructor, so they cannot be created as an instance in the form `new MyArrowFunction()`, and do not bind their own `this` but inherit `this` from their **Parent Scope**. The latter in particular will still be very helpful.

This also sounds terribly complicated, but can also be explained quite quickly using a simple example. Let's say we define a button that should write the current time into a `div` as soon as I click on it. A typical function in ES5 could look like this:

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

Since the function specified as **Event Listener** has no access to its **Parent Scope**, i.e. the **TimeButton**, we store `this` in the variable `self` here alternatively. No unusual pattern in ES5. Alternatively, you could explicitly bind the scope of the function to `this` and teach the **Event Listener** in which scope its code should be executed:

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

Here you save at least the additional variable `self`. This is also possible, but not particularly elegant.

At this point the **Arrow Function** comes into play, which, as just mentioned, gets `this` from its **Parent Scope**, in this case from our `TimeButton` instance:

```javascript
function TimeButton() {
  var button = document.getElementById('btn');
  this.showTime = function() {
    document.getElementById('time').innerHTML = new Date();
  }
  button.addEventListener('click', () {
    this.showTime();
  });
}
```

And already we have access to `this` of the overlying scope in the **Event Listener**!

No `var self = this` acrobatics anymore and also no `.bind(this)`. We can work within the Event Listener as if we were still in the `TimeButton` Scope! This is especially helpful later when working with extensive React components with their own class properties and methods, as it prevents confusion and does not always create a new scope.

## New methods for strings, arrays and objects

ES2015 also introduced a number of new static and prototype methods into JavaScript. Even though most of them are not directly relevant for the work with React, they make the work a lot easier at times, which is why I would like to briefly discuss the most important ones here.

### String methods

If you have set a string to `indexOf()` or regular expressions in the past to check if it contains a certain value, starts with a certain value or stops, the string datatype now gets its own methods for it.

These are:

```javascript
string.includes(value);
string.startsWith(value);
string.endsWith(value);
```

A boolean is returned, i.e. `true` or `false.` If I want to know if my string contains `example`an `ice`, I simply check for

```javascript
'Example'.includes('ice')
```

It behaves analogously with `startsWith`:

```javascript
'Example'.startsWith('Bei')
```

...as well as with `endsWith`:

```javascript
Example'.endsWith('game')
```

The method is case-sensitive, i.e. it distinguishes between upper and lower case.

Two other helpful methods that have been introduced into JavaScript with ES2015 are `String.prototype.padStart()` and `String.prototype.padEnd()`. You can use these methods to lengthen a string by adding \(`.padStart()`\) at the beginning or \(`.padEnd()`\) at the end until the specified length is reached. The first parameter specifies the desired length, the optional second parameter the character with which you want to fill the string up to this position. If you do not specify a second parameter, a space is used by default.

This is helpful, for example, if you want to fill numbers so that they are always uniformly three-digit:

```javascript
  '7'.padStart(3, '0'); // 007
 '72'.padStart(3, '0'); // 072
'132'.padStart(3, '0'); // 132
```

`String.prototype.padEnd()` works the same way, with the difference that it fills your string at the end, not at the beginning.

### Arrays

The array methods have both new static and methods on the array prototype. What does this mean? Prototype methods work "with the array" as such, i.e. with an existing **Array instance**, static methods are in a broader sense helper methods that do certain things that "have to do with arrays".

#### Static array methods

Let's start with the static methods:

```javascript
Array.of(3); // [3]
Array.of(1, 2, 3); // [1, 2, 3]
Array.from('Example'); // ['E', 'x', 'a', 'm', 'p', 'l', 'e']
```

`Array.of()` creates a new array instance from any number of parameters, regardless of their types. `Array.from()` also creates an array instance, but from an "array-like" iterable object. Probably the most handy example for such an object is a `HTMLCollection` or a `NodeList`. Such can be obtained by using DOM methods like `getElementsByClassName()` or the more modern `querySelectorAll()`. They don't have methods like `.map()` or `.filter()` themselves. So if you want to iterate over such an array, you first have to convert it into an array. With ES2015 this can now be easily done by using `Array.from()`.

```javascript
const links = Array.from(document.querySelectorAll('a'));
Array.isArray(left); // true
```

#### methods on the array prototype

The methods on the array prototype can be applied **directly to an array instance**. The most common during work with React and especially later with Redux are:

```javascript
Array.find(function);,
Array.findIndex(function);
Array.includes(value);
```

The `Array.find()` method is used, as the name suggests, to find the **first** element of an array that satisfies certain criteria that are checked by the function passed as the first parameter.

```javascript
const numbers = [1,2,5,9,13,24,27,39,50];
const biggerThan10 = numbers.find((number) => number > 10); // 13

const users = [
  One, name Manuel, 
  {id: 2, name: 'Bianca'}, 
  {id: 3, name: 'Steve'}
];
const userWithId2 = users.find((user) => user.id === 2); // { id: 2, name: 'Bianca'}
```

The `Array.findIndex()` method follows the same signature, but, unlike the `Array.find()` method, does not return the found element itself, but only its index in the array. In the above examples these would be `3` and `1`.

The new method `Array.includes()` added in ES2016 checks if a value exists within an array and returns **finally** a boolean. Who has realized this in the past with `Array.indexOf()` will remember how awkward it was. Now a simple `Array.includes()`:

```javascript
[1,2,3,4,5].includes(4); // true
[1,2,3,4,5].includes(6); // false
```

Attention: the method is case-sensitive. `['a', 'b'].includes('A')` returns `false`.

### Objects

#### Static object methods

Of course, objects have also been given a number of new methods and other beautiful possibilities. The most important at a glance:

```javascript
Object.assign(target, source[, source[,...]]);
Object.entries(Object)
Object.keys(Object)
Object.values(Object)
Object.freeze(Object)
```

One by one again. The most useful one from my point of view is `Object.assign()`. This makes it possible to add the properties of one or more objects to an existing object \(a merge\, so to speak). The method returns the result as an object. However, there is also a mutation of the **target object**, so this method should be used with caution. Examples say more than words, please:

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

So here we add the `role` property from the object in the second parameter of the `Object.assign()` method to the existing **target object**.

Since React follows the principle of **Pure Functions**, i.e. functions that are self-contained and do not modify their input parameters, such mutations should be avoided as far as possible. We can avoid this by simply passing an object literal as the first parameter:

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

By using a newly created object as the target object, we also get a different object as the return value than in the first example. In some cases it may be desirable to mutate the **target object** instead of creating a new object, but in most cases this is not the case while working with React.

The method also processes any number of objects as parameters. If there are properties with the same name in an object, later properties have priority:

```javascript
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

The three static object methods `Object.entries()`, `Object.keys()` and `Object.values()` work basically very similar, they return to a given object the properties \(`keys`\), the values \(`values`\) or the entries \(`entries`\) as **Array**, where the **Entries** are a nested array in the form`[[key, value], [key2, values2], ...]`.

Applied to our example above, this will result in the following return values:

```javascript
Object.keys({ id: 1, name: 'Manuel'}); 
// -> ['id', 'name']
Object.values({ id: 1, name: 'Manuel'}); 
// -> [1, 'Manuel']
Object.entries({id: 1, name: 'Manuel'}); 
// -> [['id', 1], ['name', 'Manuel']]
```

Finally, we look at `Object.freeze()`. This method is also quite self-explanatory and does exactly what its name suggests: it freezes an object, so it forbids the developer to add new properties and delete or even modify old ones. This, too, is incredibly practical in dealing with the objects that are immutable in React in most cases \(or at least should be \).

```javascript
const user = Object.freeze({ id: 1, name: 'Manuel' });
user.id = 2;
delete user.name;
user.role = 'Admin';
console.log(user);
// -> { id: 1, name: 'Manuel' }
```

An object created with `Object.freeze()` also provides good protection against accidental mutation with the new `Object.assign()` method described above. If you try to modify an object created with `Object.freeze()` using `Object.assign()`, this will inevitably lead to its `TypeError`.

#### Syntax extensions and simplifications

The latest changes to the way objects work are not methods, but syntax enhancements.

The first are the **Computed Properties**** \(i.e. _calculated properties_\). Behind this lies the possibility to use expressions \(or their values\) as object properties. If, for example, you wanted to set a property in an object in the past, you would usually create the object \(e.g. as **ObjectLiteral** `{}` or via `Object.create()`\), assign it to a variable and then add the new property to the object:

```javascript
const nationality = 'german';
const user = {
  name: Manuel,
};
user[nationality] = true;
console.log(user);
// -> { name: 'Manuel', german: true };
```

**ES2015**** now allows us to use expressions directly as object properties by putting them in square brackets `[]`. This saves us the detour of having to add properties to the already created object afterwards:

```javascript
const nationality = 'german';
const user = {
  name: 'Manuel', [nationality]: true
};
console.log(user);
// -> { name: 'Manuel', german: true };
```

The example is a simple one for ease of understanding, but the uses may become more complex later and give us many ways to write clean and understandable code, especially when it comes to **JSX**.

The latest notable innovation for properties are the so-called **short hand property names**. These save us a lot of unnecessary paperwork. Not only since React has it been known that you come across code like the following:

```javascript
const name = 'Manuel';
const job = 'Developer';
const role = 'Author';

const user = {
  name: name,
  job: job,
  role: role,
};
```

Quite a few unnecessary doublings if you take a close look, don't you think? This is exactly what the **Shorthand Property Name Syntax** in **ES2015** finally does for us. And so it is sufficient to only write the variable if it bears the name of the corresponding object property. In the above case, then:

```javascript
const name = 'Manuel';
const job = 'Developer';
const role = 'Author';

const user = {
  name, job, role
};
```

Yep. since **ES2015**** both spellings actually lead to the identical object! Shorthand syntax can also be combined with conventional syntax without any problems:

```javascript
const name = 'Manuel';
const job = 'Developer';

const user = {
  name,
  job,
  role: 'Author
};
```

## Classes

With **ES2015**, **classes** also found their way into JavaScript. **Classes** are more familiar from object-oriented languages such as Java, but in JavaScript they were not yet explicitly available. Even though it was possible to work object-oriented by using function instances and to define own methods and properties by using the `prototype` property of a function, this was very tedious and writing intensive compared to real object-oriented languages.

This changes with **ES2015**, where for the first time there are classes defined by `class` keywords. This is interesting for us, because although React follows many principles of functional programming \(**Functional Programming**\), it also relies on ES2015 classes in one essential point at the same time;- namely in the creation of components, in this case especially of **Class Components**. Also before the introduction of ES2015 classes it was of course possible to define components in React, so there was a separate `createClass()` method. However, this is no longer part of the React Core and should not be used.

A class consists of a name, \(optional\) can have a **Constructor** that is called when a class instance is created, and can have any number of class methods.

```javascript
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

**output:**

{% hint style="info" %}
Max Mustermann
{% endhint %}

The extension of existing classes with `extends` is also possible:

```javascript
class Customer extends person {}
```

Or just:

```javascript
class MyComponent extends React.Component {}
```

Also a `super()`-function knows the newly introduced **ES2015**-class to call the **Constructor** of its parent class. In the case of React this is always necessary when I define a `constructor` method in my own class. This must then call `super()` and pass its `props` to the constructor of the `React.Component` class:

```javascript
class MyComponent extends React.Component {
  constructor(props) {
    super(props);
  }
}
```

If you don't, `this.props` inside your component would be `undefined` and you wouldn't be able to access the props of your component. However, in most cases the use of a constructor should not be necessary, because React provides its own **Lifecycle methods**, which are preferable to the use of the constructor.

## Rest and Spread Operators and Destructuring

Another significant simplification is the introduction of the so-called rest and spread operators for objects and arrays. Strictly taken it does not concern ES2015 features at all with their use in combination with objects yet, since these are still in the discussion and were not finally taken up yet at all into the ECMAScript specification. This only changes with ES2018. Rest and spread were introduced in ES2015 for the first time for arrays. However, the use of Babel makes it possible to use it with objects even today, and it is usually used extensively in React-based projects.

But what is this now anyway? Let's start with the spread operator.

### Spread Operator

The spread operator ensures that values are "unpacked", so to speak. If one wanted to pass several arguments from an array to a function in ES5, this was usually done via `Function.prototype.apply()`:

```javascript
function sumAll(number1, number2, number3) {
  return number1, number2, number3
}
var myArray = [1, 2, 3];
sumAll.apply(null, myArray);
```

**output:**

{% hint style="info" %}
6
{% endhint %}

With the Spread Operator, which consists of three points \(...\), I can unpack these arguments or just "spread" them:

```javascript
function sumAll(number1, number2, number3) {
  return number1, number2, number3
}
var myArray = [1, 2, 3];
sumAll(...myArray);
```

**output:**

{% hint style="info" %}
6
{% endhint %}

So I don't have to take the detour of "apply" anymore. But this is not only helpful for functional arguments. I can also use it to easily combine two arrays into a single one:

```javascript
const greenFruits = ['kiwi', 'apple', 'pear'];
const redFruits = ['strawberry', 'cherry', 'raspberry'];
const allFruits = [...greenFruits, ...redFruits];
```

**Result:**

{% hint style="info" %}
`['kiwi', 'apple', 'pear', 'strawberry', 'cherry', 'raspberry']`
{% endhint %}

This will create a new array containing all values from both the `greenFruits` and `redFruits` arrays. But not only that: it also creates an entirely new array and not just a reference to the two old ones. This will be very useful in the further course if we are to meet the **readonly** requirement of our props yet. And so you can also use the Spread Operator to make a simple copy of an array:

```javascript
const users = ['Manuel', 'Chris', 'Ben'];
const selectedUsers = [...users];
```

`selectedUsers` in this case is a copy of our `users` array with all its values. If we now change the Users array, this has no effect on our `selectedUsers` array.

The Spread Operator behaves very similarly with objects. Here, instead of the individual values, all the properties of an object are assigned the "enumerable" values. \(countable\), that is, would be displayed roughly when used in a `for(... in ...)` loop.

Here, the Spread Operator is ideal for creating new objects:

```javascript
const globalSettings = { language: 'en-US', timezone: 'Berlin/Germany' };
const userSettings = { mutedUsers: [Manuel] };
const allSettings = {...globalSettings, ...userSettings};
console.log(allSettings);
```

**output:**

```javascript
{
  language: 'en-US',
  timezone: 'Berlin/Germany',
  mutedUsers: ['Manuel'],
}
```

The properties of both objects can be found in the newly created combined `allSettings` object. Here, the **Spread Operator** is not limited to two objects, but can combine any other objects into a single new object. Combination with individual properties is also possible:

```javascript
const settings = {
  ...userSettings,
  showWarnings: true,
}
```

If there are properties with the same name in both objects, the latter object has priority:

```javascript
const globalSettings = { language: 'en-US', timezone: 'Berlin/Germany' };
const userSettings = { language: 'de-DE' };
const allSettings = {...globalSettings, ...userSettings};
console.log(allSettings);
```

**output:**

```text
{
  language: 'de-DE',
  timezone: 'Berlin/Germany',
}
```

The latter `userSettings` object overwrites the `language` property of the same name, which is also located in the `globalSettings` object. The Spread Operator works similar to the new object method `Object.assign()` introduced in ES2015. This is also occasionally used in applications based on ES2015+.

However, there is a notable difference here: it mutates an existing object and does not generate a new object per se, as the object spread variant does. And mutation, in terms of React components and their props, is what we do not want. Nevertheless a short example for the sake of completeness.

#### Combine objects using Object.assign\(\)

`Object.assign()` takes any number of objects and combines them into a single object:

```javascript
const a = { a: 1 };
const b = { b: 2 };
const c = { c: 3 };
console.log(Object.assign(a, b, c));
```

**output:**

```javascript
{a: 1, b: 2, c: 3}
```

So the function returns a new object in which all 3 passed objects have been combined to one single object. But is this really a new object? **Let's output `a`, `b` and `c` in the console afterwards:

```javascript
console.log(a);
console.log(b);
console.log(c);
```

**output:**

```javascript
{a: 1, b: 2, c: 3}
{b: 2}
{c: 3}
```

So we find out: `Object.assign()` didn't really create a completely new object from the 3 passed objects but only added the properties of the second and third object to the first passed object. And that is, in relation to **Pure Functions** and **Immutable Objects**, extremely bad and to be avoided in any case!

But here is a simple trick to combine objects with `Object.assign()` and create a new object at the same time. For this you pass an empty object literal `{}` to the function as the first argument:

```text
Object.assign({}, a, b, c);
```

... and the properties of our objects `a`, `b` and `c` are transferred to the newly created `{}` object, the existing objects `a`, `b` and `c` remain untouched!

### Destructuring Assignment / destructuring assignment

Before I come to the **Rest Operator**, which is logically very closely connected with the **Spread Operator** and is usually mentioned in the same breath, I would like to discuss the **Destructuring Assignment** \(short: **Destructuring**\) or the **Destructuring Assignment** \(short: **Destructucturing**\), as the nice term is called in German. I will stick here, as so often with the English term, because I have rarely read the German term even in German-language texts so far.

Using **Destructuring** it is possible to extract individual elements from objects or arrays and assign them to variables. Another **clear** syntax extension that ES2015 gave us here.

#### Destructuring arrays

Let's imagine, we want to write the winner of the gold, silver and bronze medals from an ordered array with the Olympic participants in the 100m run into our own variable. In the conventional \(ES5\) way this worked as follows:

```javascript
const athletes = [
  Usain Bolt,
  Andre De Grasse,
  Christophe Lemaitre,
  Adam Gemili,
  Churandy Martina,
  LaShawn Merritt,
  Alonso Edward,
  Ramil Guliyev,
];

const gold = athletes[0];
const silver = athletes[1];
const bronze = athletes[2];
```

Thanks to **Destructuring** we can shorten this to a single statement:

```javascript
const [gold, silver, bronze] = athletes;
```

The values of the array elements `0`, `1` and `2` are then located one after the other in the variables `gold`, `silver` and `bronze`, as in the first example, but with much less paperwork!

This works wherever we work with an array on the right \(behind the `=` sign\) of an assignment, even if we get it as a `return` value from a function:

```javascript
const getAllAthletes = () => {
  return [
    Usain Bolt,
    Andre De Grasse,
    Christophe Lemaitre,
    Adam Gemili,
    Churandy Martina,
    LaShawn Merritt,
    Alonso Edward,
    Ramil Guliyev,
  ] 
}

const [gold, silver, bronze] = getAllAthletes();
```

The Arrow Function returns an array with all athletes, so we can use the destructuring directly at the call and don't have to save the `return` value in a temporary variable first.

If we want to omit individual elements of the array in this way, this is literally possible by omitting the corresponding value:

```javascript
const [, silver, bronze] = athletes;
```

Here we would do without declaring a variable `gold` and store only the winners of the silver and bronze medals in corresponding variables.

But **Array Destructuring** can not only be used for the obvious variable assignment with `let` or `const`. Even with less obvious assignments, such as passing function arguments in the form of an array.

```javascript
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

It's easier:

```javascript
const logWinners = ([gold, silver, bronze]) => {
  console.log(
    'Winners of Gold, Silver and Bronze are', 
    gold, 
    silver, 
    bronze
  );
}
```

Here we put the array into our `logWinners()` function and instead of declaring one variable per line for each medal winner, we simply use the destructuring method from above again.

#### Destructuring of objects

The principle of **Destructuring** is not limited to arrays. Objects can also be assigned in this way to variables that by default match the name of a property.

The spelling is similar to that of arrays, with the difference that we do not assign the values according to their position in the object, but according to their property name. In addition, we place the assignment in the curly brackets typical for objects instead of in square brackets.

```javascript
const user = {
  firstName: 'Manuel',
  lastName: 'Bieh',
  job: 'JavaScript Developer',
  image: 'manuel.jpg',
};
const { firstName } = user;
```

The variable `firstName` now contains the value from `user.firstName`!

Object Destructuring is one of the most common features found in most React components. It allows us to write individual props into variables and use them in the appropriate places in JSX in an uncomplicated way.

Let us take the following Stateless Functional Component as an example:

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

The constant repetition of 'props' in front of each property makes the readability of the component unnecessarily difficult. Here we can use Object Destructuring to declare one variable for each property of our `props`.

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

This makes our component appear much tidier and more legible. But it is even easier. As with arrays, it is also possible to destruct objects directly when passing them as function arguments. Instead of the `props` argument we use the **Destructuring Assignment** directly:

```javascript
const UserPersona = ({ firstName, lastName, image, job }) => (
  <div>
    <img src={image} alt="User Image" />
    {firstName} {lastName}<br />
    <strong>{job}</strong>
  </div>
);
```

As a bonus we even use the direct return from the function without curly braces and explicit `return` statement from the chapter about Arrow Functions, since we now have an expression with our **JSX** reduced to 5 lines, which we can return directly from the **Arrow Function**.

While working with React you will always find such syntax in **SFCs**, also in **Class Components** you often find a similar destructuring assignment in the form at the beginning of the `render()` method of a component:

```javascript
render() {
  const { firstName, lastName, image, job } = this.props;
  // additional code
}
```

Of course it is up to you if you want to do this afterwards or if you just want to access `this.props.firstName` directly within the function. However, this pattern has now developed into a kind of best practice and has been used in most projects as it makes the code more readable and easier to understand in most cases.

**Renaming of properties in Destructuring**

Sometimes it is necessary to rename properties, either because there are already variables with the same name or because the property would not be a valid variable name. All this is conceivable and possible. And ES2015 also offers us a solution.

```javascript
const passenger = {
  name: Manuel Bieh,
  class: 'economy',
}
```

The `passenger` object above contains the class property, which is valid as a name for a property, but not as a name for a variable. A direct destructuring would not be possible here and would lead to an error:

```javascript
const { name, class } = passenger;
```

{% hint style="danger" %}
Uncaught SyntaxError: Unexpected token }
{% endhint %}

To rename the name of the variable here, the new name must be passed to the property separated by a colon `:`. A correct **Destructuring Assignment** in this case would be something like the following:

```javascript
const { name, class: ticketClass } = passenger;
```

Here we write the value of the `class` property into a `ticketClass` variable, which unlike `class` is a valid name for a variable. The passenger's name usually ends up in a variable called `name`.

**Standard values assigned during destructuring**

It is also possible to assign standard values for **Destructuring**! If a property is not defined in the destructured object, the default is used instead. Similar to renaming, the respective property is prefixed as usual, but followed by an equal sign and the corresponding default value:

```javascript
const { name = 'Unknown passenger' } = passenger;
```

The value of `name` would now be `Unknown passenger` if there is no `name` property in the `passenger` object or if its value is `undefined`. If it exists, but is empty \(e.g. an empty string or `null`\), the default value **not** is used instead!

**Combination of renaming and default values**

Now it's going crazy because that's possible too. Renaming properties to variable names while using default values. However, the syntax for this is something where you certainly have to look a moment longer at the first encounter. We stick again to our `passenger` object from the examples before. The requirement now is to assign the `name` property to a variable named `passengerName`, which should have the value `Unknown Passenger` if no name exists. Furthermore we would like to rename `class` to `ticketClass` and classify the passenger in `Economy` if there is no `class` property in the corresponding object.

```javascript
const {
  name: passengerName = 'Unknown passenger',
  class: ticketClass = 'economy',
= passenger;
```

Here the variables `passengerName` and `ticketClass` have the values `Unknown passenger` and `economy` if they do not exist in the destructured object. But beware: The object itself must not be zero, otherwise the JavaScript interpreter will throw an unsightly error at us:

```javascript
const {
  name: passengerName = 'Unknown passenger',
  class: ticketClass = 'economy',
= zero;
```

{% hint style="danger" %}
Uncaught TypeError: Cannot destructure property \`name\` of 'undefined' or 'null'.
{% endhint %}

There is an unclean but often practical trick to ensure that the object itself is not `null` or `undefined`. For this we use the **Logical OR Operator** and use an empty object as fallback if our actual object is `null` or `undefined`:

```javascript
const {
  name: passengerName = 'Unknown passenger',
  class: ticketClass = 'economy',
} = passenger || {};
```

With the attached `||| {}` we say: if the `passenger` object is **falsy**, use an empty object instead. The probably "cleaner" variant would be to check in advance if `passenger` is really an object and only then execute the destructuring. The variant with the **Logical OR** fallback is however nicely short and should suffice in many cases.

By the way, **Destructuring** can also be used together with the **Spread Operator**:

```javascript
const globalSettings = { language: 'en-US' };
const userSettings = { timezone: 'Berlin/Germany' };
const { language, timezone } = { ...globalSettings, ...userSettings };
```

Here the **Spread Operator** is first resolved, i.e. an object with all properties is created from the two objects `globalSettings` and `userSettings` and then assigned to the corresponding variables via **Destructuring Assignment**.

### Rest Operator

The Rest Operator is used to take care of the remaining elements from a **Destructuring** and in **Function Arguments**. Hence the name: the operator takes care of the remaining **"rest "**. Like the **Spread Operator**, the **Rest Operator** is introduced with three points `...`, but not on the **right** side of an assignment, but on the **left** side. Unlike the Spread Operator, there can only be **one** Rest Operator per expression!

Let's first look at the **Rest Operator** for function arguments. Let's say we want to write a function that receives any number of arguments. Here, of course, we would like to be able to access all these arguments, regardless of whether they are 2, 5 or 25. In ES5 standard functions there is the keyword `arguments`, by means of which an array of all passed function arguments can be accessed within the function:

```javascript
function Example() {
  console.log(arguments);
}
Example(1, 2, 3, 4, 5);
```

**output:**

{% hint style="info" %}
`Arguments(5) [1, 2, 3, 4, 5, callee: ƒ]``
{% endhint %}

**Arrow Functions** no longer offer this possibility and instead throw an error:

```javascript
const Example = () => {
  console.log(arguments);
}
Example(1, 2, 3, 4, 5);
```

**output:**

{% hint style="danger" %}
Uncaught ReferenceError: arguments is not defined
{% endhint %}

This is where the **Rest Operator** comes into play for the first time. This writes all passed function arguments, which we have not already written into named variables, into another variable with an arbitrary name:

```javascript
const Example = (...rest) => {
  console.log(rest);
}
Example(1, 2, 3, 4, 5);
```

**output:**

{% hint style="info" %}
`[1, 2, 3, 4, 5]`
{% endhint %}

This works not only as a single function argument, but also if we have defined previously named parameters. Here the **Rest Operator** literally takes care of the last remaining **Rest:**

```javascript
const Example = (first, second, third, ...rest) => {
  console.log('first:', first);
  console.log('second:', second);
  console.log('third:', third);
  console.log('rest:', rest);
}
Example(1, 2, 3, 4, 5);
```

**output:**

{% hint style="info" %}
`first: 1    
second: 2    
third: 3    
rest: [4, 5]`
{% endhint %}

The **Rest Operator** collects the remaining elements from a **Destructuring** and stores them in a variable with the name specified after the three points. This does not have to be called `rest` like in the example above but can take any valid JavaScript variable name.

This does not only work with functions but also with **Array Destructuring**:

```javascript
const athletes = [
  Usain Bolt,
  Andre De Grasse,
  Christophe Lemaitre,
  Adam Gemili,
  Churandy Martina,
  LaShawn Merritt,
  Alonso Edward,
  Ramil Guliyev,
];
const [gold, silver, bronze, ...competitors] = athletes;
console.log(gold);
console.log(silver);
console.log(bronze);
console.log(competitors);
```

**output:**

{% hint style="info" %}
```javascript
Usain Bolt.
Andre De Grasse.    
Christophe Lemaitre.  
[  
  Adam Gemili,
  Churandy Martina,
  LaShawn Merritt,
  Alonso Edward,
  Ramil Guliyev.  
]
```
{% endhint %}

...as with **Object Destructuring:**

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

**output:**

{% hint style="info" %}
`Manuel`  
`Bieh`  
`{ job: 'JavaScript Developer', hair: 'Brown' }`
{% endhint %}

All the values that were not explicitly written to a variable during a **Destructuring Assignment,** can then be retrieved from the variable declared as **Rest**.

## Template Strings

**Template Strings** in ES2015 are a "third notation" for strings in JavaScript. Until now, strings could be placed either in single quotation marks \(`'Example'`\) or in double quotation marks \(`"Example"`\). Now there is also the possibility to set them in Backticks \(`Example`\).

**Template Strings** can occur in two variants. As ordinary **Template Strings**, which can contain JavaScript expressions, as well as in extended form as so-called **Tagged Template Strings**.

**Tagged Template Strings** are a much more powerful form of **Template Strings**. They can be used to modify the output of **Template Strings** using a special function. This is less important when working with React.

If you wanted to mix them with JavaScript expressions or values, in ES5 you would usually use simple **String Concatenation:**

```javascript
var age = 7;
var text = 'My daughter is ' + age + ' years old';
```

```javascript
var firstName = 'Manuel';
var lastName = 'Bieh';
var fullName = firstName + ' ' + lastName;
```

With **Template Strings** a variant of String was introduced, which itself can contain **JavaScript expressions**. These are set within a **Template String** into a string in the form `${ }`. To stay with the examples above:

```javascript
const age = 7;
const text = `My daughter is ${age} years old`;
```

```javascript
const firstName = 'Manuel';
const lastName = 'Bieh';
const fullName = `${firstName} ${lastName}`;
```

All JavaScript expressions can be used within the curly brackets. So also function calls:

```javascript
console.log(`Today's date is ${new Date().toISOString()}`);
console.log(`${firstName.toUpperCase()} ${lastName.toUpperCase()}`);
```

## Promises and async/await

Promises \ are not a fundamentally new concept in JavaScript, but in ES2015 they have been introduced into the standard for the first time and can be used natively without another library \(e.g. q, Bluebird, rsvp.js, ...\). Promises roughly allow to _linearize_ the asynchronous development by callbacks. A Promise gets a **Executor function** passed, which in turn get the two arguments `resolve` and `reject` passed, and can assume one of a total of three different states: as initial value this state is `pending` and depending on whether an operation was successful or erroneous, i.e. the Executor function has executed the first \(`resolve`\) or the second \(`reject`\) argument, this state changes to `fulfilled` or `rejected`. You can then react to both final states using the methods `.then()` and `.catch()`. If `resolve` is called, the `then()` part is executed; if `reject` is called, **all** `then()` calls are skipped and the `catch()`\` part is executed instead.

An executor function **must** execute one of the two passed methods, otherwise the promise remains permanently _unfulfilled_, which can lead to incorrect behavior and in certain cases even to memory leaks within an application.

To demonstrate the difference between Promises and Callbacks, let's take a look at the following fictional example:

```javascript
const errorHandler = (err) => {
  console.error('An error occured:', err.message);
};

getUser(id, (user) => {
  &lt;font color="#ffff00"&gt;-==- proudly presents
    friends[0].getSettings((settings) => {
      if (settings.notifications === true) {
        email.send('You are my first friend!', (status) => {
          if (status === 200) {
            alert('User has been notified via email!');
          }
        errorHandler);
      }
    errorHandler);
  }, errorHandler)
}, errorHandler)
```

We use the asynchronous `getUser()` function to get a user for a corresponding `id`. From this user we get a list of all his friends using the asynchronous `getFriends()` method. From the first friend \(`friends[0]`\) we retrieve the user settings using the asynchronous `getSettings()` method. If the user allows e-mail notifications, we send him an e-mail and respond, again asynchronously, to the response of the mail server.

The example is a relatively simple one, there is no explicit error handling and there are no notable exceptions. Nevertheless, the code in the example is already **6 levels** deeply nested. Working with callbacks can therefore quickly become confusing, especially if further callback functions are executed within a callback function, as in our example. Thus it comes fast to the often also as **Pyramid of Doom** called nesting of Callbacks.

Now let's rewrite the example and assume that our fictitious API methods each return a promise:

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

After each step we react with `then()` to the returned promise, achieve the same result as before with the callback version, but at the deepest point we only have a nesting which is 2 levels deep.

It is relatively simple to rewrite existing callback-based code in Promises. I would like to demonstrate this briefly using the Geolocation API and specifically its `getCurrentPosition()` method. Who doesn't know it: the method exists on the `navigator.geolocation` object, opens a notification in the browser and asks the user for permission to locate him. It expects two callbacks as arguments: the first, the success callback, gets an object with the position of the user, if the user agrees to the location. The second, the error callback, gets an error object if the user either did not agree to a location or a location is not possible for other reasons.

```javascript
navigator.geolocation.getCurrentPosition((position) => {
  console.log(`User position is at ${position.coords.latitude}, ${position.coords.longitude}`);
}, () => {
  console.log('Unable to locate user');
});
```

And so the callback is transformed into a promise:

```javascript
const getCurrentPositionPromise = () => {
  return new Promise((resolve, reject) => {
    navigator.geolocation.getCurrentPosition(resolve, reject);
  });
};
```

Yep. That's really it. Instead of using the callback syntax, we can now access the position of the user via the following call:

```javascript
getCurrentPositionPromise()
.then((position) => {
  console.log(`User position is at ${position.coords.latitude}, ${position.coords.longitude}`);
})
.catch(() => {
  console.log('Unable to locate user');
});
```

Some newer JavaScript APIs in the browser have already been implemented following this approach. If you want to know more about Promises and how it works, I recommend to read [the corresponding article at the MDN Web Docs](https://developer.mozilla.org/de/docs/Web/JavaScript/Reference/Global_Objects/Promise). The Promises statement should only be introductory to prepare for a much more exciting new feature, namely:

### Asynchronous functions with `async` and `await`

Asynchronous functions with the keywords `async` and `await` can perhaps be seen a bit as the "next evolutionary step" in asynchronous development after callbacks and promises. You have moved into the JavaScript specification in ES2016. Under the hood, they still use Promises, but make them largely invisible. They allow us to write asynchronous code in such a way that it looks almost like synchronous code. No more callbacks and no more `then()` or `catch()`.

For this purpose an asynchronous function call is preceded by the keyword `await`. To use `await`, the surrounding function must be preceded by the second keyword `async` to tell the JavaScript interpreter that it is such an asynchronous function. When using `await` without marking a function as `async` an exception occurs.

So let's take another look at the example of our user who wants to send an e-mail to his first friend. This time with asynchronous functions:

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

Asynchronous functions with `async` and `await` are for me personally one of the most noteworthy changes of JavaScript in the past years, because they make working with asynchronous data almost child's play, compared to complex and confusing callbacks. And even promises, which were already a great relief compared to conventional callbacks, appear almost complex in direct comparison with asynchronous functions.

## Import syntax and Javascript modules

Modules in JavaScript are such a thing. Officially they did not exist until now, but there were always attempts to introduce modules in JavaScript. If you have been working with Node.js for some time, you might know **AMD** \(Asynchronous Module Definition\), if you have worked with Node.js, you should also know CommonJS modules \(i.e. `module.exports` and `require('./myModule')`\). For a long time, there were extensive discussions about which module standard to agree on, what the syntax looks like, and finally what the implementation on the interpreter side looks like. The choice fell on modules that communicate with each other via `import` and `export` keywords.

Babel then went ahead and implemented a solution based on the then official specification. This implementation was then changed in between times, because there were updates to the appropriate standard, then came Webpack and implemented its own mechanism for resolving and loading JavaScript modules, which is based on the now adopted standard. Just like TypeScript.

Meanwhile, after just **10** years, there is agreement on the specification and the implementation of the JavaScript engines is in full swing. Sounds complicated, it has been in between, but now there is consensus and for us developers there is gradually clarity. However, there are still some pitfalls that will force us to rely on Webpack, Babel or TypeScript in the future \(or rather: should\) to work comfortably with modules. More about that later.

So much for history. So how do imports work now and what are modules anyway?

### Modules in JavaScript

The goal of modules is to encapsulate scopes in JavaScript on a **per module level**. A module in this sense is actually a single **File**. Unless you explicitly limit it by creating a new scope, e.g. by including it in a **IIFE** \(_Immediately Invoked Function Expression_\), every function or variable defined in JavaScript is globally available. Modules counteract this by making all code available only **within the module**. This avoids complications, e.g. when two libraries use the same variable, and also creates reusable code in a simple way, without having to worry about it overwriting existing variables or functions elsewhere.

Modules can **export** the functions, classes or variables defined in them, other modules can then import these exports as needed. For the export of functions and variables there is an `export` keyword, to import these exports later elsewhere there is, you think, the corresponding `import` keyword. Exports can take two forms, one being that of a **Named Export** \ and the other being that of a **Default Export** \.

#### Named Exports

Let's say we have a module `calc.mjs` that provides us with all kinds of functions to do various kinds of calculations. The module could, for example, have the following content:

```javascript
export const double = (number) => number * 2;
export const square = (number) => number * number;
export const divideBy = (number, divisor) => number / divisor;
export const divideBy5 = (number) => divideBy(number, 5);
```

So here we announce a **export**, define a variable directly afterwards, assign it an Arrow Function, get a parameter \(or two\) and directly return the result of the calculation. Alternatively, this can also be done in two separate steps:

```javascript
const double = (number) => number * 2;
export double;
```

Elsewhere within our application we can now import these functions using the `import` keyword **import**. We use `import` followed by the exports we want to import in curly braces, followed by `from` and the path to the module.

```javascript
import { double, square, divideBy5 } from './calc.mjs';

const value = 5;
console.log(double(value)); // 10
console.log(square(value)); // 25
console.log(divideBy5(value)); // 1
```

A file can theoretically **have an unlimited number of named exports**, but they must differ in name and an already exported name **must not be exported again.

#### Default Export

In addition to the \(Plural\) so-called **Named Exports** from the above example, there is also the \(Singular\) `Default Export`, a special form of export, which may only occur **once** within each module and which is marked with the keyword `default`. If a variable or function is marked as `default`, it is possible to import this export without braces. The **Default Export** can be used, for example, to bundle several named exports so that you do not have to import them individually.

```javascript
export const double = (number) => number * 2;
export const square = (number) => number * number;
export const divideBy = (number, divisor) => number / divisor;
export const divideBy5 = (number) => divideBy(number, 5);

export default {
   double, square, divideBy, divideBy5
}
```

Instead, our application would only have to import the module itself, i.e. assign its **Default Export** and a variable to it:

```javascript
import Calc from './calc.mjs';

console.log(Calc.double(value)); // 10
console.log(Calc.square(value)); // 25
console.log(Calc.divideBy5(value)); // 1
```

In many cases it makes sense that a module also has a default export. Especially with component-based libraries like React or Vue.js it is common to have only one export per module, this should be the **Default Export**. Even if syntactically this would not be absolutely necessary, it has become the de facto standard when working with React.

```javascript
export default class MyComponent extends React.Component {
  // ...
}
```

### Pitfalls: Browser vs. Node.js

If you were attentive and paid attention, you might have noticed that we import from a file named `calc.mjs` above, not `calc.js` \(`.mjs` instead of `.js`\). This is the convention agreed upon when using JavaScript modules in Node.js during the lengthy standardization process described above.

If you want to write universal JavaScript, i.e. JavaScript that can be executed server-side with Node.js as well as client-side in the browser, and if you want to do this without inserting a compiler intermediate step through e.g. Babel, Webpack or TypeScript, **you have to** use the `.mjs' extension for your files.

The loading of modules works in Node.js a bit different than in the browser. While the browser doesn't care which file extension a module has \(as long as the server sends the content-Type `text/javascript` with it\), Node.js needs the `.mjs` file extension to identify JavaScript modules as such.

To use JavaScript modules in the browser, the `type` attribute on the `<script></script>` element must have the value `module`. So:

```markup
<script src="./myApp.mjs" type="module"></script>
```

Browsers that support `type="module"` also simultaneously support and ignore the `nomodule` attribute for delivering fallbacks for browsers without module support.

```markup
<script src="./myApp.mjs" type="module"></script>
<script src="./myApp.bundle.js" nomodule></script>
```

A browser with module support would load `myApp.mjs` here, while all others would load a bundled \(e.g. by Webpack\) `myApp.bundle.js` instead.

But that's not all, because Node.js has its own mechanism for finding and loading files. For example, modules that do not have a relative path, i.e. do not begin with `./` or `../`, are searched for in `node_modules` or `node_libraries`. By default, Node.js also loads an index.js in it if Node.js finds a folder with the specified name.

```javascript
import MyModule from 'myModule';
```

Node.js would search for this import in the folder `./node_modules/myModule`, load an `index.js` there or alternatively search for the correct file in the `main` field of the `package.json`. The browser, on the other hand, cannot try different paths to find the right file, as this would cause an expensive network request each time and possibly many 404 responses.

In addition, **Import Specifier**, which is the part behind the `from`, the module you want to import from, is protected in the browser and must consist of a valid URL or a relative path.

Imports like the following are currently not possible in the browser:

```javascript
import React from 'react';
```

The **Package Name Maps**, a proposal, thus a suggestion for upcoming ECMAScript versions, which is currently still at the very beginning of the discussion, are supposed to provide a remedy here later. That's why we can't avoid to use a module bundler like Webpack in the foreseeable future to work comfortably with ES modules if we want to use JavaScript modules both server-side and client-side at the same time.

## Conclusion

ES2015 and later versions offer a lot of useful new features that were not available in JavaScript before. Many of them are almost indispensable when working with React. The most important innovations include those described here:

* Variable declarations with `let` and `const`
* **Arrow Functions** to create functions that do not bind their own `this`.
* **Classes**. Make many things easier and are the basis of **React Class Components**.
* The **Rest and Spread Operators** that greatly simplify reading and writing data in arrays and objects
* **Template Strings** to make working with JavaScript expressions in strings easier
* **Promises** and **Asynchronous functions** using `async`/`await` to make working with asynchronous data much easier
* **Import** and **Export** for encapsulating reusable JavaScript at module level