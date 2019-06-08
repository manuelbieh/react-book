# Routing

A functionality that is needed in almost all **Single Page Applications** \(SPA\) sooner or later \(mostly sooner than later\) is **Routing**. This means assigning a URL in the application to a specific function. If, for example, I call the URL `/users/manuel`, I would very probably like to display the user profile of the user `manuel` there.

Here, the **React Router** has established itself as the de facto standard in recent years. Developed by Michael Jackson \(yes, the guy's really called that!\) and Ryan Florence \(who says he's developed more than 10 different routers for various purposes\) he brings it up to more than 35.000 stars at GitHub. It is regularly maintained, has a community of over 500 contributors on GitHub and adapts wonderfully to the React principles through its declarative nature. In addition, it is compatible with both the Web \(client and server side!\) and React Native. It is therefore very universally applicable, very well tested and proven due to its wide distribution.

Its interface itself is quite simple. In about 95% of the time you will come into contact with only five components: `BrowserRouter`, `Link`, `Route`, `Redirect` and `Switch`. There is also the imperative History API, which is based on the `history` package of the same name, and the HOC `withRouter` to pass data relevant for routing from the router into a component. 

It is installed via:

```bash
npm install --save react-router-dome
```

or

```bash
yarn add react-router-dome
```

The general use is _deklarativ_ as already mentioned, i.e. it takes place in the form of the components mentioned above. This allows routers to be used anywhere within an application. The only requirement is that the respective page tree is located in a **Router Context**. This exists only once in a typical application and usually surrounds the application on the outside. About the following form:

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

### Define routes

Every component within the `<router></router>` element can now access, react to and control the **router context**. We create different routes by using the route component, which should contain a `path`-prop \(exception: 404 error routes\) and optionally a `render`-prop or a `component`-prop. The difference here is that the value of the `render` prop must be a **function** that returns a valid **react element** \(remember the corresponding chapter on **render props**), while the `component` prop expects a **component** \(no _element!_\). 

So a correct use of both props looks like this:

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

In this example, the `Example` component would be rendered twice when the `/example` URL is called, since the Route component only checks if the path of the current URL matches the value specified in the `path` prop. This may sound astonishing at first, but it can be explained quite logically.

Since React Router is just declarative, we say React with the specification of two equal routes first of all that we want to render also two components if the URL matches. This can be useful if different parts of a page are to change independently of each other and based on the URL. Suppose we have an app with a sidebar and a content area. Both should now react to a URL:

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

In this example we see how different components are rendered in two different places in our application, depending on which URL is currently called.

Here, however, we are quite free in the structure of the components. And so it is not unusual that such a duplication of routes is avoided by a slightly different structure. The above example would be written like this:

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

Here we have now avoided double routing, but to the disadvantage of the layout structure, which has now been duplicated. In a typical application we could now go here and abstract the structure into its own layout component:

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

If you try the code examples now, you will notice a behavior that is unwanted in most cases: **React Router** is very easy and casual with path matching paths. And so when we call the `/account` URL, we see not only the `Account` component, but also the `Home` component. This happens because `/account` **also****contains the path `/` ****, so both components are rendered. This is quite intentional, e.g. it would be possible to structure individual page areas under a certain URL prefix and render a component on each of these routes.

Let's think of a user account and imagine a sidebar. Maybe we're developing a community right now. There is a user area divided into several subcategories: `/account/edit` to edit your profile, `/account/images` to view your images or `/account/settings` to make changes to your settings. We can now insert a generic route into our application:

```jsx
<Route path="/account" component={AccountSidebar} />
```

The `AccountSidebar` component would now be displayed on every subpage within the account area as long as the URL starts with `/account`.

### Restrict matching via Prop

To limit the matching between the `path` and the URL, React Router offers us the `exact` prop on the `route` component. If this Boolean-Prop is specified, a route will only be rendered if its `path`-Prop matches the current URL exactly:

```jsx
<Route exact path="/" component={Home} />
```

