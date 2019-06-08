# General

Even if the explanation is very short, you can derive from it all the essential things that are important for working with React and to understand what it is all about. React is first of all just a _Library_, not a complete framework with countless functions, with which you can develop complex web applications without further dependencies. And here we come to the second part of the sentence: _for building user interfaces_.

So React is just a **Library** that makes it easy for you to **develop user interfaces**. No services or methods to make API calls, no built-in models or ORM. User interfaces only. So to speak only the view layer of your application. That?s it! In this context, one occasionally reads that React represents the "V" in **MVC** \(_Model-View-Controller_\) or **MVVM** \(_Model-View-ViewModel_\). That's what I think it's all about.

React offers a **declarative** way to describe the **state** of a user interface. To put it simply, this means that you basically explicitly describe with your code what your user interface should look like, depending on the **state** in which a component is located. Simple example to illustrate this principle: if a user is logged in, show the dashboard; if he is not, show the login form.

The logic itself is located completely in the JavaScript part of the application \(where it should always belong\) and not in the templates themselves, as is the rule with most other Web frameworks. Sounds complicated at first, but as the process progresses it becomes clearer and clearer what it actually means.

React works **component-based**, i.e. it develops **encapsulated, functional components**, which can be composed \(_composed_\) and reused as desired. Extension or inheritance of components is theoretically possible, but very unusual in the React world. The _Composition-\_approach is also propagated here by the official side, in which several components are combined to form an "overall picture" instead of working with \_Inheritance_  \(i.e. inheritance\).

So does that mean I can't develop complex web applications with React? No. Absolutely not. React has a very large, very active and for the most part very high quality ecosystem of libraries, which in turn are based on React, extend or add to it and make it a powerful tool that doesn't have to hide behind large frameworks like Ember or Angular. On the contrary. Once you've immersed yourself in the world of the React ecosystem and gained an overview, you've quickly found a number of really good tools and libraries with which you can develop professional, super individual and highly complex applications.

Particularly shortly after React picked up speed, the question was often asked whether the days of jQuery are numbered - whether everything can or should be developed with React or when the use of React makes sense or maybe not at all.

React is, as we have already clarified, first of all a _Library for the creation of user interfaces_. User interfaces always mean interaction. And interaction inevitably goes hand in hand with state management in most cases. I press a button and a dropdown opens. So I change the state from _closed_ to _open_. I enter data into an input field and get an indication whether my entered data is valid. If they are not, the state of the input field changes from _valid_ to _invalid_. And this is where React comes in. If I don't have interaction or "changing data" on my site because I'm developing a pure static image page for a company, I _probably_ don't need a react.

Incorrectly implemented, React can even do harm here, because on an image website often the content is in the foreground and if you don't already render your React components on the server side, most search engines can't do much with the page at first. Fortunately, React makes it very easy for us to render our components on the server side, so this is another problem that is usually easy to fix.

If, on the other hand, I have a lot of interaction and an interface that is often updated, using React will probably save a lot of time and nerves. The basic rule of thumb here is: the more interaction takes place in a website or web application and the more complex it is, the more worthwhile it is to use React. The most handy example here is **Single Page Applications** \(_SPA_\), where the application is called and initialized only once in the browser and any further interaction and communication with the server takes place via XHR \(most commonly known as "AJAX requests"\).

Imperative in this case means telling the browser what to do, whereas _declarative_ code, as written with React, only describes the desired end result depending on the current state. One of the core principles of React. To stay with the example from above: instead of saying "I am logged in now, dear browser, please hide the login form and show me the dashboard", I define two views: So, dear browser, my interface should look like this when I'm logged in \(Dashboard view\) and like this when I'm not \(Login view\). React then decides which of the views is displayed based on the state of the component.

React was originally developed by or on **Facebook** and later, as early as 2013, made available to the public as open source under the BSD license, which was changed to an MIT license after some protests. And so a very large part of Facebook is based on React. Meanwhile there are even more than **50,000** of their own components in use - which is nice in that Facebook naturally has a great interest in permanent further development and you don't have to fear that your application has been developed on the basis of a technology that is suddenly no longer being further developed.

