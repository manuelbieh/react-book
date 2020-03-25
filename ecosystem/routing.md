# Routing

Most **Single Page Applications** will need some form of **routing** at some point of time., meaning that we want to map certain functions or behavior to a URL. For example, if I visited the URL of `/users/manuel`, I would like to present the profile of the user `manuel`.

**React Router** has established itself as the standard for **routing** in React Single Page Applications. Developed by Michael Jackson \(yes, that's his name!\) and Ryan Florence \(who has built over 10 different routers throughout the years\), it boasts with over 35,000 stars on GitHub showing its great popularity. It's well maintained, has over 500 different contributors listed on GitHub and integrates well with React principles due to its declarative nature. Moreover, it can be used on the web \(client- and server-side\) and in React Native. It's a very universal, well-tested and popular choice for routing.

The interface of React Router is relatively simple. In 95% of situations, you will only really encounter five core components: `BrowserRouter`, `Link`, `Route`, `Redirect` and `Switch`. It also offers an imperative History API, which can be extended via the `history` package which forms a thin layer over the native browser implementation making it cross-browser compatible. It also provides a Higher-Order-Component called `withRouter` which allows us to pass in routing relevant data from the router to another component.

You can install **React Router** via:

```bash
npm install --save react-router-dom
```

or:

```bash
yarn add react-router-dom
```

React Router's usage is _declarative_ and can be achieved via the components mentioned above. Routers can be used anywhere in the application as long as the page tree itself is nested in a **Router Context.** In most cases, this context will wrap the entire application and exist only once:

```jsx
import React from "react";
import ReactDOM from "react-dom";
import { BrowserRouter as Router } from "react-router-dom";

const App = () => {
  return <Router>[...]</Router>;
};

ReactDOM.render(<App />, document.getElementById("root"));
```

### Defining routes

Each component placed inside of the `<Router></Router>` element, can access the **Router Context**, react to it or manage it. We create routes, by using the Route component and providing a `path` prop as well as an optional `render` or `component` prop \(the exception being the 404 route\). The value of a `render` prop has to be a **function** that returns a valid **React element** whereas the `component` prop expects a **component** \(not an _element_\).

The correct implementation of both props can be summarized as the following:

```jsx
import React from "react";
import ReactDOM from "react-dom";
import { BrowserRouter as Router } from "react-router-dom";

const Example = () => <p>Example Component</p>;

const App = () => {
  return (
    <Router>
      <Route path="/example" component={Example} />
      <Route path="/example" render={() => <Example />} />
    </Router>
  );
};

ReactDOM.render(<App />, document.getElementById("root"));
```

This example would render the `Example` component twice once the `/example` route is hit, as the Route component merely checks whether the path of the current URL is the same as the one provided in the `path` prop. This might sound a little odd at first, but there's a logical explanation.

As mentioned previously, React Router works declaratively which means that if we ask for two different routes that we will also receive two components if the URL matches. This can be useful if we want to render different parts of the page independently, based on a URL. Assume that we define an App with a sidebar and a content area. Both of these should react to a URL:

```jsx
import React from "react";
import ReactDOM from "react-dom";
import { BrowserRouter as Router, Route } from 'react-router-dom';

const Home = () => <p>Home Content</p>;
const Account = () => <p>AccountContent</p>;
import HomeSidebar = () => <p>Home Sidebar</p>
import AccountSidebar = () => <p>AccountSidebar</p>

const App = () => {
  return (
    <Router>
      <main>
        <Route path="/account" component={Account} />
        <Route path="/" component={Home} />
      </main>
      <aside>
        <Route path="/account" component={AccountSidebar} />
        <Route path="/" component={HomeSidebar} />
      </aside>
    </Router>
  );
}

ReactDOM.render(<App />, document.getElementById("root"));
```

In this example, we can see that two different components are rendered in different parts of our application, depending on the URL which is currently active.

However, while the above is completely valid code, one could argue that we are creating unnecessary duplication in this example. Thus, many try to avoid this structure and would rewrite the above example as the following:

```jsx
import React from "react";
import ReactDOM from "react-dom";
import { BrowserRouter as Router, Route } from "react-router-dom";

const Home = () => (
  <>
    <main>Home Content</main>
    <aside>Home Sidebar</aside>
  </>
);

const Account = () => (
  <>
    <main>Account Content</main>
    <aside>Account Sidebar</aside>
  </>
);

const App = () => {
  return (
    <Router>
      <Route path="/account" component={Account} />
      <Route path="/" component={Home} />
    </Router>
  );
};

ReactDOM.render(<App />, document.getElementById("root"));
```

While we avoided duplication of routing in this case, we have created duplication of the layout code. One should probably abstract this structure in its own layout component.

```jsx
import React from "react";

const Layout = (props) => (
  <>
    <main>{props.content}</main>
    <aside>{props.sidebar}</aside>
  </>
);

const Home = () => <Layout content="Home Content" sidebar="Home Sidebar" />;
const Account = () => <Layout content="Account Content" sidebar="Account Sidebar" />;

const App = () => {
  return (
    <Router>
      <Route path="/account" component={Account} />
      <Route path="/" component={Home} />
    </Router>
  );
};

ReactDOM.render(<App />, document.getElementById("root"));
```

If you have tried these code examples on your own, you might have noticed something that strikes you as a little odd. **React Router** intentionally has a rather relaxed approach to path matching. When we hit the `/account` URL, we do not only render the `Account` component, but also the Home component as the `/account` url **also includes** the path of `/` which then renders both components. This is intentional as it allows us to group certain areas of the page under a certain URL prefix and have all components render on these types of routes.

Imagine a user account and a sidebar within this user account. Let's assume that we are building a community which has a user area which is divided into different subcategories: `/account/edit` to edit your profile, `/account/images` to view your own pictures or `/account/settings` to change your account settings. We can use a generic route in this application:

```jsx
<Route path="/account" component={AccountSidebar} />
```

This `AccountSidebar` component would now be rendered on every subpage that is part of the `/account` area, as all their URLs would also include `/account`.

### Limit matching with props

In order to limit the matching between `path` and the URL, React Router provides an `exact` prop on the `Route` component. If this Boolean prop is provided, the route is only rendered if the path prop exactly matches the current URL.

```jsx
<Route exact path="/" component={Home} />
```

Where exactly you place this prop in **JSX**, is not important. I like to place it just before the path prop to let it speak for itself: "Here is a route that matches an **exact path.**" If we included the `exact` prop in the Account Sidebar, the sidebar would only be rendered if the URL with `/account` was hit, and would not register the components for `/account/edit`, `/account/images` or `/account/settings`.

### Limiting matching to a single route via Switch component

The `exact` prop only ever covers a **single route** and does not affect other routes at all. If we have a number of URLs which could all match multiple routes, it would be very cumbersome to add an exact prop to all of these routes. **React Router** offers a solution to this problem though by offering the `Switch` component.

The `Switch` component that can wrap a number of `<Route />` elements, helps us to only ever render the **first** route whose `path` matches with the currently present in the URL. It is not a bad idea to wrap Routes with a `Switch` element by default unless you want more than one route rendered. In the above example, we could have used a `Switch` component instead of the `exact` prop too:

```jsx
<Router>
  <Switch>
    <Route path="/account" component={Account} />
    <Route path="/" component={Home} />
  </Switch>
</Router>
```

Once the `/account` URL is hit, the first route would match and all other routes that follow would be ignored. In this case, only the `Account` component would get rendered. `Switch` components only match the URL of their **direct** children. If the `Account` component contained its own routes, the Switch component would not take care of these. These routes would get rendered if the `path` matched the current URL.

Using Switch, we can implement a 404 page as a fallback route. If we remove the `path` prop from the route, the route now matches **every** URL. However, if it is used within a `Switch` element as the very last component, we inform **React Router** to only ever render this component if no other component matches:

```jsx
const Error404 = () => <h1>404 – Page not found</h1>;

const App = () => (
  <Router>
    <Switch>
      <Route path="/account" component={Account} />
      <Route path="/contacts" component={Contacts} />
      <Route path="/inbox" component={Inbox} />
      <Route exact path="/" component={Home} />
      <Route component={Error404} />
    </Switch>
  </Router>
);
```

In this particular `Switch` element example, we also need to provide an `exact` prop for the `/` route. Why? If we did not provide the exact prop, this route would always match. Even an error route like `/does-not-exist` would be found under the `/` route. By providing the `exact` prop on the `/` route, we avoid this particular problem and an error component at the end of the `Switch` component can be safely rendered \(if no other route matches\). It fulfills a similar job to the `default` case of a `switch` statement in JavaScript.

### Parameters in URLs

Most applications require some sort of usage of parameters in URLs. React Router also supports parameters by using colons \( : \) which many of you might be already familiar with.

```jsx
<Route path="/users/:userid" component={UserProfile} />
```

One can easily restrict which parameters should be detected and can even provide further customization. For example, React Router allows the limitation of matching of routes by providing an `asc` \(ascending\) or `desc` \(descending\) keyword in regular expressions just after the parameter:

```jsx
<Route path="/products/:order(asc|dec)" />
```

The above route would only match, if the URL provided was either `/products/asc` or `/products/desc`.

If we were to only allow numerical values in the ~~`:userid`~~, we would define routes such as `/users/:userid(\d*)` or `/users/:userid([0-9]*)` to limit these. A URL of `/users/123` would lead to a render of the `UserProfile` component, while `/users/abc` would not.

If **React Router** finds such a URL, the value of the parameter is extracted and passed in to the rendered component via the `match` prop.

### Controlling redirects of particular routes

Apart from the usual `Route` component to react to particular Routes, **React Router** also offers a `Redirect` component. The Redirect component is initialized with a `to` prop in which we can provide a destination URL that the component should redirect to. It allows us to declaratively decide in **JSX** where to send a particular user in certain situations. Whenever a `Redirect` component is equipped with only a `to` prop, a redirect to the URL provided will take place.

`Redirect` components are a great solution to the common use case of having to redirect users to a login page if they are not logged in. Logged in users will continue to be directed to a Dashboard:

```jsx
<Route
  exact
  path="/"
  render={() => {
    return isLoggedIn ? <Dashboard /> : <Redirect to="/login" />;
  }}
/>
```

The `render` prop of the Route component can be used to check whether a user is logged in. If they are, a `Dashboard` component will be shown on the `/` Route, otherwise the user will be redirected to the `/login` Route via the `Redirect` component.

`Redirect` components can also be used inside `Switch` elements. It behaves just like a `Route` component and only matches if no other `Route` or `Redirect` has matched with the current URL.

If the `Redirect` is used inside of a `Switch` element, it can also receive a `from` prop. This is the equivalent to the `path` prop of the `Route` component, but ensures that the `Redirect` is taking place whenever the URL matches the value provided in the from prop:

```jsx
<Switch>
  <Redirect from="/old" to="/new" />
  <Route path="/new" component={NewComponent} />
</Switch>
```

The `Redirect` component behaves just as the `Route` component concerning URL matching. It also supports the `exact` prop and also supports redirects to other routes with parameters:

```jsx
<Switch>
  <Redirect from="/users/:userid" to="/users/profile/:userid" />
  <Route path="/users/profile/:userid" component={UserProfile} />
</Switch>
```

If the URL `/old` is hit \(first example\) or `/users/123` \(second example\), the user will be redirected to the URL specified in the `to` prop.

### Using Router Props

Each component that was rendered by React Router and has been added as a `component` prop to a `Route` component, automatically receives three other props:

- `match`
- `location`
- `history`

Each of these props can be accessed just like any other props. **Class components** can access these via `this.props` whereas **function components** can access these with `props`:

```jsx
import React from "react";
import ReactDOM from "react-dom";
import { BrowserRouter as Router, Route } from "react-router-dom";

const Example = (props) => {
  console.log(props);
  return <p>Example</p>;
};

const App = () => (
  <Router>
    <Route path="/users/:userid" component={Example} />
  </Router>
);

ReactDOM.render(<App />, document.getElementById("root"));
```

Let's have a look at the console output of this component when the route `/users/123` is hit:

```javascript
{
  history: { /* ... */ },
  location: { /* ... */ },
  match: {
    path: "/:userid",
    url: "/users/123",
    isExact: true,
    params: {
      userid: "123"
    }
  }
}
```

Without getting into too much detail for each and every property, we get a feel for which properties of the router we are able to access. For now, we only care about the `match` property which will contain the `paths` we have defined in the `match.params` once a matching URL is called. In this case, we can access `match.params.userid` which will yield the value of `123`.

A possible extension of this use case could be the start of an API request in a user profile component. The user id could be used to fetch the particular data for the user `123` and then display their user profile.

**React Router** ensures that each component connected to the Router will have a `match` property which always has its own `params` property. This will either contain the actual parameters or an empty object. `props.match.params` is therefore safe to access without having to fear that the property might be `undefined` and throwing an error.

The `render` prop on the `Route` component also receives the props of the router:

```jsx
<Route
  path="/users/:userid"
  render={(props) => {
    return <p>Profile of ID {props.match.params.userid}</p>;
  }}
/>
```

### Navigating different routes

Once we've divided an application into different routes, we would also like to be able to link between these URLs. While we could easily use regular HTML anchors `<a href="...">...</a>`, this is not recommended. Each time such an anchor is used, we would trigger a "hard" page refresh in the browser. The page would be left **completely** and then be loaded again.

In practice, we would ask for an HTML document which would in turn load CSS and the JavaScript containing our React application from the server again \(unless it is in the browser cache\). Doing this would mean that everything is re-initialised based on each new URL. Any state that might have been set globally previously would be reset.

Single Page Applications should not follow this pattern and any HTML, CSS and JavaScript should only be loaded from the server once. Global state should be persisted once one navigates through the routes and only those parts of the page which actually change, should re-render.

To facilitate said behavior, **React Router** supports a `Link` component. It can be imported from the `react-router-dom` package and also comes with a `to` prop, which roughly equates to the regular `href` attribute on a HTML anchor element.

```jsx
<Link to="/account">Account</Link>
```

**React Router** also uses an `<a href />` under the hood. However, any clicks to this page are being intercepted and sent to an internal function which then deals with displaying new page content based on the new URL - without triggering a complete refresh of the page.

Apart from the `to` prop, `Links` can also be equipped with an `innerRef`. These will be filled by `createRef()` or `useRef()` which both create a ref that can be used by the component. Moreover, `Link` also supports a `replace` prop which allows us to replace the current URL in the browser history instead of creating a new history entry. Be careful though, you cannot access the previous route when pressing the back button in the browser anymore if you chose to make use of the `replace`prop.

Any other props that are passed to the `<Link />` element, will be passed down to the generated anchor element. `<Link to="/" title="Homepage">Home</Link>` would generate the following markup: `<a href="/" title="Homepage">Home</a>`.

### Special case: NavLink

A special form of the `Link` element is the `NavLink` element. Apart from the usual props that can also be received by the `Link` component, `NavLinks` can change based on their state. `NavLinks` can access information relating to which page they are currently linking to and whether that page is the same as the current page. If this is the case, we can alter it's display using `activeClassName` and `activeStyle`.

A classic example for this type of behavior is the overall page navigation. The currently active route in the menu is highlighted in a different color:

```jsx
<NavLink to="/" activeClassName="active">Home</NavLink>
<NavLink to="/account" activeClassName="active">Account</NavLink>
<NavLink to="/contacts" activeClassName="active">Contacts</NavLink>
```

Only the Link of the current page receives the `activeClassName` active. If we had navigated to the `/account` URL, the markup would resemble the following:

```markup
<a href="/">Home</a>
<a href="/account" class="active">Account</a>
<a href="/contacts">Kontakte</a>
```

`NavLinks` can also receive an `exact` and `strict` prop \(similar to the same **props** for the `Route` component\) as well as an `isActive` prop. The latter expects a function which is either returns `true` \(if the current page is the same as the one provided in `NavLink`\) or `false` \(active page is not the same as `NavLink`\). The function takes the aforementioned `match` object as its first argument and a `location` object as its second which are passed in from the router. The function can then decide whether to mark the `NavLink` as active or not - based on the information available.

### Navigating programmatically using the History API

We now know how we can use `Route` elements on different URLs and how to react with different components. We have also learned how to avoid a full page reload by using the `<Link />` element. In some cases however, it can be useful to programmatically force the change of a URL. For example, we might want to send the user to another site after successful completion of an asynchronous request.

The `history` property which can be accessed via each `Route`'s props can help with this endeavor:

```javascript
{
  history: {
    action: "POP"
    block: Function(prompt),
    go: Function(number),
    goBack: Function(),
    goForward: Function(),
    length: 1
    push: Function(path, state)
    replace: Function(path, state)
  },
  location: { /* ... */ },
  match: { /* ... */ }
}
```

Let's look at the `push()` and `replace()` functions in particular. Using `props.history.push('/destination')`, we can change the URL to `/destination` as well as triggering a re-render. This will create a new entry in the browser's history just as using a `Link` component did. If no entry in the browser's history is desired, the `props.history.replace('/destination')` function can be used instead.

We also have access to a so-called `props.history.go()` function which allows us to programmatically switch between entries in the browser's history. The function expects a parameter which indicates how many pages ahead or back should be turned. A negative value informs the function to go back in the entries whereas a positive value informs the function to go forward in the collection of entries. `goBack()` and `goForward()` are shortcuts for going back a step in the browser's history and forward. They are equivalent to calling `go(-1)` and `go(1)`.

The `action` property in `history` confirms how the user has ended up on the current page. Possible values include `POP`, `PUSH` or `REPLACE`. `POP` can either signify that the user has pressed the back button in the browser or that they have loaded the page for the first time. `PUSH` informs us that the `history.push()` method has been called which is also the case once a `<Link />` has been clicked. If the `action` properties value is `REPLACE`, `history.replace()` has been called or a `<Link />` element has been clicked that contained a `replace` property.

### Connecting components with a router using HOC

Each component which is used as a `component` prop in a `Router` element, is automatically passed the router props `history`, `location`, and `match`. Sometimes however, it would prove useful to not only be able to access Router functionality in components which are not using a direct Route. For example: to redirect to another page using `history.push()`.

In order to allow for such use cases, **React Router** provides a `withRouter` Higher Order Component. Each component wrapped by this HOC will automatically receive the router's props even though they are not used as a Route component:

```jsx
withRouter(MyComponent);
```

This component is sometimes used in another situation: it can prevent **"Update blocking"**. This used to be a necessary workaround before **React Router 5.0.0**. Since then however, this problem has been solved and is no longer necessary. I will still outline the reasons and rationale as to why this is necessary though should you ever find yourself working with older versions of **React Router**.

If components have been implemented as `PureComponent`s or have been wrapped by a `React.memo()` call for optimization reasons, re-renders are suppressed if no **props** or **state** have changed. As most Router components, for example `NavLink`, access data in the router via **React context**, a `PureComponent` or component wrapped by `React.memo()` might not receive any information that its children need re-rendered.

In those types of situations, it is recommend to wrap these components with a `withRouter()` HOC. Whenever a change in the Routing occurs and new props with a new location are passed to the respective component, a re-render will be triggered. This principle also applies to the state management library **Redux**. If a component is connected to the **Redux** store via the `connect()` function, the component would prohibit the re-rendering of router-specific parts, unless something has also changed in the store.

To avoid such cases, these components should be wrapped by a `withRouter()` HOC:

```javascript
withRouter(connect()(MyComponent));
```

Since **React 16.3.0** however and **React Router 5.0.0**, this issue has been solved though and you will only come across it while working with previous versions of these two libraries.

### React Router and Hooks

Since React Router **5.1.0**, the library has been enhanced by a number of own hooks that are available to the developer. It is now possible to access Router information in **function components** without using the `withRouter()` HOC provided that these components are not directly used as argument for the `<Route />` element prop component \(`<Route component={MyComponent} />`\).

As is the case with most hooks, using React Router's own hooks is a very pleasant and straightforward experience. Four hooks allow us to access the `location` object, the `history` instance, the Route parameter or the `match` object. Conveniently, these hooks are aptly called `useLocation`, `useHistory`, `useParams` and `useRouteMatch`. In order to use these hooks, the component intended for the hooks implementation needs to be be nested within a `<Router>` tree. However, it is not necessary for the component to follow directly after the Router element or be placed on the most upper layer. These hooks can access the Router context and can thus be used anywhere in your application.

#### useLocation\(\)

First, let's have a closer look at the arguably most simple hook in our list. We can import it via the `react-router-dom` package. Once installed, we can access location data by using the return value of the hook:

```jsx
import React from "react";
import { useLocation } from "react-router-dom";

const ShowLocationInfo = () => {
  const location = useLocation();
  return <pre>{JSON.stringify(location, null, 2)}</pre>;
};
```

In this example, we would obtain the following output for the `showLocationInfo` component:

```javascript
{
  "pathname": "/",
  "search": "",
  "hash": ""
}
```

#### useHistory\(\)

This hook allows us to the `history` instance of React Router. It can access and change the URL via the `push()` and `replace()` methods and thus trigger a re-render of the application. `goBack()`, `goForward()` as well as the more general `go()` can be used to navigate the browser's history.

```jsx
import React from 'react';
import { useHistory } from 'react-router-dom';

const NavigateHomeButton = () => {
  const history = useHistory();

  const goHome = () => {
    history.push('/');
  };

  return <button onClick={goHome}Take me home</button>;
}
```

This example demonstrates how we can implement a button which will direct us to the home page once it's clicked using the `useHistory()` hook.

#### useParams\(\)

This hook is basically a shortcut for accessing parameters which were hidden in `match.params`. If a route has been implemented using placeholders, such as `/users/:userid` , and then a URL such as `/users/123` has been called, the `params` object will contain a key/value pair in the form of `{ "userid": "123" }.`

The `useParams()` hook allows us to _directly_ access this object:

```jsx
import React from "react";
import { useParams, useLocation } from "react-router-dom";

const ShowParams = () => {
  const params = useParams();
  const location = useLocation();
  return <pre>{JSON.stringify({ location, params }, null, 2)}</pre>;
};
```

If a route such as `/users/:userid` had been created and a URL such as `/users/123` has been called, the output would resemble the following:

```javascript
{
  "location": {
    "pathname": "/users/123",
    "search": "",
    "hash": ""
  },
  "params": {
    "userid": "123"
  }
}
```

#### useRouteMatch\(\)

The last hook introduced by React Router **5.1.0** forms the `useRouteMatch()` hook. It allows us to access the whole `match` object of a Route, meaning we can now obtain information on `params`, `url`, `path` and `isExact`. This means that one can now safely check whether the URL matches the `path` of a route.

The function can take in a path which will in turn return a `match` object for the said route. If no particular path is provided to the function, the path of the current route will be used. Using our previous example with the path of `/users/:userid` , the Route `<Route path="/users/:userid">` will return the following `match` object if the URL `/users/123` is provided as an argument:

```javascript
{
  "path": "/users/:userid",
  "url": "/users/123",
  "isExact": true,
  "params": {
    "userid": "123"
  }
}
```

If the hook is called with a path that does not match with the current route, `null` will be returned from the function:

```jsx
useRouteMatch("/orders/:orderid");
```

Calling this function with `/users/:userid` will return a value of `null`.