The place where the prop is exactly indicated is negligible, as always in **JSX**; I like to write it directly in front of the `path` prop to give the impression of a "talking" prop: here I have a **route** with the **exact path**. In our example with the account sidebar, the sidebar would only be rendered when using the `exact` prop if the URL corresponds to `/account`, but no longer with `/account/edit`, `/account/images` or `/account/settings`.

### Limit matching to a route via switch component

The `exact` prop always refers to only one **individual route** and leaves other routes completely unaffected. If we have a number of URLs, some of which can match multiple routes in certain cases, it can be difficult to add another prop to each of these routes. Here the next import from the **React Router** packet helps us: `Switch`.

With the `Switch` component, which wraps around a series of `<route />` elements, we make sure that only the **first** route whose `path` matches the current URL is rendered. Often it's not a bad idea to always package routes in one `switch` element unless you explicitly want multiple routes to be rendered. Instead of using the `exact` prop as in the above example, you could also use the `Switch` component:

```jsx
<Router>
  <Switch>
    <Route path="/account" component={Account} />
    <Route path="/" component={Home} />
  </Switch>
</Router>
```

When calling the `/account`-URL the first route would apply and all following routes would be ignored, so we would only render the `account` component. The `Switch` component always only checks its **direct** child elements for matching with the URL. If the `Account` component again contains its own routes, which is possible without problems, these are not impressed by the Switch component and are rendered if their `path` matches the current URL.

This also allows us to create a 404 error page as a fallback route. If we omit the `path` prop, this means that this route applies to **any** URL. So if we use them within a `Switch` element as the last component, we tell the **React Router**: _Render this component whenever no other route applies - and only then!_ 

And it'll look like this:

```jsx
const Error404 = () => <h1>404 - Page not found</h1>;

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

In this case, in addition to the `switch` element, an `exact` prop is also required for the `/` route, because otherwise this would always apply, if none of the other routes applies to the current URL, because the router works in such a way that, for example, `/exists-not` would be found under the `/` route. With the `exact` prop on the `Home` route this doesn't happen and instead we see our `Error404` component intercepting all calls at the end of our `Switch` block. Comparable to the `default` case in a `switch` statement in JavaScript.

### Parameter in URLs

Hardly any application can do without URLs that contain parameters. This case is of course also covered by the React Router and in a form that might seem familiar to anyone who has ever worked with other routing mechanisms, namely by using a parameter name preceded by a colon \(`:`\). 

```jsx
<Route path="/users/:userid" component={UserProfile} />
```

Here you can restrict what exactly is to be recognized as a parameter of the route. If, for example, I want to limit the sorting on a route to _ascending_ \(asc\) or _descending_ \(desc\), I can do this with a regular expression in brackets, which I append directly to the parameter:

```jsx
<Route path="/products/:order(asc|dec)"
```

The above route would only apply if the URL is `/products/asc` or `/products/desc`.

If I want in the first example that only numeric values are allowed as `:userid`, I can define the route: `/users/:userid(\d*)` or `/users/:userid([0-9]*)` and so the URL `/users/123` would render the `UserProfile` component, `/users/abc` not.

If the **React Router** finds such a URL with a parameter, it extracts its value and passes it in a `match` prop to the rendered component.

### Control forwarding of certain routes

In addition to the `Route` component to respond to specific routes, React Router also provides a `Redirect` component. This contains a `to` pop with which a destination can be specified and is intended to be used to decide declaratively \(i.e. in **JSX**\) where a user is redirected in certain situations. Whenever a `Redirect` component is rendered with only one `to` prop, a corresponding redirect to the URL specified in the `to` prop is executed.

A common use case for a `Redirect` component is, for example, redirecting to a login page for logged-in users if they are not yet logged in:

```jsx
<Route
  exact
  path="/"
  render={() => {
    return isLoggedIn ? <Dashboard /> : <Redirect to="/login" />;
  }}
