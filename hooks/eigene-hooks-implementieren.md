# Implement your own hooks

Besides the **internal hooks** like `useState` or `useEffect` it is also possible to create **own hooks**, which in turn make use of the **internal hooks** or of other **own hooks** and encapsulate own logic of any kind in reusable form. So-called custom hooks. The development of such **Custom Hooks** is particularly useful if the same logic is to be used in several components. Even if the logic in a component becomes more and more complex, it can be useful to split it into **own hooks** with easy to understand names in order to keep the actual **Function Component** clearer.

### The first custom hook of your own

Let's start with a very simple example and assume that we want to implement a **custom hook** to trigger a side effect and change the background color whenever a component is mounted. A typical name for such a **custom hook** - we remember that hooks always have to start with `use` - would be `useBackgroundColor()`. The hook expects a valid CSS color and sets it as the background color when a component that makes use of this hook is mounted or passes a new value to the **custom hook**.

Since we might want to use this logic in several components and don't want to implement the same functionality of the same `useEffect` function every time, we create the following **Custom Hook** as a first finger exercise:

```javascript
// useBackgroundColor.js
import { useEffect } from 'react';

const useBackgroundColor = (color) => {
  useEffect(
    () => {
      document.body.style.backgroundColor = color;
      return () => {
        document.body.style.backgroundColor = '';
      }
    }, 
    [color]
  );
}

export default useBackgroundColor;
```

As an example for the usage we create a small `Tabs` component, which shows three buttons, which in turn show other content when clicking. Depending on which component is displayed, we want to change the background of our application. We use our newly created `useBackgroundColor()` Hook. 

The whole thing then turns out as follows:

```jsx
// Tabs.js
import React, { useState } from "react";
import ReactDOM from "react-dom";
import useBackgroundColor from "./useBackgroundColor";

const DefaultContent = () => {
  return <p>Default Content</p>;
};

const SpecialContent = () => {
  useBackgroundColor("red");
  return <p>Special Content</p>;
};

const OtherSpecialContent = () => {
  useBackgroundColor("orange");
  return <p>Other Special Content</p>;
};

const Tabs = () => {
  const [tab, setTab] = useState("home");

  return (
    <div className="tabs">
      <div className="tabBar">
        <button onClick={() => setTab("home")}>Home</button>
        <button onClick={() => setTab("special")}>Special</button>
        <button onClick={() => setTab("other")}>Other Special</button>
      </div>
      <div className="tabContent">
        {tab === "home" && <DefaultContent />}
        {tab === "special" && <SpecialContent />}
        {tab === "other" && <OtherSpecialContent />}
      </div>
    </div>
  );
};

ReactDOM.render(<Tabs/>, document.getElementById("root"));
```

In this example we have implemented three simple components that represent our content: `DefaultContent`, `SpecialContent`, `OtherSpecialContent`. Two of these components use our **Custom Hook** `useBackgroundColor()` created in the first step to change the global background color in a `useEffect()` hook when the component is mounted.

Alternatively, we could manually implement the `useEffect()`-Hook, which we use in the `useBackgroundColor()`-Hook, wherever we need to change the background color. But this would lead to a lot of duplication in the code. Instead, we store the corresponding logic in a separate hook function, make it conditionally configurable by passing the desired color and can then use it in any number of **Function Components**.

While the simple creation of reusable components at the display level \(i.e. in the user interface layer\) is the great strength of React and JSX, hooks at the logic level offer us for the first time a way to ensure reusability with React on-board resources without having to make compromises.

### Working with data from hooks

Data transfer in **Custom Hooks** is not a one-way street. In our first own hook we have seen how we pass data \(in this case a color\) to the **custom hook**. Namely as simple function parameters. However, the hook can also return data that can then be used in the component. The form in which data is returned from the **Hook** is entirely up to the developer. From a simple string over a tuple like the internal `useState()`-**Hook** up to whole React components or elements or even a mix of everything there are no limits to your own fantasy.

Suppose we want to access the data of an API. We want to make it parameterizable which exact data is accessed. The hook is designed to get us the data - whatever component we use it in - and then return it to the component that needs the data. In a real example this could be user data from GitHub. This will be our next own hook: `useGitHubUserData`. 

We pass a GitHub user name to the hook and get an object with all relevant information back to the user. The hook takes care of getting the data via the GitHub API and then passing it back to the component:

```jsx
// useGitHubAccountData.js
import { useEffect, useState } from "react";
import axios from "axios";

const useGitHubAccountData = (account) => {
  const [accountData, setAccountData] = useState({});

  useEffect(
    () => {
      if (!account) {
        return;
      }

      axios.get(`https://api.github.com/users/${account}`).then((response) => {
        setAccountData(response.data);
      });
    },
    [account]
  );

  return accountData;
};

export default useGitHubAccountData;
```

Again we use the already known hooks `useEffect()` and `useState()`. We use the state \(specifically `accountData`\) to manage the GitHub user data. We always execute the effect \(and only then\) if the user name passed changes. For the given username we get the user data via the GitHub API, wait for the response and write the data from the response via `setAccountData()` into the state. At the end of the hook, the `accountData` state is returned to the component that called the hook.

The data is now available in the component and can be used in any way:

```jsx
// RepoInfo.js
import React from "react";
import ReactDOM from "react-dom";
import useGitHubAccountData from "./useGitHubAccountData";

const RepoInfo = () => {
  const accountData = useGitHubAccountData("manual");

  return (
    <p>
      The GitHub user {accountData.name} owns {accountData.public_repos}{" "}
      public repositories.
    </p>
  );
};

ReactDOM.render(<RepoInfo />, document.getElementById("root"));
```

At this point we can now proceed based on the `RepoInfo` component and implement further functionality. Instead of a fixed GitHub account we might want to let the user search for an account. To do this, we create a new **State** with the `useState()` hook, in which we write the account entered by the user and pass this state to our **Custom Hook**.

Since the `useEffect()` hook has the account name as **Dependency**, this is always executed when a new account name is passed. This means that with every change in the search field, a new API request is initiated, which provides us with the data for the account entered:

```jsx
// RepoLookup.js
import React, { useState } from "react";
import ReactDOM from "react-dom";
import useGitHubAccountData from "./useGitHubAccountData";

const RepoLookup = () => {
  const [account, setAccount] = useState(""");
  const accountData = useGitHubAccountData(account);

  return (
    <div>
      <input value={account} onChange={(e) => setAccount(e.target.value)} />
      {!account ? (
        <p>Please enter a GitHub user name</p>
      ) : (
        <p>
          The GitHub user {accountData.name} ({accountData.login}) owns{" "}
          {accountData.public_repos} public repositories.
        </p>
      )}
    </div>
  );
};

ReactDOM.render(<RepoLookup />, document.getElementById("root"));
```

Brief info: The Public API of GitHub only allows a maximum of 60 API requests per hour. So if you want to try it out seriously you can create an API token and attach it to the URL via `access_token=xyz`. The token can be created here: [https://github.com/settings/tokens](https://github.com/settings/tokens).

