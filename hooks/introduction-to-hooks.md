# Introduction to Hooks

**Hooks** are a completely new concept that has been introduced in React **16.8.0**. This concept is so new and novel that I want to dedicate a full chapter to it. Many React Core Developers have already titled **Hooks** as one of the most exciting and fundamental shifts in React. Indeed, **Hooks** were making waves at **React Conf 2018** when they were first announced and then introduced in their first alpha release in React 16.7. Even other frameworks have followed suit and implemented their own version of **Hooks**. But what are Hooks exactly?

**Hooks** allow us to use certain mechanisms that were only really available to Class components before in **Function components**. Features such as `setState` or lifecycle methods such as`componentDidMount()` or `componentDidUpdate()` reserved entirely for Class Components in the past, can now be used in **Function components** thanks to **Hooks**. Put simply, **Hooks** are nothing more than special functions that follow a pre-defined schema. One of them is the convention that the name of a **Hook** has to start with `use`.

React offers a number of internal **Hooks** that are ready to use out of the box such as `useState`, `useEffect` or `useContext`. It also allows the creation of own **Custom Hooks** Functions in which we can use our own logic. To follow the convention mentioned already, they also have to start with the `use` keyword but apart from that the developer can freely choose to give the Hook any name they want as long as the name is also a valid JavaScript function name. Names such as `useAccountInfo` or `useDocumentInfo` are great examples.

{% hint style="info" %}
A little personal anecdote: Since the introduction of **Hooks**, I have had to rewrite entire sections in the book and still have to to this day. In the past, the documentation has almost entirely focused on calling **Function components** - **Stateless Functional components** \(SFC for short\) - in the past. This naming has disappeared immediately from the official  React documentation since **Hooks** came to be which is why I also removed this name from the book.
{% endhint %}

**Hooks** allow us to reuse component logic between in a coherent fashion. Before **Hooks**, many components often used identical `componentdidMount()` or `componentDidUpdate()` methods that followed the same implementation patterns with the difference of checking whether certain different parameters had changed during `componentDidUpdate()` . For example whether a user ID had changed which was passed via props which in turn initiated another API request.

**Hooks** were thus introduced with the aim to make it possible to share complex logic much easier and without the need for writing a  lot of unnecessary duplicated code. But I will be honest with you, if you are comfortable with Class components and how they work, Hooks might force you to completely re-think how components can and should interact. Processes as well as the composition of the components themselves have changed. Where complex classes with class properties, inheritance and a mutual `this` context have prevailed in the past, **Hooks** replace these with relatively simple and clean functions. How this is achieved in detail, we are going to look at in the course of this chapter.

## Are class components still worth it?

Now we only have to answer the question: are class components still worth using?

This question came up again and again shortly after the announcement and introduction of **React Hooks**. The React Team has discouraged people from rewriting their existing applications which use classes at the moment into **Function components** using **Hooks**. Classes will remain part of React.

The community did not really take this advice to heart and many developers started rewriting entire applications post **React 16.7.0-alpha** \(the first version containing **Hooks**\) and announced on Twitter how excited they were about the simplicity **Hooks** introduced for their development and the reduced overhead from not having to use Class components.

Of course you can continue to use Class components as they continue to be supported. There are no plans to remove them from React either. Once you have started getting used to writing components with React **Hooks** however, it will be tough to trade the simplicity and increased understanding of this new component.

## Class Components vs Hooks - a comparison

In order to demonstrate just how much simpler a component definition can be by using Hooks, Sunil Pai \(one of the React Core Team developers\) has created an example in which he compares a Class component with a **Function component** using **Hooks**. Logic that is shared between the two is coloured in the same colour in both examples. Logic from the class component that is no longer needed has been blacked out. What you see below is an interesting result which demonstrates clearly just how the information is now bundled in the version using Hooks and not littered across the component:

![Class component](../.gitbook/assets/react-class.jpg)

![The same functionality with React Hooks](../.gitbook/assets/react-hooks.jpg)

Source: [Sunil Pai on Twitter](https://twitter.com/threepointone/status/1056594421079261185)

