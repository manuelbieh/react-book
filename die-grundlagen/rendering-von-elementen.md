# Rendering of Elements

In the previous chapters I mentioned them several times as a matter of course, but what exactly are **React elements**?

**React elements** are the smallest possible component in a **React application**. Using the **elements** you describe what the user will see on the screen later. Despite having the same name, they differ from DOM elements in one essential point: they are only a simple object and therefore cheap to create \(in the sense of performance\). The mere creation of a **React element** using `React.createElement()` does not trigger a DOM operation!

{% hint style="info" %}
React-**elements** are often mixed up with React-**components** or used in analogue language. But that's not correct! **Elements** are the elements of which **components** ultimately consist. **Components** are covered in detail in the next chapter, but if you have read this chapter about **elements** before continuing, you should read it first.
{% endhint %}

We already know how to create a **React element** using **JSX** and that **JSX** is just a simplification to save us a lot of paperwork and constant `React.createElement()` calls. But how do we render a created element, in other words, display it in the browser?

Here we use `ReactDOM`, more precisely its own `render()` method. To render a **React element**, we always need a **Root-** or **Mount node**. This is a DOM node that serves as a placeholder and tells React where to render an element. Theoretically, you can have multiple root nodes in your HTML document without any problems. React controls them all independently of each other. So instead of just one big **React application**, you can also place many small \(or even big\) applications in a single HTML document. However, it is only common to have **a root node** for your **react application**.

Let's get to the essential: to render a **React element**, you pass it as the first argument of the `ReactDOM.render()` method together with the **Root node** as the second argument, i.e. the DOM node into which your **Element** is to be rendered.

Imagine you have a `div` with the ID `root` in your HTML document, which should serve as **Root-Node**:

```markup
<!DOCTYPE html>
<html>
  <head>
    <title></title>
  </head>
  <body>
    <div id="root"></div>
  </body>
</html>
```

The corresponding call is then the following:

```jsx
const myFirstElement = <div>My first react element</div>;
ReactDOM.render(myFirstElement, document.getElementById('root'));
```

If you now execute this code in the browser, you will see **within** the `root`-div your `<div>My first react element</div>`.

**React elements** are **immutable**, i.e. immutable. This means: Once an element has been created, it always represents a certain state \("State"\) in the user interface. The official React documentary metaphorically speaks of a single frame \("Frame"\) in a film. If we want to update our user interface, we have to create a new **React element** with the changed data. Don't worry, it sounds more complicated than it is and happens intuitively later.

React itself is so smart that it uses a comparison algorithm to update only those parts of an application that have actually changed. In this case **React elements** and their child elements are compared with their predecessor versions and only trigger a DOM operation if there is a change. As a result, React applications, properly done, have very good rendering performance, since DOM operations are usually very costly \(i.e. performancelastig\), but React and its **Reconciliation** process are reduced to a minimum. In doing so, whole DOM elements are not always recreated according to the description of a **React element**, but only individual attributes are updated if only one has changed.

Let's take a look at this at the practice:

```javascript
function showTime() {
  var time = new Date().toLocaleTimeString();
  var timeElement = (
    <div>
      <p>It's time now {time} Clock</p>
    </div>
  );
  ReactDOM.render(timeElement, document.getElementById('root'));
}
setInterval(showTime, 1000);
```

Again we create a **React element**, this time it should give us the current time when calling `ReactDOM.render()`. Since we always want to know the exact time, we put the element and the `ReactDOM.render()` call into a function that is called by `setInterval` every 1000 ms.

A look into the **Chrome Devtools** reveals: With each `ReactDOM.render()` call only the time itself is updated, the remaining elements, like the DOM nodes or also not affected part of the displayed text remain untouched:

![React only updates the time itself, nothing else](../.gitbook/assets/react-update.png)

And here we also get to know one of the basic principles of React in practice: the **declarative** procedure for creating user interfaces. Instead of telling our Mini-App **imperativ** to update the time every second, we define **deklarativ** in the **React-Element** that we always want to see the current time at each rendering.

A similar functionality, implemented without React, would have looked something like this instead:

```javascript
function changeTime() {
  var time = new Date().toLocaleTimeString();
  var target = document.getElementById('root');
  target.textContent = 'It is now ' + time ' clock';
}
setInterval(changeTime, 1000);
```

The advantage of the **declarative** approach is that we only describe **states** and say how something should be displayed, not how we want to achieve this target state every step of the way. This makes many things simpler, clearer and at the same time much less prone to errors, especially with more complex applications.

{% hint style="info" %}
In practice, it is more common to call `ReactDOM.render()` only once, usually when opening a page. The repeated call of the `render()` method here only serves to illustrate how **ReactDOM** and **React elements** interact.

Rendering is then usually triggered by **components** by changing their state or by new props being submitted to them from outside. The next chapter continues with components!
{% endhint %}