The React Core developers are doing a very good job of involving the community in decisions at an early stage and allowing them to participate in discussions. There is also a GitHub repository with [React RFCs](https://github.com/reactjs/rfcs) \(_"Request for Comments"_\), which allows planned changes to be discussed at an early stage and allows the React team to make their own suggestions.

**Breaking Changes** follow a fixed **Deprecation Schema** and thus methods, properties and functions, which are planned to be removed, are provided with meaningful **Deprecation Warnings** for some time and even tools are provided, with which old code can be adapted as automatically as possible \([React-Codemod](https://github.com/reactjs/react-codemod)\). React adheres strictly to Semver conventions.

This means that only new _Major-Releases_ contain \(`16.x.x` on `17.x.x`\) Breaking Changes, _Minor-Releases_ contain \(e.g. `16.6.x` on `16.7.x`\) contain new features or get deprecation warnings that prepare the developer for upcoming major releases, while _Patch releases_ \(e.g. `16.8.0` on `16.8.1`\) contain only bug fixes.

Before major or minor releases are released, there are also alpha, beta and RC \(_"Release Candidate"_\) versions on a regular basis, with which you can take a look at upcoming features in advance. However, these should be taken with caution, as the functionality of new features may change before the final release.

This is surely due to the fact that Facebook also uses a lot of React components and you can't just make profound changes without causing problems. The thoughts and justifications of the developers can be followed in detail in the GitHub Issue Tracker, all important changes are summarized in so called[ Umbrella-Tickets](https://github.com/facebook/react/issues?utf8=✓&q=is%3Aissue%20is%3Aopen%20umbrella).

## What is React and what isn't?

Let's quote the React documentation in the first place, because it sums it up very succinctly:

\[React is\] a library for building user interfaces.

Even if the explanation is very short, you can derive from it all the essential things that are important for working with React and for understanding what it is all about. React is first of all just a _Library_, not a complete framework with countless functions, with which you can develop complex web applications without further dependencies. And here we come to the second part of the sentence: _for building user interfaces_.

So React is just a **Library** that makes it easy for you to **develop user interfaces**. No services or methods to make API calls, no built-in models or ORM. User interfaces only. So to speak only the view layer of your application. That?s it! In this context, one occasionally reads that React represents the "V" in **MVC** \(_Model-View-Controller_\) or **MVVM** \(_Model-View-ViewModel_\). That's what I think it's all about.

React offers a **declarative** way to describe the **state** of a user interface. To put it simply, this means that your code basically describes explicitly how your user interface should look like, depending on the **state** in which a component is located. Simple example to illustrate this principle: if a user is logged in, show the dashboard, if he is not, show the login form.

The logic itself is located completely in the JavaScript part of the application \(where it should always belong\) and not in the templates themselves, as is the rule with most other Web frameworks. Sounds complicated at first, but in the course of time it becomes clearer and clearer what it actually means.

React works **component-based**, i.e. you develop **encapsulated, functional components** that can be composed \(_composed_\) and reused as you like. Extension or inheritance of components is theoretically possible, but very unusual in the React world. Here, the _Composition_ approach is also propagated by the official side, in which several components are combined to form an "overall picture" instead of working with _Inheritance_, i.e. inheritance.

So does that mean I can't develop complex web applications with React? No. Absolutely not. React has a very large, very active and for the most part very high quality ecosystem of libraries, which in turn are based on React, extend or add to it and make it a powerful tool that doesn't have to hide behind large frameworks like Ember or Angular. On the contrary. Once you've immersed yourself in the world of the React ecosystem and gained an overview, you've quickly found a number of really good tools and libraries with which you can develop professional, super individual and highly complex applications.

## When should I use React and when not?

Especially shortly after React started to run, the question was often asked if the days of jQuery are counted, if everything can or should be developed with React or when the use of React makes sense or maybe not at all.

React is, as we have already clarified, a _Library for the creation of user interfaces_. User interfaces always mean interaction. And interaction inevitably goes hand in hand with state management in most cases. I press a button and a dropdown opens. So I change the state from _closed_ to _open_. I enter data into an input field and get an indication if my entered data is valid. If they are not, the state of the input field changes from _valid_ to _invalid_. And this is where React comes in. If I have no interaction or "changing data" on my site because I'm developing a static image site for a company, I probably don't need a react.

Incorrectly implemented, React can even do harm here, because on an image website often the content is in the foreground and if you don't already render your React components on the server side, most search engines can't do much with the page at first. Fortunately React makes it very easy for us to render our components on the server side, so this is still a problem that is usually easy to fix.

If, on the other hand, I have a lot of interaction and an interface that updates itself often, the use of React will probably save a lot of time and nerves. The basic rule of thumb here is: the more interaction takes place in a website or web application and the more complex it is, the more worthwhile it is to use React. The most handy example here is **Single Page Applications** \(_SPA_\), where the application is called and initialized only once in the browser and any further interaction and communication with the server takes place via XHR \(most commonly known as "AJAX requests"\).

I recently experienced it myself in a project, that I had to develop a registration form, which seemed quite simple to me and I started without React. In the course of the development it turned out that for the purpose of better usability more and more \(background\)interaction became necessary. For example, an automatic live validation of form data was to be added afterwards and the login process divided into 2 steps, so that I quickly resorted to React because the manual state management and the **imperative** change of the user interface became too cumbersome for me.

Imperative means in this case that I tell the browser what to do, whereas _declarative_ code, as written with React, only describes the desired end result depending on the current state. One of the core principles of React. To stay with the example from above: instead of saying "I am logged in now, dear browser, please hide the login form and show me the dashboard", I define two views: So, dear browser, my interface should look like this when I'm logged in \(Dashboard view\) and like this when I'm not \(Login view\). Which of the views is displayed is then determined by React based on the state of the component.

## Where does React come from?

React was originally developed by or at **Facebook** and later, already in 2013, made available to the public as open source under the BSD license, which was changed to an MIT license after some protests. And so a very large part of Facebook is based on React. In the meantime, more than **50,000** of the company's own components are even said to be in use there. Which is nice in the sense that Facebook naturally has a great interest in permanent further development and you don't have to fear that you have developed your application on the basis of a technology that is suddenly no longer being further developed.

The React Core developers are doing a very good job of involving the community in decisions at an early stage and allowing them to participate in discussions. There is also a GitHub repository with [React RFCs](https://github.com/reactjs/rfcs) \(_"Request for Comments"_\), which allows planned changes to be discussed at an early stage and allows the React team to make their own suggestions.

**Breaking Changes** follow a fixed **Deprecation Schema** and thus methods, properties and functions whose removal is planned are provided with meaningful **Deprecation Warnings** for some time and even tools are provided with which old code can be adapted as automatically as possible \([React-Codemod](https://github.com/reactjs/react-codemod)\). React adheres strictly to Semver conventions.

This means that only new _Major-Releases_ \(`16.x.x` on `17.x.x`\) contain Breaking Changes, _Minor-Releases_ \(e.g. `16.6.x` on `16.7.x`\) contain new features or get deprecation warnings that prepare the developer for upcoming major releases, while _Patch-Releases_ \(e.g. `16.8.0` on `16.8.1`\) only contain bugfixes.

Before major or minor releases are released, there are also alpha, beta and rc \(_"Release Candidate"_\) versions available on a regular basis, with which you can take a look at upcoming features in advance. However, these should be taken with caution, as the functionality of new features may change before the final release.

![Example for a Deprecation Warning](../.gitbook/assets/deprecation-warning.png)

This is probably due to the fact that Facebook also uses a lot of React components and you can't just make profound changes without causing problems. The thoughts and justifications of the developers can be followed in detail in the GitHub Issue Tracker, all important changes are summarized in so-called[ Umbrella-Tickets](https://github.com/facebook/react/issues?utf8=✓&q=is%3Aissue%20is%3Aopen%20umbrella).