/>
```

Here we use the `render` pop of the `Route` component to query in a function whether a user is logged in \(using `isLoggedIn`\) and show on the `/` URL either a dashboard component, or render a redirect to `/login`, which would then redirect the user to a login page.

A second way to define redirects is to use the `Redirect` component within a `<Switch/>` element. Here it behaves like the `Route` component and only applies if another route or redirect has not already matched the current URL. 

If the `Redirect` component is used in a `<Switch/>` element \(and only then!\) it can also get a `from` prop. This corresponds to the `path` prop for the `route` component, so that the redirect is only performed if the current URL corresponds to the value of the `from` prop:

```jsx
<Switch>
  <Redirect from="/old" to="/new" />
  <Route path="/new" component={NewComponent} />
</Switch>
```

The `Redirect` component behaves exactly like the `Route` component when it comes to matching URLs. It also supports `exact` prop to force exact matching and it also supports redirection to other routes with parameters:

```jsx
<Switch>
  <Redirect from="/users/:userid" to="/users/profile/:userid" />
  <Route path="/users/profile/:userid" component={UserProfile} />
</Switch>
```

When the URL `/old` \(first example\) or `/users/123` \(second example\) is called, the user is then redirected to the URL specified as `to`-Prop.

### Using the Router Props

Each component rendered by the React Router because it was specified as the `component` prop of a `route` component is automatically passed three routing relevant props:

* `match` 
* `location` 
* `history`

These can be accessed within the respective component like any other prop via `this.props` in **class components** or `props` in **function components**:

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

If we take a look at the console output of this component, we see a result corresponding to the URL `/users/123` \(abbreviated\):

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

Without going into the details of each individual property right now, we get an impression here of which properties of the router we can access here. For now we are interested in the `match` property. If the URL in `match.params` is correct, this contains the parameters we defined in the `path` along with their values. In this case `match.params.userid` with the value `123`.

In a user profile component, for example, we could go here and start an API request to get all relevant data for the user with the user ID `123` and display the profile view of this user.

**React Router** ensures that the `match` property always exists in every component connected to the router and that it always has a `params` property that either contains the parameters \(if any\) or is an empty object. It is therefore ** safe,** to access `props.match.params` without having to fear that one of these properties is `undefined` and therefore throws an error.

Also the `render` prop on the `route` component gets all props of the router passed:

```jsx
<Route path="/users/:userid" render={(props) => {
  return <p>User profile for the ID {props.match.params.userid}</p>;
}} />
```

### Navigation between routes

Once we have divided an application into several routes, we naturally want to link between these URLs as well. Of course this would work with the HTML element `<a href="...">...</a>`, no question. However, we trigger a completely new "hard" page call in the browser, leave the current page **completely** and call the new page completely new. 

This means that we request an HTML document, the HTML document loads CSS and JavaScript from the server again with our React application \(or ideally gets it from the browser cache\), reinitializes everything and decides which route to render based on the changed URL. Any state we previously set globally would be reset. 

But this is not the behavior we want in a single page application. Here we want to load the HTML together with its CSS and JavaScript only once from the server. We want to persist the global state when navigating between routes and only render those parts of the page that should change based on the route.

For this **React Router** brings us the `Link` component. This is imported like all other components from the `react-router-dom` package and has a `to` prop, which can be roughly compared to the `href` attribute of HTML `a` elements:

```jsx
<Link to="/account">Account</Link>
```

Internally, **React Router** also uses a common `<a href />` to display these links, but clicks on these links are intercepted and forwarded to an internal function that then takes care of displaying new page content based on the new URL without triggering a full page load.

In addition to the `to` prop, a `link` can also have an `innerRef` that can get a ref created via `createRef()` or `useRef()` and a `replace` prop that lets us specify that we want to _replace_ the current URL in the browser history instead of creating a new history entry. If you click on the Back button in the browser, you can no longer return to the previous route.

All other props passed to the `<link />` element are passed to the created `a` element. So `<Link to="/" title="Homepage">Home</Link>` would create the following markup in the output accordingly: `<a href="/"" title="Homepage">Home</a>`.

### special edition: NavLink

A special characteristic of the `Link` element is the `NavLink`. In addition to the props, which the `Link` component can also receive, the `NavLink` also knows its current state in the sense that it knows whether it is currently linking to the current page. If this is the case, it can be given an alternative appearance using `activeClassName` and `activeStyle`.

