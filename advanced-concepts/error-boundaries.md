# Error Boundaries

Whenever an error occurs and an exception is thrown in a React application, there is a strong possibility that the application display no longer works and that the user will only see a blank page. To avoid this behavior, React introduced so-called **Error Boundaries** in version 16.0.

An **Error Boundary** describes a component which can catch certain errors **in its children** and can also render an alternative component tree to protect users from experiencing a blank page. Error Boundaries always serve as a parent component of a component tree. If an exception is thrown in the component tree, the Error Boundary can intercept and handle the error. Try and think of error boundaries as a special form of a `try / catch` block for component hierarchies.

They can deal with mistakes that result from the handling of the following situations:

- Errors in **lifecycle methods**
- Errors in the `render()` method anywhere **inside** the Error Boundary
- Errors in the `constructor()` of a component

If React encounters an error in a lifecycle method, the `render()` method or in the constructor of a component, the Error Boundary can safely prevent it. It can display a fallback that can prompt the user to restart their application or inform them that something has gone wrong. Similar to Context components, Error Boundaries can be nested inside each other. If an error occurs, the implementation of the higher Error Boundary component takes precedence.

{% hint style="info" %}
**Attention: Error Boundaries**' primary goal is to prevent and deal with errors in the handling of user interfaces which would otherwise prevent further rendering of the application status. If you think about implementing form validation with Error Boundaries, please refrain from doing so as Error Boundaries were not intended for this use case and should not be used for that matter.
{% endhint %}

There are certain situations in which Error Boundaries don't work:

- in event handlers
- in asynchronous code \(like `setTimeOut()` or `requestAnimationFrame()`\)
- in server-side rendered components \(SSR\)
- in errors which occur in the **Error Boundary** itself

**Error Boundaries** will not work in these situations as it is either not necessary or not possible for them to deal with the problem at hand. If an event-handler throws an error, this might not necessarily impact its render and React can continue to show a working interface to the user. The only repercussion would be the missing interaction based on said event.

### Implementing an Error Boundary

There are two simple rules when it comes to implementing an **Error Boundary**:

1. Only Class components can be turned into an **Error Boundary**
2. The class has to implement the static `getDerivedStateFromError()` method or the class method `componentDidCatch()` \(or both of them\)

Strictly speaking, we are already dealing with an **Error Boundary** from a technical point of view if one or both of the methods mentioned above have been implemented. All other rules that apply to regular Class components also apply to **Error Boundaries**.

Let's look at an implementation of an **Error Boundary**:

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
      return <h1>An error has occured.</h1>;
    }

    return this.props.children;
  }
}
```

First of all, we define a new component. We have named this component `ErrorBoundary` but it is possible to give it any other name too. You can freely choose the name of the **Error Boundary** and only need to adhere to React's component naming conventions: components need to start with a capital letter and be a valid JavaScript function name.

For matters of simplicity and readability, I would urge you to choose clear and identifiable component names such as`AppErrorBoundary` or `DataTableErrorFallback`. This will allow other team members in your project to see which components are used to deal with errors at a glance.

In the above example we have set up state with a property of `hasError` and provided an initial value of `false` as errors usually do not occur during initialization.

Next, let's look at the static `getDerivedStateFromError()` method. Using this method, React is informed that the component in use is supposed to act as an **Error Boundary** and should come into effect if an error occurs in its children. The method itself is passed an `error` object which is the same as the object which is also passed to the `catch` block of the `try / catch` statement.

`getDerivedStateFromError()` works very similar to the `getDerivedStateFromProps()` method we have already encountered in the chapter on lifecycle methods. It can return a new object and thus create new state or leave all as is by returning `null`. In the above example, we have set the `hasError` property to `true` and also save the `error` object in our state. As the method itself is static though, it cannot access other methods in the component.

This method is called during the `render()` phase of a component when React compares the current component tree with its previous version and just before the changes are committed to the DOM.

The `componentDidCatch()` method has also been implemented. It receives an error object as its first parameter and React-specific information as its second. This information contains the _"Component Stack"_ — crucial information which allows us to trace in which components we have encountered errors and more specifically how which children and children of children were involved. It will display the component tree up until an error will occur. If you want to use an external service to log these errors, this method is a good place to deal with side effects. `componentDidCatch()` is run during the _Commit_ phase meaning just after React has displayed changes from state in the DOM.

As `componentDidCatch()` is not a static method, it would be entirely possible to modify its state via `this.setState()`. However, the React Team plans to prohibit this usage in the future which is why I do not recommend it at this point. It is safer to use the static `getDerivedStateFromError()` method instead to create a new state and react to errors once they have occurred.

Finally, we react to possible errors in the `render()` method. If the `hasError` property in state is set to `true`, we know that an error has occurred and can thus display a warning such as `<h1>An error occured.</h1>`. If on the other hand everything works as expected, we simply return `this.props.children`. How exactly the errors encountered are dealt with is up to the developer. For example, it might be sufficient to inform the user that certain information cannot be displayed at the moment if the error is only small. If however serious errors have been encountered, we should prompt the user to reload the application.

### Error Boundaries in practice

We now know how to implement an **Error Boundary**: by adding either `static getDerivedStateFromError()` or `componentDidCatch()` to your components. **Error Boundaries** should not implement their own logic, should not be too tightly coupled to other components and be as independent as possible. It's at the developer's discretion to decide how granular the **Error Boundary** should be according to the specific use case.

It's a good idea to implement different and nested **Error Boundaries** to cater to a variety of errors: one Error Boundary that wraps around the whole application, as well as one that wraps only optional components in the component tree. Let's look at another example:

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
  );
};

ReactDOM.render(<App />, document.querySelector('#root'));
```

Two **Error Boundaries** are used in the above example: `ErrorBoundary` and `ServiceUnavailableBoundary`. While the outer boundary will catch errors that might occur in the `ApplicationLogic` component, the `ServiceUnavailableBoundary` could catch errors in the weather widget and display a more granular error message like _"the service requested cannot be reached at the moment. Please try again later"_.

If the `WeatherWidget` component throws an error, the `ServiceUnavailableBoundary` will catch it and everything that is currently used in the `ApplicationLogic` component will remain intact. If we didn't include the `WeatherWidget` in its own **Error Boundary**, the outer **Error Boundary** would be used instead and the `ApplicationLogic` component would not be shown.

Generally, it is good practice to have at least one **Error Boundary** as high up as possible in the component hierarchy. This will catch most unexpected errors like a `500 Internal Server Error` page would do and can also log them. If needed, further **Error Boundaries** should be added to encompass useful logic in further component trees. This depends entirely on how error prone a specific area of the tree is \(due to unknown or changing data\) or if a specific area of the tree has been neglected.

{% hint style="info" %}
Since React version 16, components will be "unmounted" and removed from the tree if a serious error occurred or an exception was thrown. This is important as it ensures that the user interface does not suddenly stop working or returns incorrect data. It is especially critical to ensure if we were to work with online banking data. Imagine the consequences if we were to incorrectly send money to the wrong recipient or transfer an incorrect amount.

In order to deal with these errors and risks properly, **Error Boundaries** were introduced. They allow developers to inform users that the application is currently in an erroneous state. As errors and mistakes can never be fully avoided in an application, using **Error Boundaries** is highly recommended.
{% endhint %}
