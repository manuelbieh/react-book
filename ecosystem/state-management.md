# State Management

The more an application grows in the size, the more its complexity and data grows. Managing **application state** becomes increasingly difficult and cumbersome to do. When do I pass props to which component and how? How do the props influence the state of my components and what happens if I change the state of a component?

React has introduced great tooling in the last few years to deal with this increase of complexity in application state. The **Context API** as well as the **useReducer** **hook** have been great additions to the React developer's toolbox and allow us to work more comfortably with complex data. However, in some cases it is still very difficult to keep track of all the different pieces of data and how they transform. To deal with this type of problem, external tools for **global state management** can be a great choice. The React ecosystem has seen a number of players enter this space giving us lots of libraries to choose from.

One of the more commonly known tools is **MobX** which describes itself as a tool _"Simple, scalable state management"_ solution. The other well-known tool in the industry is **Redux**. Redux was in part developed by Dan Abramov and Andrew Clark who are now part of the official React Core Team. It positions itself as _"A predictable state container for JavaScript apps"._ Redux is arguably the more dominating tool boasting with higher usage stats and downloads.

This chapter will dive deeper into **Redux.** This is due to a number of reasons: first of all, I have experience with **Redux** myself and have used in it a number of different projects. Additionally, my experience with Redux has been very pleasant. But those are not the only reasons. Redux still has around **4 million weekly installations**. Compared to **MobX** which still has a very respectable number of **200.000 installations** per week, Redux is the dominant force in the React ecosystem though.

Despite a number of voices which keep proclaiming Redux's death, the interest in **Redux** has remained steady over the years and the number of downloads is still growing. When the **Context API** was introduced with **React 16.3.0**, **Redux** was said to be obsolete. Once **React 16.8.0** introduced Hooks, more specifically the **useReducer hook**, the critics' voices were raised again. Despite all this criticism, the number of installations of Redux keep growing.

In fact, **Redux** makes use of **Context** and **Hooks** under the hood and uses these principles to optimize its own performance and simplify its API. Moreover, **Redux** has seen a great increase of custom add-ons and tools for their own ecosystem which is enhanced rather than replaced by the new functionality offered by React.

### Introduction to Redux

As mentioned earlier, **Redux** is a predictable state container. But what exactly does this mean? I want to add some back story here to answer this question properly. It's important to really understand the principles of Redux to avoid common errors and set yourself up for success.

**Redux** as a tool is based on the principles of **Flux Architecture**. This type of architecture has been developed at **Facebook** too with the aim to simplify development in client-side web applications. As is the case in **React**, **Flux** dictates a **unidirectional data flow**, meaning data is only ever flowing one way: an **action** \(could be triggered by a button click\) alters the **state**, and this **state change** will cause a re-render and thus allow further actions.

**Redux** works almost the same. However, state is managed only within one component - **globally**. In practice, this translates to all components gaining access to this state no matter where they are placed.

In order to use **Redux** in a particular project, we have to install it via the command line:

```bash
npm install redux react-redux
```

Or using Yarn:

```bash
yarn add redux react-redux
```

We have to install **two** packets: `redux` and `react-redux`. The `redux` package will install the main state management library whereas `react-redux` will install the so-called **bindings**. While the name might sound a little obscure, there is no actual magic involved. **Bindings** refer to **React components** which have been specifically optimized for usage with **Redux**, allowing us to use these out of the box.

While in theory **Redux** could be used as a stand-alone solution, we would then need to manage components rendering and data flow from the state container ourselves. While possible, it introduces a great deal of complexity which most of us do not want to deal with. `react-redux` is thus a sensible choice for most of us.

### Store, Actions and Reducers

I've just introduced another set of new terminology to you. Don't be scared if any of these sound unfamiliar to you now. We will look at all of them in turn and hopefully clear up any confusion you might have at the moment.

