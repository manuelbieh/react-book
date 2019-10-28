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

 

