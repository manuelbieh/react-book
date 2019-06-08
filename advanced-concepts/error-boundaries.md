# Error Boundaries

If an error occurs within a React application and an exception is thrown, the application may no longer be displayed and the user may only see a white screen. To prevent this unsightly behavior, React 16.0 introduced the so-called **Error Boundaries**.

This is a specific type of component that can intercept and process various errors **within its child hierarchy** and, in the event of an error, render an alternative page tree to save the user from a complete crash and the sight of a _white screen_. **Error Boundaries** always act as a parent component of a page tree. If an error occurs in a component within this page tree, the Error Boundary jumps in and takes care of the handling of the corresponding error. This behavior can be seen as a special kind of `try`/`catch` block for component hierarchies.

They take care of the handling of errors resulting from one of the following situations:

* Error in **Lifecycle Methods**
* Error in the `render()` method somewhere **below** the Error Boundary
* Error in `constructor()` of a component

If an error occurs in a lifecycle method, a `render()` method, or in the constructor of a component, it is caught by the **Error Boundary**. This can then react with a fallback display, prompt the user to restart the application, or inform the user what went wrong. Similar to context components, error boundaries can also be nested in each other. If an error then occurs, the implementation of the next higher Error Boundary component takes effect.

{% hint style="info" %}
**Note:**  _Error Boundary **components are primarily about catching and handling** user interface-specific errors\*_ that make rendering a particular application status impossible. It would also be conceivable to implement form validation using error boundaries, but this would contradict the intended purpose and is therefore not recommended!
{% endhint %}

There are also situations where Error Boundaries do not grab. This is the case:

* in event handlers
* in asynchronous code, such as `setTimeout()` or `requestAnimationFrame()`
* for server-side rendered components \(Server Side Rendering\)
* for errors that occur in the **Error Boundary** itself

In these situations a **Error Boundary** does not apply, because it is either not possible or not necessary. If, for example, an event handler throws an error, this does not directly affect the rendering and React can still display a user interface. In this case, no corresponding interaction is carried out based on the event that took place.

## Implementing an Error Boundary

There are two simple rules for implementing a **Error Boundary**:

1. only class components can become a **Error Boundary**
2. a class has to implement either the static method `getDerivedStateFromError()` or the class method `componentDidCatch()` or both methods at the same time

From a technical point of view, an **Error Boundary** is only a component that implements one or both of the above methods. Otherwise, the same rules apply to them as to other class components.

Let's have a look at a simple implementation of a **Error Boundary**:

```jsx
class ErrorBoundary extends React.Component {
  state = {
    hasError: false,
  };

  static getDerivedStateFromError(error) {
    return {
      hasError: true,
      error,
    };
  }

  componentDidCatch(error, info) {
    console.log(error, info);
  }

  render() {
    if (this.state.hasError) {
      return <h1>An error has occurred.</h1>;
    }

    return this.props.children; 
  }
}
```

So first of all we define a new component. This is called `ErrorBoundary` here, but this is not a mandatory name, which is absolutely necessary. The name of a **Error Boundary** component can be selected here as freely as for any other component, it only has to comply with the rules for component names. So basically start with a capital letter and be a valid JavaScript function name.

For the sake of clarity, however, I recommend that you make it clear in the name of the component that it is a component that is used in particular for error handling. So for example names like `AppErrorBoundary` or `DataTableErrorFallback`. Thus, the new colleague in the project quickly becomes aware of the purpose of the corresponding component.

In the above example, we initialize a state in the component with the `hasError` property and its default value `false`. Finally, no error normally occurred when initializing the component.

Next we see the static `getDerivedStateFromError()` method. At this point, we signal React that we are dealing with a component that acts as **Error Boundary** and that is used if an error occurs below the component  \(i.e. in the child elements\). The method receives a `error` object, which also corresponds to the object that the `catch` block receives at a `try`/`catch`.

The method works very much like the `getDerivedStateFromProps()` method in the lifecycles. It can return an object or `null` and thus create a new state or just leave everything as it is, it just has to return `null`. In the example, we set the `hasError` property to `true` and additionally store the `error` object in the state. However, since the method is static, it cannot access other methods of the component.

