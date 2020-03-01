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
import { BrowserRouter as Router } from 'react-router-dom';

const App = () => {
  return (
    <Router>
      [...]
    </Router>
  );
}

ReactDOM.render(<App />, document.getElementById("root"));
```

### Defining routes

Each component placed inside of the `<Router></Router>` element, can access the **Router Context**, react to it or manage it. We create routes, by using the Route component and providing a `path` prop as well as an optional `render` or `component` prop \(the exception being the 404 route\). The value of a `render` prop  has to be a **function** that returns a ****valid **React element** whereas the `component` prop expects a **component** \(not an _element_\).

The correct implementation of both props can be summarized as the following: 

```jsx
import React from "react";
import ReactDOM from "react-dom";
import { BrowserRouter as Router } from 'react-router-dom';

const Example = () => <p>Example Component</p>

const App = () => {
  return (
    <Router>
      <Route path="/example" component={Example} />
      <Route path="/example" render={() => <Example />} />
    </Router>
  );
}

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
import { BrowserRouter as Router, Route } from 'react-router-dom';

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
}

ReactDOM.render(<App />, document.getElementById("root"));
```

While we avoided duplication of routing in this case, we have created duplication of the layout code. One should probably abstract this structure in its own layout component.

```jsx
import React from "react";

const Layout = props => (
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
}

ReactDOM.render(<App />, document.getElementById("root"));
```

If you have tried these code examples on your own, you might have noticed something that strikes you as a little odd. **React Router** intentionally has a rather relaxed approach to path matching. When we hit the `/account` URL, we do not only render the `Account` component, but also the Home component as the `/account` url **also includes** the path of `/` which then renders both components. This is intentional as it allows us to group certain areas of the page under a certain URL prefix and have all components render on these types of routes.

Imagine a user account and a sidebar within this user account. Let's assume that we are building a community which has a user area which is divided into different subcategories: `/account/edit` to edit your profile, `/account/images` to view your own pictures or `/account/settings` to change your account settings. We can use a generic route in this application:

```jsx
<Route path="/account" component={AccountSidebar} />
```

This `AccountSidebar` component would now be rendered on every subpage that is part of the `/account` area, as all their URLs would also include `/account`.

### Limit matching with props

In order to limit the matching between `path` and the URL, React Router provides with an `exact` prop on the `Route` component. If this boolean prop is provided, the route is only rendered if the path prop exactly matches the current URL.

```jsx
<Route exact path="/" component={Home} />
```

Where exactly you place this prop in **JSX**, is not important. I like to place it just before the path prop to let it speak for itself: "Here is a route that matches an **exact path.**"  If we included the `exact` prop in the Account Sidebar, the sidebar would only be rendered if the URL with `/account` was hit, and would not register the components for `/account/edit`, `/account/images` oder `/account/settings`.

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

If we were to only allow numerical values in the :userid, we would define routes such as `/users/:userid(\d*)` or `/users/:userid([0-9]*)` to limit these. A URL of `/users/123` would lead to a render of the `UserProfile` component, while `/users/abc` would not.

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

