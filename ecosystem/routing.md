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

In order to limit the matching between `path` and the URL, React Router provides an `exact` prop on the `Route` component. If this Boolean prop is provided, the route is only rendered if the path prop exactly matches the current URL.

```jsx
<Route exact path="/" component={Home} />
```

Where exactly you place this prop in **JSX**, is not important. I like to place it just before the path prop to let it speak for itself: "Here is a route that matches an **exact path.**"  If we included the `exact` prop in the Account Sidebar, the sidebar would only be rendered if the URL with `/account` was hit, and would not register the components for `/account/edit`, `/account/images` or `/account/settings`.

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

* `match` 
* `location` 
* `history`

Each of these props can be accessed just like any other props. **Class components** can access these via `this.props` whereas **function components** can access these with `props`:

```jsx
import React from "react";
import ReactDOM from "react-dom";
import { BrowserRouter as Router, Route } from "react-router-dom";

const Example = (props) => {
  console.log(props);
  return <p>Example</p>;
}

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
<Route path="/users/:userid" render={(props) => {
  return <p>Profile of ID {props.match.params.userid}</p>;
}} />
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

A special form of the `Link` element is the `NavLink` element. Apart from the usual props that can also be received by the `Link` component, `NavLinks` can change based on their state.  `NavLinks` can access information relating to which page they are currently linking to and whether that page is the same as the current page. If this is the case, we can alter it's display using `activeClassName` and `activeStyle`.

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

