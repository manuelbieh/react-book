# Error Boundaries

Whenever an error occurs and an exception is thrown in a React application, there is the strong possibility that the application display no longer works and that the user will only see a blank page. In order to avoid this behavior, React introduced so-called **Error Boundaries** in version 16.0.

An **Error Boundary** describes a component which can catch certain errors **in its children** and can also render an alternative component tree to protect users from experiencing a blank page. They always serve as a parent component of a component tree. If an exception is thrown in this component tree, the Error Boundary can interject and handle the error. You can think of error boundaries as a special form of a `try / catch` block for component hierarchies.

They can deal with mistakes that result from the handling of the following situations:

* Errors in **lifecycle methods**
* Errors in the `render()` method anywhere **inside** the Error Boundary
* Errors in the `constructor()` of a component

If React encounters an error in a lifecycle method, a `render()` method or in the constructor of a component, the Error Boundary can safely mitigate it. It can display a fallback that can prompt the user to restart their application or inform them that something has went wrong. Similar to Context components, Error Boundaries can be nested inside each other. If an error occurs, the implementation of the higher Error Boundary component takes precedence.

{% hint style="info" %}
**Attention: Error Boundaries**' primary goal is to prevent and deal with errors in the handling of user interfaces which would otherwise prevent further rendering of the application status. If you think about implementing form validation with Error Boundaries, please refrain from doing so as Error Boundaries were not intended for this use case and should not be used for that matter in this case.
{% endhint %}

There are certain situations in which Error Boundaries would not take effect:

* in event handlers
* in asynchronous code \(like `setTimeOut()` or `requestAnimationFrame()`\)
* in server-side rendered components \(SSR\)
* in errors which occur in the **Error Boundary** itself

**Error Boundaries** will not work in these situations as it is either not necessary or not possible for them to deal with the problem at hand. If an event-handler throws an error, this might not necessarily impact its render and React can continue to show a working interface to the user. The only repercussion would be the missing interaction based on said event.

### Implementing an Error Boundary

There are two simple rules when it comes to implementing an **Error Boundary**:

1. Only Class components can be turned into an **Error Boundary**
2. The class has to implement the static `getDerivedStateFromError()` method or the class method `componentDidCatch()` \(or both of them\)

Strictly speaking, we are already dealing with an **Error Boundary** from technical point of view if one or both of the methods mentioned above have been implemented. All other rules that apply to regular Class components also apply to **Error Boundaries**.

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

