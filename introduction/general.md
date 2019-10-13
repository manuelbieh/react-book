# General

## What React is and what it is not

Let's cite the React documentation at this point that have summarised it nicely:

> \[React is\] a library for building user interfaces.

Even if this explanation might seem a little short, you can deduce a few key points from it in order to understand React better. In the first instance, React is just a _library,_ not a framework. It does not boast with an abundance of in-built functions with which you can complex web applications with ease. The second part of the sentence is equally important: _for building user interfaces._

Strictly speaking, React is just a **library** that makes it easy to develop **user interfaces**. There are no services or methods to make API calls or built-in models or ORM. React is only concerned with user interfaces. You could say, it is the view layer in your application. That's it! In this context, you occasionally read that React can be thought of the "V" in **MVC** \(Model-View-Controller\) or **MVVM** \(Model-View-ViewModel\). This is a pretty good explanation in my eyes.

React offers a declarative way to model state, state here referring to the state of the user interface itself. Put simply, this means that you declare in your code how your user interface should look given each state of the component. A simple example to illustrate this: If a user is logged in, show the dashboard. If he is not, show the login form.

The logic itself is found in the JavaScript part of the application \(where it should live anyway\) and not in the templates, as opposed to many other web frameworks who mix the two. This sounds complicated at first, but it will become clearer soon what this means.

React is **component-based**. This means you write **encapsulated, functional components** that can be composed together at will and reused at your leisure. Extending or inheriting components is possible in theory, but rather uncommon in the world of React. Rather than working with _inheritance_, React encourages c_omposition_, the act of combining multiple components into a "whole".

Does this mean that you cannot build complex web applications with React? No. Absolutely not. React boasts a large and very active and also highly qualitative ecosystem of libraries that in turn is based on React, extends it or complements it in such a way that it does not need to hide behind large scale frameworks like Ember or Angular. Quite the opposite actually. Once you have entered the world of the React ecosystem and gained a first impression, you quickly find a number of really good tools and libraries with which you can build professionally, super individual and highly complex applications.