A classic example here is a page navigation where the current menu item is highlighted in color.

```jsx
<NavLink to="/" activeClassName="active">Home</NavLink>
<NavLink to="/account" activeClassName="active">Account</NavLink>
<NavLink to="/contacts" activeClassName="active">Contacts</NavLink>
```

Here the link of the current page, and only this one, would get the class `active`. If we are e.g. on the `/account` URL, the markup would be accordingly:

```markup
<a href="/">Home</a>
<a href="/account" class="active">Account</a>
<a href="/contacts">Contacts</a>
```

Further **Props,** which can contain a `NavLink` element are `exact` and `strict`, analogous to the corresponding **Props** in the `Route` component, as well as `isActive`. The latter expects a function that returns either `true` \(active page corresponds to `NavLink`\) or `false` \(active page does not correspond to \). The function itself gets the `match` object as the first argument from the router and the `location` object as the second argument. The function can now use this information to determine whether it marks the respective `NavLink` as active or not.

### Navigating programmatically with the History API

Now we know how to use `Route` elements to react to different URLs with the rendering of corresponding components and how to avoid a hard pageload by using the `<Link />` element. But in some cases it is necessary to force a change of URL programmatically. For example, to redirect the user to another page after an asynchronous request was successful.

The `history` property, which the router passes to all components used as routes in the `props`, serves this purpose:

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

Of particular interest here are the two functions `push()` and `replace()`. With `props.history.push('/ziel')` we can change the URL in the browser to `/ziel` and trigger a rendering. This creates a new entry in the browser history, just like when using the `Link` component. If we don't want to create a new entry in the browser history, we can use the function `props.history.replace('/ziel')` instead.

The function `props.history.go()` allows us to programmatically scroll forward and backward in the browser history. The function expects an integer as parameter, which specifies how many steps should be scrolled in the history. Negative values scroll back while positive values scroll forward. The two functions `goBack()` and `goForward()` are shortcuts that scroll back and forth one step in the history. They therefore correspond to `go(-1)` or `go(1)`.

The `action` property tells us by which action a user has landed on the current route. It is either `POP`, `PUSH` or `REPLACE`, where `POP` can mean that it is the initial page call or the user has pressed the back button in the browser, `PUSH` means that the `history.push()` function has been called, which is also the case when clicking on a `<Link />`. If the value of the `action` property is `REPLACE` - you can imagine it - `history.replace()` was called or a `<Link />` element was clicked that has a `replace` prop.

### Connect components to the router via the HOC

Any component used as a `component`-prop within a `<router />`-element gets the router props \(`history`, `location`, `match`\) automatically from it. But sometimes we also want to access router functionality in components that are not used as a direct route. For example, because we want to redirect to another page and want to use `history.push()` for that.

For this purpose **React Router** provides the `withRouter` Higher Order Component. Components that are not used directly as router components are then also given the three props of the router:

```jsx
withRouter(MyComponent);
```

But the component also fulfills another purpose, which, however, has more of a workaround character and is fixed as of **React Router 5.0.0****: It removes the so-called **"Update-Blocking "**.

In some cases, components may be implemented as `PureComponents` or optimized by a `React.memo()` in order to optimize performance, and thus prevent rendering if neither the **Props** nor the **State** of the respective component change. Since most router components such as `NavLink` access the router's data through the **React Context**, such a component might not know that a child element needs to be re-rendered.

The solution to this problem in such a case is to pack the corresponding component into a `withRouter()` HOC, which then passes the new props with the new location to the respective component each time the routing is changed, thus causing a rendering in this component. This is, for example, the case when used with the State Management Library **Redux**. If a component is connected to the Redux store via the `connect()` function in Redux, this component prevents the rendering of router-specific parts unless something changes in the store at the same time.

In this case it helps to wrap the component with a `withRouter()` call:

```javascript
withRouter(connect()(MyComponent));
```

This has been fixed by using the new **Context API** from **React 16.3.0** in **React Router** from version **5.0.0** and is therefore primarily relevant for projects where older versions are used.
