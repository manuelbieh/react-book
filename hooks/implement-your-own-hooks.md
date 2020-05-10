# Implement your own Hooks

Apart from the **internal hooks** such as `useState` or `useEffect`, it is also possible to create our own **custom hooks**. These in turn can use **internal hooks** or other **custom hooks**, and encapsulate logic in a reusable form. Creating **custom hooks** can be really useful if you want to use the same logic in multiple components. Even if the complexity of the component's logic grows, it can be useful to **divide** it and break it up into **custom hooks** with meaningful names in order to keep the current **Function component** more manageable and readable.

### Your first custom Hook

Let's start with a relatively simple example and assume that we want to create a **custom hook** with which we can invoke Side Effects and change the background-color of a component whenever it is mounted. A typical name for such a **custom hook** would be `useBackgroundColor()` — remember that hooks always need to start with `use`. This hook expects a valid CSS color and then applies the color as the current background color. It is applied as soon as a component using it is mounted or passes a new value to the **custom hook**.

As we might want to use this logic in other components and do not want to repeat ourselves every time and implement the same functionality with `useEffect` again and again, it is worth extracting our **first custom hook**:

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

To use this component, we are creating a little `Tabs` component which allows us to select content based on three buttons. Depending on which component is currently shown to us, we would like to change the background color of our application. Thus, we can use our `useBackgroundColor()` hook:

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

We've implemented three basic components in this example which help us to display our content: `DefaultContent`, `SpecialContent` and `OtherSpecialContent`. Two of these components already use our **custom hook** `useBackgroundColor()` to change the global background color in a `useEffect()` once the component has mounted.

Alternatively, we could have implemented the `useEffect()` hook manually in each component that needs to change its background color. However, this would have led to a lot of code duplication. Instead, we are much better off extracting this logic into its own custom hook and making it configurable by passing the required color as an argument. This can then be used in as many **Function components** as we like.

While JSX allows us to create reusable components for the user interface, hooks offer us the opportunity to reuse logic across components without having to make any compromises.

### Working with data in Hooks

Passing data via **Custom hooks** is no one-way street. In our first custom hook example we've seen that we can pass data into a **Custom hook** as a function parameter. The hook can also _return_ data that can then be used in the component. The form in which this data is returned from the **hook** is entirely at the discretion of the developer. You can easily return strings, tuples but also entire React components or elements or even a combination of all of them. You're free to choose so to say.

Let's assume that we want to access data from an API. We should parameterize the data that we want to access to make it easier to work with. Hooks can help us to access this data in this case \(it does not matter in which component it ends up being used\) and then return it to the component that needs it. In our next example, we will deal with user data from GitHub. Thus, a good name for our next **Custom hook** would be `useGitHubUserData`.

We pass a GitHub username to this hook and obtain an object with all the relevant information from the user. The hook itself deals with requesting the data from the GitHub API and will pass it to the component:

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

Once again, we've made use of the hooks that we have already encountered: `useEffect()` and `useState()`. We're using the state, `accountData`, to manage the GitHub user data inside of it. The effect is only ever executed if the username changes. Afterwards, we request the user data for this account from the GitHub API, wait for the response and then write the data of this response using `setAccountData()` into state. Finally, we pass the state `accountData` back to the component which has called the Hook in the first place.

We can now safely access the data from this component and use it as we see fit:

```jsx
// RepoInfo.js
import React from "react";
import ReactDOM from "react-dom";
import useGitHubAccountData from "./useGitHubAccountData";

const RepoInfo = () => {
  const accountData = useGitHubAccountData("manuelbieh");

  return (
    <p>
      GitHub user {accountData.name} has {accountData.public_repos}{" "}
      public repositories.
    </p>
  );
};

ReactDOM.render(<RepoInfo />, document.getElementById("root"));
```

At this point, we could implement further info based on `RepoInfo`. For example, we could implement a search for a particular GitHub Account instead of setting this beforehand. To achieve this, we can use `useState()` to create a new **state** in which we write the account that the user has provided and then pass this **state** to our **Custom hook**.

As our `useEffect()` hook contains the account name in the dependency array, it will be executed once a new user account has been passed in. This means that a new API request will be fired each time we make a change to the search field and will then request the data for the provided account:

```jsx
// RepoLookup.js
import React, { useState } from "react";
import ReactDOM from "react-dom";
import useGitHubAccountData from "./useGitHubAccountData";

const RepoLookup = () => {
  const [account, setAccount] = useState("");
  const accountData = useGitHubAccountData(account);

  return (
    <div>
      <input value={account} onChange={(e) => setAccount(e.target.value)} />
      {!account ? (
        <p>Please provide a GitHub username</p>
      ) : (
        <p>
          GitHub user {accountData.name} ({accountData.login}) has{" "}
          {accountData.public_repos} public repositories.
        </p>
      )}
    </div>
  );
};

ReactDOM.render(<RepoLookup />, document.getElementById("root"));
```

Side note: GitHub's public API only allows about 60 API requests per hour. If you feel that you want to access more and play around with these examples a bit more, you should create a GitHub API token and append it to the URL with `access_token=xyz`. The token can be generated here: [https://github.com/settings/tokens](https://github.com/settings/tokens).