The `getDerivedStateFromError()` method is executed during the _Render_ phase. So while React compares the latest state of the component tree with the current one, and before the new changes are finally written to the DOM \(_"committed"_\).

We have also implemented the `componentDidCatch()` method here. The error object is passed as the first parameter and the React info as the second parameter. This contains the _"Component Stack"_, the information about which component the error occurred in and which child components and child-child components were involved, thus mapping the component tree to the component with the error. If an external service is to be used to log the errors, this is the right place for it, because this is the intended purpose of this method. Side effects are to take place here. The method is only executed in the _Commit_ phase, i.e. after React has mapped the changes to the state in the DOM.

Since `componentDidCatch()` is not a static method, it would also be possible to modify the state of the component with `this.setState()`. However, it is planned to prevent this in the future, which is why a cautious approach should be taken here. In any case, the better way is to use the static `getDerivedStateFromError()` method to create a new state after an error occurs and respond to the error.

Finally, we respond to a possible error in our render\(\) method. If the `hasError` property in the state of the component is `true`, we know that an error has occurred and display a message `<h1>An error has occurred</h1>`. If everything is okay, we simply return the child elements of the component \(`this.props.children`\). It is up to the developer how exactly the error is reacted to. In the case of serious errors, it would be conceivable to ask the user to reload the application or, in the case of minor errors, to display only the message on the screen that a component cannot currently be displayed.

## Using an Error Boundary

We now know how to implement a **Error Boundary** by adding `static getDerivedStateFromError()` or `componentDidCatch()` in a component. **Error Boundaries** should be as independent as possible and implement little or no logic of their own or be coupled too closely to certain components. How granular such a **Error Boundary** is in the end depends on the decision of the developer.

In complex applications it makes sense to have different \(also nested\) **Error Boundaries** that cover different error cases. One that wraps around the entire application and intercepts errors, another for specific areas of the page tree that may be optional. Let's take a look at an example of this:

```jsx
import React from 'react';
import ReactDOM from 'react-dom';

const App = () => {
  return (
    <ErrorBoundary>
      <ApplicationLogic />
      <ServiceUnavailableBoundary>
        <WeatherWidget />
      </ServiceUnavailableBoundary>
    </ErrorBoundary>
  )
};

ReactDOM.render(<App />, document.querySelector('#root'));
```

Here we have two **Error-Boundary** components: `ErrorBoundary` and `ServiceUnavailableBoundary`. While the outer `ErrorBoundary` component may well represent the above example 1:1, i.e. only one `<h1>an error has occurred.</h1>`, if an error occurs in the `ApplicationLogic` component, the `ServiceUnavailableBoundary` component may issue an alternative message such as \_"The requested service is currently unavailable. Please try again later if the error occurs in the \(fictitious\) `WeatherWidget` component.

If an error occurs in the `WeatherWidget` component, it is intercepted by the `ServiceUnavailableBoundary` and everything rendered in the `ApplicationLogic` component remains intact! If we didn't include the `WeatherWidget` in its own **Error Boundary**, the outer **Error Boundary** would grab and not display the `ApplicationLogic` component either!

In general it can be said that it is recommended to always have at least one **Error Boundary** in your application which is as high up in the component hierarchy as possible and intercepts all unexpected errors \(and logged!\) and acts similar to a `500 Internal Server Error` page. If necessary, you can and should add more **Error Boundaries** and enclose certain side trees with them. Depending on how great the danger of a rendering error in a certain area of a tree is \(e.g. because you are dealing with unknown or changing data\), or how negligible a certain component tree is.

{% hint style="info" %}
Since version 16, components in which a serious error occurs, in short in which an error/exception is thrown, are completely "unmounted" by React and disappear from the page tree. This is important to ensure that the user interface does not suddenly remain in an unexpected and erroneous state in the event of an unhandled error. This is critical, for example, in an online banking application if transfers are then made to an incorrect recipient or an incorrect amount is transferred.

The **Error Boundaries** have been introduced to handle such errors in the rendering of components. They can be used to alert users to the erroneous state of the application. Since errors in an application can never be completely excluded \(users can be very resourceful!\), the use of **Error Boundaries** is highly recommended!
{% endhint %}