All data in **Redux** is managed in so-called **stores** which manage the **global state**. Theoretically, applications could have a number of different stores. In fact, **Flux architecture** even encourages the use of multiple stores. However, in most React applications using **Redux** we will only find a single store. This reduces complexity dramatically and also declares this _single_ **store** as the **single source of truth** for all of our data. The store also provides a number of methods which can be used to change \(`dispatch`\) the data currently stored in the store, read \(`getState)` the data from the store and react to changes \(`subscribe`\).  

The only way to change data in a store is to "_dispatch"_ an **action**. Once again, **Redux** has taken its inspiration from **Flux Architecture** and prescribes for these **actions** to be in **Flux Standard Action** \(FSA\) format. **FSAs** consist of a simple JavaScript object which always **has to have** a `type` property and _can_ contain a `payload`, `meta` and `error` properties. We are going to focus mostly on the `payload`. In 9 out of 10 cases, we are going to deal with a `payload` when we are _dispatching_ an **action**.

The **payload** describes the actual _content_ of an **action** and can take the form of data which is serializable in JSON format. This could include strings, booleans, numerical values or even complex arrays or objects.

Let's have a look at a typical **action** in **Redux**:

```javascript
{
  type: "SET_USER",
  payload: {
    id: "d929e553-7079-4309-8c7d-2d2db39922c6",
    name: "Manuel"
  }
}
```

Once an **action** has been _dispatched_ \(the **store** provides this `dispatch` method for us\), the current **state** as well as the **action** dispatched are passed to the **reducer**. The **reducer** is a **pure function** which we also already encountered in React. Its primary aim is to create a **new state** based on the **current state** and the action's `type` and `payload` properties. A little reminder: a **pure function** always creates the **same output**, given the **same input parameters** - no matter how many times it is being called. This behavior makes **Reducers** predictable and also easy to to test.

Example of a **reducer**:

```javascript
const reducer = (state, action) => {
  switch (action.type) {
    case "PLUS": {
      return state + action.payload;
    }
    case "MINUS": {
      return state - action.payload;
    }
    default: {
      return 0;
    }
  }
};
```

A **store** generally expects only a **single reducer**. **Redux** allows us to split the **reducer** function into many small little parts, making them more digestible and easy to read. A `combineReduers()` function then takes care of merging all these parts into one main **reducer** - the **root reducer**. When an action is _dispatched_, each **reducer** is called with the same parameters: **state** and the **action**. 

Each **reducer** reacts to the `type` property of an **action**. Due to this, a convention has emerged to extract all used types into variables with the same name which allow us to avoid typos. Why is that? Typos might be hard to spot \(e.g. `USER_ADDDED`\) without. On top of that, the JavaScript interpreter will throw an error if we tried to access a variable which is not defined, eliminating yet another source of error that's hard to track down. Thus, you often find the following code-blocks at the beginning of a file in **Redux** applications:

```javascript
const PLUS = 'PLUS';
const MINUS = 'MINUS';
```

This allow us to create some sort of coherence among the different **action types**.

### Creating a store

To create a store which will manage the global state, we habe to import the `createStore` function from the `redux` package. We can call it by passing it a **reducer** function. This function will return a store object to us which will contain all the methods necessary to interact with the store, namely `dispatch`, `getState` and `subscribe`. The latter two are not of the same importance when working with React, but I have mentioned them for the sake of completion. In React and Redux applications, the `react-redux` bindings components take care of the re-rendering of components if they are affected by a change of the **state**.

```javascript
import { createStore } from "redux";

const initialState = 0;

const counterReducer = (state = initialState, action) => {
  switch (action.type) {
    case "PLUS": {
      return state + (action.payload || 0);
    }
    case "MINUS": {
      return state - (action.payload || 0);
    }
    default: {
      return state;
    }
  }
};

const store = createStore(counterReducer);
```

We've just created our first simple store with a counter reducer. The `createStore` function only takes a single parameter in this example \(the `counterReducer`\) indicating that the initial `state` will be set to `undefined`. Thus, we have set the `initialState` as the standard parameter which equates to 0 in this example.

If we pass another parameter to the `createStore` function, this parameter would be passed as first state to our reducer function:

```javascript
const store = createStore(counterReducer, 3);
```

Instead of `undefined`, the initial value for our `state` parameter that is passed to the `counterReducer` would equate to `3`. The `initialState` would not be set and our counter would start counting at `3` instead of `0`.

The first initial **action** being _dispatched_ takes the form of `{type: '@@redux/INIT5.3.p.j.l.8'}`. Looking at our example, this means that the `default` case will get activated and that only the `state` passed will be returned \(which also equates to the `initialState`\).

This `default` case is important. If no other `case` of the `switch` statement can be fulfilled, the current `state` needs to be returned from the function to avoid unwanted side-effects. The `reducer` function is executed after **each** `dispatch` call and dictates the value of the **next state** by returning it from the function.

Calling `store.getState()` after initialization, we obtain an `initialState` of `0`:

```javascript
console.log(store.getState()); // 0
```

Let's try things out a bit and try **dispatch a few actions** to see how **state** reacts to these **actions**:

```javascript
store.dispatch({ type: "PLUS", payload: 2 });
console.log(store.getState()); // 2

store.dispatch({ type: "PLUS", payload: 1 });
console.log(store.getState()); // 3

store.dispatch({ type: "MINUS", payload: 2 });
console.log(store.getState()); // 1
```

We've _dispatched_ an **action** of `type` `PLUS` twice and of `type` `MINUS` once. A `payload` is also passed that indicates by how many numbers our last state is supposed to be incremented or decremented by. These operations result in the following **state mutations**:

```javascript
0 (initialer State) + 2 (payload) = 2 (neuer State)
2 (alter State)     + 1 (payload) = 3 (neuer State)
3 (alter State)     - 2 (payload) = 1 (neuer State)
```

This state remains relatively simple and only consists of a single value. We will look at more complex **state** consisting of different objects soon and create a number of different **reducers**.

