# Implement your own Hooks

Apart from the **internal Hooks** such as `useState` or `useEffect`, it is also possible to create your own **custom Hooks**. These in turn can then again use **internal Hooks** or other **custom Hooks** and encapsulate logic in any reusable form. Creating **custom Hooks** can be really useful if you want to use the same logic in multiple components. Even if the complexity of the component's logic grows, it can be useful to **divide** it and break it up into **custom Hooks** with meaningful names in order to keep the current **function component** more manageable and readable.

### Your first custom Hook

Let's start with a relatively simple example and assume that we want to create a **custom Hook** with which we can invoke Side Effects and change the background-color of a component whenever it is mounted. A typical name for such a **custom** **Hook** would be `useBackgroundColor()` - remember that Hooks always need to start with `use`. This Hook expects a valid CSS color and then applies the color as the current background color. It is applied as soon as a component using it is mounted or passes a  new value to the **custom Hook**.

As we might want to use this logic in other components and do not want to repeat ourselves every time and implement the same functionality with `useEffect` again and again, it is worth extracting our **first custom Hook**:

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

To use this component, we are creating a little `Tabs` component which allows us to select content based on three buttons. Depending on which component is currently shown to us, we would like to change the background color of our application. Thus, we can use our `useBackgroundColor()` Hook:

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

