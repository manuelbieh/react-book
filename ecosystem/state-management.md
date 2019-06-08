# State Management

As an application grows in size, so does its complexity and the data it is designed to manage. In other words: The **Application State** becomes more difficult and confusing to manage. When do I add which props to which components and how? How do these props affect the state of my component and what happens if I modify the state in a component?

Even though React has become much better in recent years and has done a lot to improve the clarity of complex data structures with the new **Context API** and the **useReducer hook**, it is still not easy to keep an eye on all data and data transformations. To solve this problem, there are some external tools for **global state management** that have formed around React in the ecosystem.

One of the better known tools of this kind here is **MobX\*\***, which describes itself as _"Simple, scalable state management"_, i.e. as a tool for _simples and scalable state management_. On the other hand, we have **Redux**, undoubtedly the "top dog" in the React world, co-developed by Dan Abramov and Andrew Clark, among others, who now also belong to the official React team. Redux refers to itself as _"A predictable state container for JavaScript apps"_ i.e. as _"predictable state container for JavaScript applications"_.

This chapter will focus in particular on **Redux**. On the one hand, because I myself have worked with **Redux** in many projects and had very good experiences with it, on the other hand, because with \(according to npmjs.com\) about **4 million installations** per week, it has clearly arrived in the mainstream much more than **MobX**, with still respectable, but compared to **Redux** manageable **200,000 installations**.

**Redux** is still enjoying increasing popularity and has growing download numbers to record, although it has already been declared dead several times by some prophets. When the final **Context API** was released in **React 16.3.0**, it was said that **Redux** would become obsolete \(it didn't\), when **React 16.8.0** introduced the **Hooks** and especially the **useReducer-Hook**, which is very similar to **Redux**, there were these votes again.

In reality, this means that the number of installations continues to increase even after the introduction of **Context** and **Hooks** and **Redux** itself makes internal **use** of these new possibilities to improve performance on the one hand and simplify the use of its own API on the other. In addition, **Redux** has gathered an incredibly large ecosystem of its own addons and tools around it, which will not be replaced by the new functions added in React.

## Introduction to Redux

As described above, **Redux** is a _predictable state container_. But what exactly does that mean? At this point I would like to go a little further, since I consider it important to understand the basic principle in order to avoid mistakes that are made quickly afterwards, the principle behind **Redux** has not been internalized at least in very rough approaches.

First of all, with **Redux** we have a tool that is partly based on the principles of the **Flux architecture**. Like **React** itself, it was developed by **Facebook** to simplify the development of client-side web applications. The basic principle is **unidirectional data flow**, where data always flows **in one direction**. So the principle, as we already know it from **React** itself: an **Action** \(e.g. triggered by a button-click\) changes the **State**, the **State-Change** triggers a **Rerendering** and then allows to execute further actions.

This principle also works **Redux**, with the decisive difference that the state is managed **global** instead of just within one component, so that all components, no matter where they are in the page tree, can access **all data**.

Of course, to use Redux in a project, we must first install it again from the command line:

```bash
npm install redux react-redux
```

or with yarn:

```bash
yarn add redux react-redux
```

Here you can install **two** packages. The library `redux` itself and `react-redux`. While with the `redux` package the actual State Management Library is installed, with `react-redux` the so-called **Bindings** are installed. In plain English, this is simply a package with some **React components** that are specifically designed and optimized for using **Redux** with **React**. Not much magic.

Theoretically, it would also be possible to use **Redux** alone, but then we'd have to take care of when components are re-rendered and how data from a component gets in and out of the state container. Since we don't want that because someone else \(who knows it much better than all of us\) has already done it, we use `react-redux` in addition.

## Store, Actions and Reducer

Do not let the possibly still unknown terms unsettle you. We go through them one by one and at some point it clicks.

All data in **Redux** are located in a so-called **Store**, which takes care of the administration of the global **State**. Theoretically an application can have multiple stores, in the concept of Flux this is even explicitly provided for, but in **Redux** this is rather unusual in order to reduce complexity and so React applications using **Redux** are usually limited to only one _single_ **Store** as **Single Source of Truth**, so as _the_ only true source for all data. The store provides methods to modify the data contained in it \(`dispatch`\), to read \(`getState`\), and to react to changes \(`subscribe`\).

The only way to change data in a **Store** is to trigger \(_"dispatchen"_\) **Actions**. Again **Redux** is inspired by ideas from the Flux architecture and makes it necessary to keep these **Actions** in the format of the **Flux Standard Actions** \(FSA\). Such a **FSA** consists of a simple JavaScript object, which always **forcibly** has to have a `type` property _must_ and in addition the three other properties `payload`, `meta` and `error` have _can_, whereby we should primarily be interested in the `payload`, which we will have to deal with in 9 of 10 cases in which we have a **Action** _dispatchen_.

The **Payload** represents the _content_ of a **Action** and can contain any data from simple Boolean or String, over numeric values, up to complex arrays or objects, which are serializable, thus in form of a JSON representation can be stored.

Example of a typical **Action** in **Redux**:

```javascript
{
  type: "SET_USER",
  payload: {
    id: "d929e553-7079-4309-8c7d-2d2db39922c6",
    name: "Manuel"
  }
}
```

If a **Action** is triggered by the `dispatch` method provided by the **Store**, the **State** current at the time of the call is passed to the **Reducer** together with the triggered **Action**. A **Reducer** is a **pure function**, which we already know from React, and is used to create a **new state** from the **current state** and the respective action with its `type` and `payload` properties. Remember: A **pure Function** always generates the **same output** with **same input parameters**, no matter how often it is called. It is this behavior that makes them predictable and thus also easy to test.

Example for a **Reducer**:

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

A **Store** basically expects a single **Reducer** when it is created, but **Redux** with the `combineReducers()`-function offers the possibility to split this **Reducer**-function into as many small parts as you like to make it easier to understand and read, and then to put them together to a large, common **Reducer**, also called **Root-Reducer**. If an action is _dispatched_, each **Reducer** is called with the same parameters, namely the **State** and the **Action**. We will take a closer look at this method later in this chapter.

Since every **Reducer** reacts to the `type` property of a **Action**, it has become a bit of a convention to store all used types in variables of the same name, since a typo is not always immediately noticeable \(e.g. `USER_ADDDED`\), but the JavaScript interpreter makes an error when accessing an undefined variable. In apps where **Redux** is used, you will often find a code block of the following format at the beginning of a file:

```javascript
const PLUS = 'PLUS';
const MINUS = 'MINUS';
```

This is to ensure coherence in the **Action Types** used.

## Create a store

To create a store that will manage our global state, we import the `createStore` function from the `redux` package, call it and pass it a **Reducer** function. The function then returns the store object with methods for working with the store: `dispatch`, `getState`, and `subscribe`, the latter playing a minor role in working with React, since the components in the `react-redux` package are responsible for rendering components affected by a change to the **State**.

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

Here we have created a first and still very simple store with a counter-reducer. Since we only pass the first parameter to the `createStore` function here, the `counterReducer` function is called when initializing with `undefined` as the previous `state`, which is why the `initialState` is used as the default parameter. This corresponds here to the numeric value `0`.

If we pass a second parameter to the `createStore` function, it would be passed to the Reducer function as the first state during initialization:

```javascript
const store = createStore(counterReducer, 3);
```

Here would be the starting value for our `state` parameter passed to the `counterReducer` function `3` instead of `undefined`, the `initialState` default parameter would not step in and our counter would start counting at `3` instead of `0`.

The initial **Action** is a Redux internal action _dispatched_ with the form `{type: '@@redux/INIT5.3.p.j.l.8'}`. Therefore in our `switch` block the `default` case takes effect and simply returns the passed `state` \(which in this case corresponds to the `initialState`\).

Such a 'default' case is important. If there is no other `case` in the `switch`, the last state of the state must always be returned from the function to avoid unwanted side effects. The `Reducer` function is executed on **every** `dispatch` call and its return value always determines the **next state!**

If we call `store.getState()` right after the initialization, we get our `initialState` back: `0`:

```javascript
console.log(store.getState()); // 0
```

We can now play around, dispatch different **Actions** and see how our **State** reacts to the **Actions**:

```javascript
store.dispatch({ type: "PLUS", payload: 2 });
console.log(store.getState()); // 2

store.dispatch({ type: "PLUS", payload: 1 });
console.log(store.getState()); // 3

store.dispatch({ type: "MINUS", payload: 2 });
console.log(store.getState()); // 1
```

Here we _dispatchen_ twice an **Action** with the `type` `PLUS` and once an action of the `type` `MINUS`. We pass a `payload`, with which we specify by how many digits we want to count up or down the last state. Our state mutation takes place as follows:

```javascript
0 (initial state) + 2 (payload) = 2 (new state)
2 (old state) + 1 (payload) = 3 (new state)
3 (old state) - 2 (payload) = 1 (new state)
```

This state is still very simple and consists only of a single value. Later, we will look at how to create more complex **State** from different objects and with multiple **Reducers**.

## Action Creators vs. Actions

Anyone who reads articles about **Redux** or even looks at the official documentary will always be confronted with the terms **Action** and **Action Creator**. At first it was a bit difficult for me to understand the differences and I also know from others that it wasn't just me. To make matters worse, the terms are also occasionally used synonymously although they are not. Therefore I would like to briefly throw in no excursus at this point to distinguish the two terms **Action Creator** and **Action** from each other.

We already got to know the action upstairs. It is a simple and if possible _serializable_ **object**, with which we describe how we want to change the state. This always contains a `type` property and usually also a `payload`.

A **Action** _**Creator**_ is a **Function**, which finally returns a **Action** **Factory**, so to speak a **Factory**, which creates a **Action** _\(hence: \_Creator_\). Usually **Action Creators** are used to encapsulate **logic**, which is necessary to create a **Action**. Sometimes, however, they are simply used to abstract the rather cumbersome nature of **Actions** behind a memorable function. **Action Creators** are then called instead of the original **Action** as parameters of the `dispatch` method.

Typical **Action Creators** from our example above would look like this:

```javascript
const add = (number) => {
  return { type: 'PLUS', payload: number };
};

const subtract = (number) => {
  return { type: 'MINUS', payload: number };
}
```

Or if you are familiar with the **short hand notations** from **ES2015+**, you can shorten the whole thing even further:

```javascript
const add = (payload) => ({ type: 'PLUS', payload });
const subtract = (payload) => ({ type: 'MINUS', payload });
```

Then the corresponding **Action Creators** are called as parameters of the `dispatch` method instead of passing the **Actions** directly:

```javascript
store.dispatch(add(2));
store.dispatch(add(1));
store.dispatch(subtract(2));
```

If the **Action Creator** functions are named accordingly, this is clearly in favour of better comprehensibility of the code. **Action Creators** are an important puzzle piece in the Redux concept and are an indispensable part of our daily work, as they save us a lot of paperwork and repetition and thus also reduce potential sources of error such as typos in the `type` of an **Action** \(e.g. `PLSU` instead of `PLUS`\).

## Complex reducers

The previous examples were used to become familiar with the concept of **Actions** and **Reducers** and to understand how **Actions** are used to mutate the **Store** with the **Reducer**. Typically, the **State** in a React application consists of much more complex data and objects. So let's take a look at a store to see how it could look realistic in a small application.

As an example we want to use the state management for a fictitious little **Todo app**, which manages a list of todos as well as a logged-in user. So our state has the two top level properties `todos` \(of type Array\) and `user` \(of type Object\). We map this in our initial state in the same way:

```javascript
const initialState = Object.freeze({
  user: {},
  todos: [],
});
```

Additionally, since we want to make sure that a new state object is created with every call and not the old one is mutated, we enclose our initial state with an `Object.freeze()`. This ensures that we get a `TypeError` if the **State object** is mutated directly.

Let's take a look at what a **Reducer** function might look like, allowing us to set the logged in user, add and remove new todos, and change their status:

```javascript
const rootReducer = (state = initialState, action) => {
  switch (action.type) {
    case "SET_USER": {
      return {
        user: {
          name: action.payload.name,
          accessToken: action.payload.accessToken,
        },
        todos: state.todos,
      };
    }
    case "ADD_TODO": {
      return {
        user: state.user,
        todos: state.todos.concat({
          id: action.payload.id,
          text: action.payload.text,
          done: Boolean(action.payload.done),
        }),
      };
    }
    case "REMOVE_TODO": {
      return {
        user: state.user,
        todos: state.todos.filter((todo) => todo.id !== action.payload),
      };
    }
    case "CHANGE_TODO_STATUS": {
      return {
        user: state.user,
        todos: state.todos.map((todo) => {
          if (todo.id !== action.payload.id) {
            return todo;
          }
          return {
            ...todo,
            done: action.payload.done,
          };
        }),
      };
    }
    default: {
      return state;
    }
  }
};

const store = createStore(rootReducer);
```

I don't want to go into everything that happens here in detail, because this is about how a big, confusing **Reducer** can be divided into several smaller and ideally clearer ones. But some passages are not unimportant for the general understanding. So let's go through the `switch` block one by one, where each `case` block returns a new state object.

Starting with `SET_USER`: the **State object** created here changes the `user` object and sets its `name` property to `action.payload.name`, and the `accessToken` property to `action.payload.accessToken`. If you like, you can set `user` to `action.payload` instead, then all properties passed in **Payload** of **Action** would end up in the `user` object. However, you should make sure that the `action.payload` is always an object, because otherwise we would change the initial form of the `user` object, which can lead to problems if other parts of the **reducer** access the object and the object is suddenly no longer an object. In our example, however, we ignore all other properties by explicitly getting only `name` and `accessToken` from the **Payload** of the triggered action.

Besides the modified `user` we also return a `todos` property, which we set to `state.todos`, i.e. leave it at the previous value. **This is important** because in our state object the `todos` would otherwise be completely removed from the state and we would have set the user, but lost our todos from the state!

Continue with `ADD_TODO`: Here it is the other way around and we first return the `user` branch of our state tree unchanged. Then we add the new todo item to the `todo` array using the `.concat()` method. Here it is important to use `concat()` and not `push()`, since `push()` is a so-called _mutative_ method, i.e. it changes the existing state instead of creating a new one. Using `state.todos.concat` we take the current `todos` array as base and create a new array with the new todo-item and return it.

Very similar happens in the next case: `REMOVE_TODO`. Here we first return the `user` branch before searching for the entry to remove in the `todos` array and filtering it out. Which entry to remove is passed to the action as `action.payload` in the form of a todo-ID. The filtered array is then our new `todos` state. We use the `Array.filter()` method here, because it is not _mutative_ unlike e.g. `Array.splice()` and creates a new array.

Last we have the `CHANGE_TODO_STATUS` case. With this we change the status of the todo element, i.e. from _ToDo_ \(`false`\) to _Done_ \(`true`\) - or vice versa. We return the unchanged `user` object from the previous state and iterate through all todos using `state.todos.map()`. In the map function, we check whether the ID of the current todo object corresponds to the ID from the `action.payload`. If this is not the case, we simply return the respective todo element unchanged.

If the ID from the payload corresponds to the ID of the current todo element, we create a new object, write all properties with their respective values using ES2015+ spread syntax \(`{ ...todo }`\) into the newly created object and overwrite the `done` property with the new value from the action payload. We generate a new object here instead of simply overwriting the existing one, since we have to create a new state so that our **Reducer** remains a **Pure Function**. Using the `Array.map()` method already causes us to create a new array as well.

Here we are only dealing with two branches in our state tree: `user` and `todos` - and yet our **Reducer** function is already very long. With the increasing complexity of the state, the function becomes correspondingly longer and above all: **More error-prone. Since we always have to return all other parts besides the changed part of the state - for example the unchanged `user` if we modify the `todos` or the `todos` if we modify the `user` - the function quickly becomes confusing and difficult to manage. To make things easier, we could also use the** Object Spread Syntax\*\* from ES2015+, create a new state from the previous state and then overwrite the changed branch of the state tree. This could look like this using the example of the `ADD_TODO` case:

```javascript
case "ADD_TODO": {
  return {
    ...state,
    todos: state.todos.concat(action.payload),
  };
}
```

But even that doesn't make things much easier for us, as it can still happen quite quickly that we forget to return the old, unchanged part of the state.

For this reason, **Redux** provides the `combineReducer()` method. It makes it possible to divide our **Reducers** \(or the **State** they generate\\) into individual named sub-areas, which then each take care of a specific task and can be stored in their own files.

Starting from our example, here we have the two **Reducer-**functions `user` and `todos`. Both are in a separate file that exports the **Reducer-**function as `default`:

```javascript
// user/reducer.js
const initialState = Object.freeze({});

export default (state = initialState, action) => {
  switch (action.type) {
    case "SET_USER": {
      return {
        name: action.payload.name,
        accessToken: action.payload.accessToken,
      };
    }
    default: {
      return state;
    }
  }
};
```

```javascript
// todos/reducer.js
const initialState = Object.freeze([]);

export default (state = initialState, action) => {
  switch (action.type) {
    case "ADD_TODO": {
      return state.concat(action.payload);
    }
    case "REMOVE_TODO": {
      return state.filter((todo) => todo.id !== action.payload);
    }
    case "CHANGE_TODO_STATUS": {
      return state.map((todo) => {
        if (todo.id !== action.payload.id) {
          return todo;
        }
        return {
          ...todo,
          done: action.payload.done,
        };
      });
    }
    default: {
      return state;
    }
  }
};
```

If you take a closer look, you will notice that our large, confusing **Reducer** function has not only been divided into two smaller and much clearer functions. The functions themselves have also been simplified. Instead of returning the _unchanged_ part of the **State tree** from the **Reducer**, we only return the part of the state that is _relevant for the respective_ **Reducer**. In the **User-Reducer** this is also the **User**, in the **Todos-Reducer** this is correspondingly the **Todos**.

To reassemble our two _smaller_ **Reducers** into one _big_ **Reducer** function, which we can then pass to the `createStore()` function, we use `combineReducers()`. This function expects an object whose property names correspond to those of the created state tree. The values must be valid **reducers** themselves:

```javascript
import { combineReducers, createStore } from "redux";
import userReducer from './store/user/reducer';
import todosReducer from './store/todos/reducer';

const rootReducer = combineReducers({
  todos: todosReducer,
  user: userReducer,
});

const store = createStore(rootReducer);
```

The `combineReducers()`-function combines all reducers from the given object to a new function, which can now be passed as our **Root Reducer** to the `createStore()`-function. When an action is triggered, the generated function calls _each_ transferred reducer and then creates a new state object from its return values. This state object has the same form as the initially transferred object. So in our example from above the initial state is:

```javascript
{
  "todos": [],
  "User": {}
}
```

Tip: by cleverly using the ES2015+ Object Shorthand Notation, we can save a little code by naming the imports as the properties they will later represent in the state. So:

```javascript
import user from './store/user/reducer';
import todos from './store/todos/reducer';
```

The object passed to `combineReducers()` then shrinks to a clearer one:

```javascript
const rootReducer = combineReducers({ todos, user });
```

The use of `combineReducers()` is however bound to some formal rules, which, once internalized, are not really obstructive or even hard to remember. So every Reducer function passed to `combineReducers()` must meet the following criteria:

* For every unknown **Action** \(i.e. every **Action** whose `type` property we don't react to\) that a **Reducer** \(in its _second_ Argument\) gets passed, the `state` that the **Reducer** always gets as _first_ argument must be returned.
* Unlike manually generated, simple **Root Reducer**, a **Reducer** function used within `combineReducer()` must never return `undefined`. The `combineReducers()` function then throws a **Error** to point out this circumstance and not to unnecessarily complicate the search for the cause of the error by moving the error to another location. In our example we do this by returning in the `default` case within the `switch` block the `state`.
* If the `state` passed in the first argument is of type `undefined`, the **initial state** must be returned. The easiest way to do this is to set the initial state as the default value, as we did in the above example via `state = initialState`.

**By the way:** `combineReducer()` can also be "nested" arbitrarily. And so the **Reducer** functions passed to `combineReducer()` may have already been created by `combineReducer()` themselves. However, it should be noted that too finely granular division into ever smaller state branches does not necessarily make the code any more clearer at some point. In practice, I have personally worked with a maximum nesting depth of _one_ level \(i.e. a total of two `combineReducer()`calls, one inside, one outside\).

## Asynchronous actions

All **Actions** in previous examples were always executed **synchronously**. I.e. their **Action Creators** were executed whenever we wanted to modify the state without having to wait for the result of asynchronous processes. However, in dynamic web applications, where the strengths of **React** are most apparent, we regularly deal with **asynchronous data flows**, especially network requests. Here our _synchronous_ **Action Creator** functions don't really help us, because the `dispatch` method of a **Store** necessarily expects a **Action**, which, as we already learned, is a simple and simple object with a `type` property.

This is where the **Middleware** concept of **Redux** comes into play, and with it especially the **Redux Thunk Middleware**. But one thing at a time.

The `createStore` function from the `redux` package, which we used earlier to create different stores, processes up to three parameters:

* The **Reducer** function, which is the only parameter that must be specified **must** and, in connection with the triggered **Actions**, takes care of the mutation of our **State** by returning a **State** with each triggered **Action**.
* An **initial state**, which can already be filled with data when initializing the store, for example. This initial state is also transferred to the **Reducer** function when the store is initialized.
* A so-called **Enhancer** function \(for example: _Extension function_\), with which we can add our own functionality to our store. Like the **middleware** just mentioned. 

If the `createStore` function receives two parameters, it treats the second parameter as an **enhancer,** if the parameter is a **function**. Otherwise, the second parameter is passed as **initial state** to the **reducer** function.

A **Middleware** in **Redux** wraps itself around the `dispatch` method, intercepts calls to it, allows the called **Action** to be modified _before_ it is passed to the **Reducer**, and returns a new `dispatch` function at the end of its execution. For example, if we want to pass asynchronous functions or promises as parameters to the `dispatch()` method, we can use the **StoreEnhancer** to register **Middleware**, which allows us to do exactly that. The best known of this kind is said **Thunk Middleware**.

To the installation:

```bash
npm install redux-thunk
```

or Yarn:

```bash
yarn add redux-thunk
```

If the **Thunk Middleware** is installed, we have to _register_ it with the **Enhancer** using the **Redux** own `applyMiddleware()` function. To do this we import the **Middleware** and the `applyMiddleware()` function directly from **Redux** and pass each **Middleware,** we want to use as our own parameter. In our case this is first of all only `thunk`:

```javascript
import { applyMiddleware, createStore } from 'redux';
import thunk from 'redux-thunk';

// ...

const store = createStore(
  reducer, 
  applyMiddleware(thunk)
);
```

By including the **Thunk middleware** we can now write **Action Creator**, execute the _asynchronous_ code and _dispatch their **Actions** when a result is available. A **Thunk** function is a **Action Creator**, which itself returns a function whose two parameters are a `dispatch()` and `getState()` function. In the **Thunk Action Creator** function we can then decide for ourselves when the time has come to \_dispatch_ our action.

```javascript
const delayedAdd = (newTodo) => {
  return (dispatch, getState) => {
    setTimeout(() => {
      return dispatch({
        type: 'ADD_TODO',
        payload: newTodo,
      });
    }, 500);
  };
};

store.dispatch(delayedAction({
  id: 1,
  text: 'Explain Thunk Actions',
  done: false,
}));
```

In this example we create a `delayedAdd` **Action Creator**. This will get the new todo element and then return a new function in the form `(dispatch, getState) => {}`. The **Thunk middleware** then makes sure that the `dispatch()`- and `getState()`-functions are submitted to this function accordingly. After a delay of \(in this example\) 500 ms we call the `dispatch()` function with the `ADD_TODO`-**Action** and add the new object.

To _dispatch the action,_ we can now use the _asynchronous_ **Action Creator** just like we used to _dispatch our \_synchronous_ **Action Creators** _dispatched_ by passing the called function to the `dispatch()` function of the **Store**: `store.dispatch(ActionCreator)`. The thunk middleware then recognizes that it is a **thunk** function, executes it and passes the two arguments `dispatch` and `getState` into it.

If you are already familiar with the **Arrow Function Syntax** from ES2015, you can shorten the whole thing even further:

```javascript
const delayedAdd = (newTodo) => (dispatch, getState) => {
  setTimeout(() => {
    return dispatch({
      type: 'ADD_TODO',
      payload: newTodo,
    });
  }, 500);
};
```

Here we use the shortened **Arrow Function** with **implicit return value** to directly return the function called by the **Thunk middleware** without having to use the `return` keyword. This saves another two lines of code, but makes the code more difficult to understand, especially at the beginning.

## A typical example of asynchronous actions in practice

In many applications where interfaces \(APIs\) are used, it is a common pattern to notify the user of the load status as soon as data is retrieved from the API. E.g. via a graphical spinner or just by a text hint like _"Data will be loaded"_. Here a corresponding **Thunk Action** is excellently suited to cover this application with a corresponding **Reducer**.

To do this, we create three cases in the **Reducer** in which we react to the following **Actions**:

* `FETCH_REPOS_REQUEST` to reset errors from previous requests at the beginning of the network request and to initiate our load status,
* `FETCH_REPOS_SUCCESS`, which we call if the request is successful and which receives the result of the request and the date of the last update of this data, and
* `FETCH_REPOS_FAILURE`, which we use to react to errors that have occurred and, for example, set a `error` flag to inform the user that his request has failed.

In the form of code this could look like this:

```javascript
import { applyMiddleware, createStore } from 'redux';
import thunk from 'redux-thunk';
import axios from 'axios';

const initialState = Object.freeze({
  error: zero,
  items: [],
  isFetching: false,
  lastUpdated: zero,
  selectedAccount: 'manuelbieh',
});

const rootReducer = (state = initialState, action) => {
  switch (action.type) {
    case 'FETCH_REPOS_REQUEST': {
      return {
        ...state,
        isFetching: true,
        error: zero,
      };
    }
    case 'FETCH_REPOS_SUCCESS': {
      return {
        ...state,
        isFetching: false,
        items: action.payload.items,
        lastUpdated: action.payload.lastUpdated,
      };
    }
    case 'FETCH_REPOS_FAILED': {
      return {
        ...state,
        isFetching: false,
        error: action.payload,
      };
    }
    default: {
      return state;
    }
  }
};

const fetchGithubRepos = () => (dispatch, getState) => {
  dispatch({ type: 'FETCH_REPOS_REQUESTED' });
  const state = getState();
  axios
    .get(`https://api.github.com/users/${state.selectedAccount}/repos`)
    .then((response) => {
      dispatch({
        type: 'FETCH_REPOS_SUCCESS',
        payload: {
          lastUpdated: new Date(),
          items: response.data,
        },
      });
    })
    .catch((error) => {
      dispatch({
        type: 'FETCH_REPOS_FAILURE',
        error: true,
        payload: error.response.data,
      });
    });
};

const store = createStore(rootReducer, applyMiddleware(thunk));

store.dispatch(fetchGithubRepos());
```

_Dispatchen_ we here the `fetchGithubRepos()` **Action Creator,** the following happens:

First the **Thunk middleware** recognizes that it is not a simple **Action** \(an object\), but a function, executes it in the form `Action()(dispatch, getState)`. The **Action Creator** receives the `dispatch` function, in order to _dispatch_ itself **Actions** from the **Action Creator** function.

In the **Action Creator** _dispatchen_ we now first of all the `FETCH_REPOS_REQUESTED` Action. The **Reducer** reacts to the action, creates a new **State object** by copying the _existing_ **State** into a new object via **ES2015+ Spread Operator** and also resets any existing `error` to `null`. At the same time the state is informed by `isFetching` that a request will follow. This is a matter of taste here, so some prefer not to set the `error' property to`null\` until the pending request has actually been successful.

For the request we first get the _current_ **State** using `getState()`, from this we then get the selected account from `selectedAccount`, for which we then get the GitHub repos via the API request. We start the request \(and use Axios for simplification here as in previous examples\) and react to two possible cases:

* The request is successful and we get data from the GitHub API. We _dispatchen_ then the next **Action**, `FETCH_REPOS_SUCCESS`, pass the current time \(which we can use later e.g. for caching or automatic reloads\) as well as the array with the repos hidden in `response.data`. Since the request is also no longer active, we reset `isFetching` to `false`.
* The request is missing. In this case _dispatchen_ we will do the `FETCH_REPOS_FAILURE` action and pass the error message that Axios holds here in `error.response.data` as payload. Also here we reset `isFetching` back to `false`, because the request is finished here, too, even if with an unpleasant result for us, namely an error.

Our state now contains the GitHub repos of the user selected according to `state.selectedAccount` if the request was successful or an error message if it was not. In both cases we could now react accordingly in our user interface!

## Debugging with the Redux Devtools

To inspect the store we have several possibilities, e.g. there is a practical **logger middleware**, which logs the previous state, the action itself and the new state into the browser console for each dispatched **action**: [https://github.com/LogRocket/redux-logger](https://github.com/LogRocket/redux-logger)

Of course, we can also output the current state manually with `console.log(store.getState())` at any time, which is tedious and can sometimes be confusing, especially with asynchronous actions.

And then there's the Redux Devtools. These come as a browser extension and integrate seamlessly into the developer tools in Chrome and Firefox, even with future Edge versions, the use of **Redux Devtools** is possible once the browser engine has been finally converted to Chromium. The extensions can be found in the respective Add-On Stores:

* &lt;font color="\#ffff00"&gt;Chrome: [https://chrome.google.com/webstore/detail/redux-devtools/lmhkpmbekcpmknklioeibfkpmmfibljd](https://chrome.google.com/webstore/detail/redux-devtools/lmhkpmbekcpmknklioeibfkpmmfibljd)
* Firefox: [https://addons.mozilla.org/en-US/firefox/addon/reduxdevtools/](https://addons.mozilla.org/en-US/firefox/addon/reduxdevtools/)

Once the **Redux Devtools** are installed, the Developer Tools get a new tab in the browser **"Redux "**, where we can find them. But to use it, we need to take some precautions in our code. We need to add her as another enhancer to our store.

![Browser Devtools with installed Redux Addon](../.gitbook/assets/redux-devtools.png)

The **Redux Devtools** register themselves with two own global variables on the `window` object in the browser: `window.REDUX_DEVTOOLS_EXTENSION` and `window.REDUX_DEVTOOLS_EXTENSION_COMPOSE`. If we don't use our own store enhancers, like `applyMiddleware()` to register middlewares like Thunk, for example, the thing is quite simple: We use Logical AND Operator \(`&&`\) to see if the **Redux Devtools** are installed and, if so, pass a call to the `createStore()` function to the `window.REDUX_DEVTOOLS_EXTENSION`:

```javascript
createStore(
  rootReducer,
  window.__REDUX_DEVTOOLS_EXTENSION__ && window.__REDUX_DEVTOOLS_EXTENSION__()
);
```

Now we automatically see every **Action** triggered in the **Devtools**, can see in detail which **Action** was _dispatched_ with which **Payload** and can even manually **Actions** _dispatchen_. There is also the option of **Time Traveling**, i.e. "leafing back" to previous states of the store. This is sometimes very helpful for debugging.

Here it is particularly important that our **Reducers** are _Pure Functions_, so that they generate the same **State** as the original call each time they browse through the history of the **Store**. Otherwise you might run after a bug that is simply not reproducible, because it creates a different **State** with every call.

If, as in our last example, an Enhancer function is used, we use the `window.REDUX_DEVTOOLS_EXTENSION_COMPOSE` function instead. This function is a `compose` function that allows _several_ **Enhancer** functions to be combined into one _individual_ function, which is then called one after the other. So it's the same principle as the `combineReducers()`-method for **Reducer**.

With `compose`, Redux itself also offers such a function to combine several enhancer functions into a single one. For the sake of simplicity we import these to create a new `composeEnhancer()` function which now depends on one condition: If the Redux Devtools are installed, it uses the `REDUX_DEVTOOLS_EXTENSION_COMPOSE` function to add the **Devtools** to the **StoreEnhancer**. If they are not installed, we use Redux's own `compose()` function instead to create a function with the same signature:

```javascript
import { applyMiddleware, compose, createStore } from 'redux';
import thunk from 'thunk-middleware';

const composeEnhancers =
  typeof window === 'object' &&
  window.__REDUX_DEVTOOLS_EXTENSION_COMPOSE__ 
  ? window.__REDUX_DEVTOOLS_EXTENSION_COMPOSE__({}) 
  : compose;

// ...

const store = createStore(
  rootReducer, 
  composeEnhancers(
    applyMiddleware(thunk),
  )
);
```

If you feel overwhelmed by all the functions and concepts here, I can comfort you a little: In practice, it is normally not so relevant to understand and master all this. I myself use the **Redux Devtools** in every project in which I use **Redux** and I still have to look up every time how exactly this worked with the integration of the Devtools. This text in particular therefore serves primarily to raise awareness of how debugging **Redux stores** is possible and to provide a small guide for those who would like to know more about it.

## Using Redux with React

Now we already know pretty much how to create a new **Store**, how to **Actions** _dispatchen_, what role the **Reducer** plays and how to use **Middleware**. But how do we reconcile this with **React?**

This is where the `react-redux` package comes into play, which was already touched on at the beginning of the chapter. These are the _"official React Bindings for Redux"_, i.e. the official connection of **Redux** to **React**, which is maintained by the Redux developers and was originally developed by Dan Abramov, who is now also part of the official React Core Team.

The package essentially consists of only two React components, or a component and a function that creates a **Higher Order Component** \(plus another function that is used internally by React Redux itself, but is practically irrelevant in daily work\). First, there is the `Provider` component, which we place around the part of our component tree within which we later want to access the shared **Store**, and the `connect()` function, which returns a **Higher Order Component**, which we can use to _connect_ individual components to the store.

### The Provider Component

Since in most cases an application has only one **Store** and all components of this application should have access to exactly this store, the `Provider` component is usually used very high up in the component hierarchy; not seldom as the first and therefore highest component at all. The `Provider` component gets a **Redux Store** as `store` prop and also contains child elements. All their child elements then have the corresponding option to access the **Store** specified in the `store` prop, i.e. to read it and change it by triggering **Actions**:

```jsx
import React from 'react';
import ReactDOM from 'react-dom';
import { createStore } from 'redux';
import { Provider } from 'react-redux';

const dummyReducer = (state = {}, action) => {
  return state;
};

const store = createStore(dummyReducer);

const App = () => (
  <p>Here we now have access to the created Redux store</p>
);

ReactDOM.render(
  <Provider store={store}>
    <App />
  </Provider>,
  document.getElementById('root')
);
```

Instead of passing the `ReactDOM.render()`-method this time not only the `<App />`-element, but also the `Provider`-component, which gets the previously created \(dummy\)store passed to it, instead of most of the other examples in this book.

The `Provider` component can also be nested into each other. Components that are connected to the store then always use the **Store** of the _next higher_ `Provider` component. Such a procedure is however rather unusual and in such a situation, in which two stores should exist in parallel, one would probably rather connect the reducers of both stores via the `combineReducer()` function to a common store, in order to use finally again only one common `provider` element for all components.

### Connecting components to the store via the connect function

This was the easier exercise. The second part, namely the **connecting** of a React component with the **Redux Store** via said `connect()` function, becomes a bit more complicated. This can receive up to 4 parameters, of which the first 3 are functions, which can themselves receive up to 3 parameters. Oh, boy. The good news: in the vast majority of cases, in practice we need a maximum of 2 of the 4 parameters and for these two only the first parameter in each case. Let's work it out one by one, from simple to complex.

The basic function signature is the following:

```javascript
connect(mapStateToProps, mapDispatchToProps, mergeProps, options)
```

First of all, calling the `connect()` function creates a **Higher Order Component**. We can use this to pass certain parts of the state from the store to the component enclosed by it. To decide which part of the state to pass to the component, we use the first parameter, usually called the `mapStateToProps` function, as mentioned in the documentation.

### Access to parts of the global state via mapStateToProps

The `mapStateToProps` function gets as first parameter the complete **State** of Redux, as second, optional parameter the "own" props of the component, called `ownProps` in the documentation. Those that are passed to the generated HOC, if necessary. Depending on whether only one or two parameters are passed, the function is then only called if something changes in the **Redux State** _or_ even if the **Props** that are passed to the component change.

```javascript
const mapStateToProps = (state, ownProps) => {
  // ...
};
```

In both cases the function expects a **object** as return value. The properties of this object are then passed to the component as the so-called `stateProps`. Let's remember back to our todo store from a little further up. As an example, we now want to give a component the todos prefiltered by their status \(done or not done\), as well as the total number of todos.

Our `mapStateToProps` function looks like this:

```jsx
const mapStateToProps = (state) => {
  return {
    openTodos: state.todos.filter((todo) => todo.completed !== true),
    completedTodos: state.todos.filter((todo) => todo.completed === true),
    totalCount: state.todos.length,
  };
};
```

The properties of this object, i.e. `openTodos`, `completedTodos` and `totalCount` are then passed as **Props** to the enclosed component. This is done by passing the `mapStateToProps` function to the `connect()` function. This then returns an HOC to which we pass the component in which we want to access our three props from the state:

```jsx
const ConnectedTodoList = connect(mapStateToProps)(TodoList);
```

If we now use a `<ConnectedTodoList />` element in the **JSX** and this element is also inside a part of the application enclosed by the `Provider` component, the `TodoList` will be rendered with the above props from the global **Redux Store:**.

```jsx
import React from "react";
import ReactDOM from "react-dom";
import { combineReducers, createStore } from "redux";
import { Provider, connect } from "react-redux";
import user from "./store/user/reducer";
import todos from "./store/todos/reducer";

const rootReducer = combineReducers({ todos, user });

const store = createStore(rootReducer);

const TodoList = (props) => (
  <div>
    <p>
      {props.totalCount} Todos. Of which {props.completedTodos.length} completed 
      and {props.openTodos.length} still open.
    </p>
  </div>
);

const mapStateToProps = (state) => {
  return {
    openTodos: state.todos.filter((todo) => todo.completed !== true),
    completedTodos: state.todos.filter((todo) => todo.completed === true),
    totalCount: state.todos.length,
  };
};

const ConnectedTodoList = connect(mapStateToProps)(TodoList);

ReactDOM.render(
  <Provider store={store}>
    <ConnectedTodoList />
  </Provider>,
  document.getElementById("root")
);
```

Here we render a rather spartan `TodoList` component, which shows us for demonstration purposes only the number of all todos, as well as the number of open and already completed todos.

Using the second possible parameter of the `mapStateToProps` function, usually called `ownProps`, it is possible to access the **Props** of the component within the function, e.g. to decide which part of the state we want to pass into the connected component. For example, if we always want to return only the open todos _or_ the closed todos, and if the decision is to be based on a prop, the corresponding part could look like this:

```jsx
const mapStateToProps = (state, ownProps) => {
  const filteredTodos =
    ownProps.type === "completed"
      ? state.todos.filter((todo) => todo.completed === true)
      : state.todos.filter((todo) => todo.completed !== true);

  return {
    totalCount: state.todos.length,
    todos: filteredTodos,
  };
};

const ConnectedTodoList = connect(mapStateToProps)(TodoList);

ReactDOM.render(
  <Provider store={store}>
    <ConnectedTodoList type="completed" />
  </Provider>,
  document.getElementById("root")
);
```

Via `ownProps.type` we first look whether we want to display the completed todos or the open ones. Then we filter `state.todos` accordingly and return only the desired todos from the state. Since we no longer subdivide the passed todos by type, but already preselect them in the `mapStateToProps` function, we return the todos under a general `todos` property, so we can now access them via `props.todos` within the component.

Using the `mapStateToProps()` function we get read access to the complete **State** from our **Redux Store**. All data that we want to use in a component is returned here as an object. The corresponding React component will only be re-rendered if the relevant data in the store has actually changed. A rendering of the connected component is then triggered accordingly.

### Dispatching actions via mapDispatchToProps

Next is the second parameter for the `connect()` function: `mapDispatchToProps`:

```javascript
const mapDispatchToProps = (dispatch, ownProps) => {
  // ...
};
```

or alternatively:

```javascript
const mapDispatchToProps = {
  // ...
};
```

While we are accessing the store using `mapStateToProps` **reading**, `mapDispatchToProps` allows us to control the store by dispatching actions, **writing**. The function signature is quite similar to `mapStateToProps`, but the first parameter we get is not the state, but the `dispatch` method of the store we connect to. The second parameter also corresponds here to the `ownProps`, i.e. the **Props**, which are passed to the component itself. As a small feature it is possible to pass a `mapDispatchToProps`-**object** to the `connect()` call instead of a function. But more about that later. Let's first look at how the `mapDispatchToProps`-**function** is used.

For this purpose we want to extend our `TodoList` component by the possibility to add new todos, mark them as done or delete them completely from the list. The corresponding **Actions** are already provided in the example via **Reducer** earlier in this chapter: `ADD_TODO`, `REMOVE_TODO` and `CHANGE_TODO_STATUS`. Now we want to make it possible for a user who interacts with our application to trigger these **Actions**:

```javascript
// Helper function to create a (hopefully ;)) unique ID
const getPseudoRandomId = () =>
  Math.random()
    .toString(36)
    .slice(-6);

const mapDispatchToProps = (dispatch) => {
  return {
    addTodo: (text) =>
      dispatch({
        type: "ADD_TODO",
        payload: {
          id: getPseudoRandomId(),
          text,
        },
      }),
    removeTodo: (id) =>
      dispatch({
        type: "REMOVE_TODO",
        payload: id,
      }),
    changeStatus: (id, done) =>
      dispatch({
        type: "CHANGE_TODO_STATUS",
        payload: {
          id,
          done,
        },
      }),
  };
};
```

Here we return an object with the three properties `addTodo`, `removeTodo` and `changeStatus`, which is passed under exactly this name to a connected component, in our case to the `TodoList`, in its **Props**. We pass the `mapStateToProps()` function as a second parameter to `connect()`:

```javascript
const ConnectedTodoList = connect(
  mapStateToProps,
  mapDispatchToProps
)(TodoList);
```

The **Actions** that we pass inline here in `mapDispatchToProps` are usually extracted into corresponding **Action Creator** functions. These provide for better readability, are easier to test and normally also more understandable:

```javascript
const addTodo = (text) => ({
  type: "ADD_TODO",
  payload: {
    id: getPseudoRandomId(),
    text,
  },
});

const removeTodo = (id) => ({
  type: "REMOVE_TODO",
  payload: id,
});

const changeStatus = (id, done) => ({
  type: "CHANGE_TODO_STATUS",
  payload: {
    id,
    done,
  },
});
```

Our `mapStateToProps` function then shortens and becomes much clearer:

```javascript
const mapDispatchToProps = (dispatch) => {
  return {
    addTodo: (text) => dispatch(addTodo(text)),
    removeTodo: (id) => dispatch(removeTodo(id)),
    changeStatus: (id, done) => dispatch(changeStatus(id, done)),
  };
};
```

But this variant has another decisive advantage: Since we have a lot of repetition in the code and developers avoid repetitions as much as possible, Redux offers us a shortcut. If the function signatures of the **Action Creators** match those of the functions as we return them from `mapDispatchToProps`, we can return our **Action Creators** as **ES2015+ Shorthand Object**! **Redux** sets the necessary `dispatch()` call around all functions by itself.

With the following code we achieve the same functionality as with the code from the previous example:

```javascript
const mapDispatchToProps = {
  addTodo,
  removeTodo,
  changeStatus,
}
```

But beware: This only works if all **Action Creator** functions are called with the same functions from the connected React component and `mapDispatchToProps` is passed as object in exactly this form!

By using the two functions `mapStateToProps` and `mapDispatchToProps` you get a call which is similar to the following one:

```jsx
<TodoList todos={...} addTodo={...} removeTodo={...} changeStatus={...} />
```

All properties we return from `mapStateToProps` as well as those we return from `mapDispatchToProps` are passed to the component connected to the **Store** by the `connect()` function. In the component \(in the above example in the `TodoList` component\) we can then access it via the **Props** and by calling the functions from `mapDispatchToProps` **Actions** _dispatchen_ or by accessing the properties from `mapStateToProps` we can read the state from the **Store**.

If we just want to pass `mapDispatchToProps()` to the `connect()` call to be able to dispatch **Actions** from a component, but don't need to read the state itself in the component, we can pass `null` as the first parameter:

```javascript
const ConnectedTodoList = connect(
  zero,
  mapDispatchToProps
)(TodoList);
```

### Merging the StateProps and the DispatchProps with mergeProps

The third parameter covers a case that is extremely rare in practice. For the sake of completeness, I would therefore not like to leave it unmentioned at this point, but I would also not like to go into it too much in detail. This is the `mergeProps()` function:

```javascript
const mergeProps = (stateProps, dispatchProps, ownProps) => {
  // ...
}
```

The function gets as first parameter the result of `mapStateToProps` and `mapDispatchToProps`, as well as `ownProps` again. A new object is expected as the return value, whose properties are then also transferred to the component connected to the store via the props.

This function can be helpful if you want to dispatch certain **Actions** without using the **Thunk Middleware**, which depend on data from the state. It would also be possible to filter the **Actions**, based on the state, so that a component e.g. a possible `updateProfile()` **Action** is not submitted if `state.profile` does not exist, i.e. the user is not logged in. However, such conditions can usually be solved much more elegantly within the components themselves.

### Last: the options as fourth parameter for connect\(\)

If you are ready to need the 4th parameter, you should know pretty much what you are doing there. Redux is optimized by default so that the options only need to be used in very rare exceptional cases. For example, you can specify your own context that Redux should use, or you can pass your own comparison functions to determine whether a component should be re-rendered or not. The complete list of available options is the following:

```javascript
{
  context: Object,
  pure: boolean,
  areStatesEqual: Function,
  areOwnPropsEqual: Function,
  areStatePropsEqual: Function,
  areMergedPropsEqual: Function,
  forwardRef: boolean,
}
```

If you feel you need the options \(usually the answer is "no"\), take a look at the official documentary: [https://react-redux.js.org/api/connect\#options-object](https://react-redux.js.org/api/connect#options-object)

### How to connect all the pieces of the puzzle together

Now we know the purpose of the `provider` and how to use the `connect()` function. Let's take a look at a very comprehensive, but also complete example of a fully functional TodoList app, with which we can add new todos, mark them as done or not done and remove them again:

```javascript
// store/todos/reducer.js
const initialState = Object.freeze([]);

export default (state = initialState, action) => {
  switch (action.type) {
    case 'ADD_TODO': {
      return state.concat(action.payload);
    }
    case 'REMOVE_TODO': {
      return state.filter((todo) => todo.id !== action.payload);
    }
    case 'CHANGE_TODO_STATUS': {
      return state.map((todo) => {
        if (todo.id !== action.payload.id) {
          return todo;
        }
        return {
          ...todo,
          done: action.payload.done,
        };
      });
    }
    default: {
      return state;
    }
  }
};
```

```javascript
// store/todos/actions.js
const getPseudoRandomId = () =>
  Math.random()
    .toString(36)
    .slice(-6);

export const addTodo = (text) => ({
  type: "ADD_TODO",
  payload: {
    id: getPseudoRandomId(),
    text,
  },
});

export const removeTodo = (id) => ({
  type: "REMOVE_TODO",
  payload: id,
});

export const changeStatus = (id, done) => ({
  type: "CHANGE_TODO_STATUS",
  payload: {
    id,
    done,
  },
});
```

```jsx
// TodoList.js
import React, { useState } from 'react';
import { connect } from 'react-redux';
import { addTodo, removeTodo, changeStatus } from './store/todos/actions';

const TodoList = (props) => {
  const [todoText, setTodoText] = useState('');

  return (
    <div>
      <p>{props.todos.length} Todos.</p>
      <ul>
        {props.todos.map((todo) => (
          <li key={todo.id}>
            <button
              type="button"
              onClick={() => {
                props.removeTodo(todo.id);
              }}>
              delete
            </button>
            <label
              style={{ textDecoration: todo.done ? line-through : 'none' }}>
              <input
                type="checkbox"
                name={todo.id}
                checked={Boolean(todo.done)}
                onChange={(e) => {
                  const { name, checked } = e.target;
                  props.changeStatus(name, checked);
                }}
              />
              {todo.text}
            </label>
          </li>
        ))}
      </ul>
      <input onChange={(e) => setTodoText(e.target.value)} value={todoText} />
      <button
        type="button"
        onClick={() => {
          props.addTodo(todoText);
          setTodoText('');
        }}>
        add
      </button>
    </div>
  );
};

const mapStateToProps = (state) => ({
  todos: state.todos,
});

const mapDispatchToProps = {
  addTodo,
  removeTodo,
  changeStatus,
};

export default connect(
  mapStateToProps,
  mapDispatchToProps
)(TodoList);
```

```jsx
// index.js
import React from 'react';
import ReactDOM from 'react-dom';
import { combineReducers, createStore } from 'redux';
import { Provider } from 'react-redux';
import todosReducer from './store/todos/reducer';
import TodoList from './TodoList';

const rootReducer = combineReducers({
  todos: todosReducer,
});

const store = createStore(rootReducer);

const App = () => (
  <Provider store={store}>
    <TodoList />
  </Provider>
);

ReactDOM.render(<App />, document.getElementById('root'));
```

Here we first define our `todosReducer`, as well as the three **Actions** `addTodo`, `removeTodo` and `changeStatus`, which should seem familiar to us from previous examples. For a better overview, we store both **Reducer** and **Actions** in our own files, which we put in our own subdirectory `./store/todos`.

{% hint style="warning" %}
Attention, controversial: There are always heated debates about the "correct" folder structure when splitting an application into several files. I myself have worked with many different structures and found the division by domain \(i.e. `todos`, `user`, `repositories`, ...\) and by type \(`actions`, `reducer`, ...\) to be the clearest. Others prefer to collect all actions in one folder `actions`, all reducers in one folder `reducer`. Still others avoid splitting actions and reducers into separate files.

There is no unique _correct_ or _false_ here. This depends on the one hand on the personal preferences, but on the other hand also to a certain extent on the other structure of the application, its size, its complexity and finally also on how and by whom the application is used on the developer side.
{% endhint %}

We also create a new file that will contain our `TodoList` component: `./TodoList.js`. With it we will immediately be able to display our todos, create new ones or remove them again, as well as mark individual todos as done. We connect the component to the store via `connect()`. Accordingly, we must also import our **Actions** into the component, which we pass to `connect()` in `mapDispatchToProps`. For better clarity we use the **Object Shorthand Syntax**:

```javascript
const mapDispatchToProps = {
  addTodo,
  removeTodo,
  changeStatus,
};
```

The functions are then automatically enclosed by a `dispatch` call to React Redux.

In `mapStateToProps` we specify that we want to pass the `todos` branch of our store to the component. We then pass both `mapStateToProps` and `mapDispatchToProps` to the `connect()` function:

```javascript
connect
  mapStateToProps,
  mapDispatchToProps
)
```

But that's not all: the `connect()` function creates a new **HOC** to which we pass our `TodoList` component:

```javascript
connect(...)(TodoList);
```

Our `TodoList` component is now connected to the **Redux-Store** and we just have to make sure that we use it only within one `<Provider>` element. We also use `export default` before calling the `connect()` function to export our store connected component.

Finally, we take a look at the `index.js`, which ultimately represents the "entry point" of our app. This is where the `ReactDOM.render()` call takes place, with which we render our app into the respective DOM element. But before that happens, a lot is happening here:

We import `combineReducers` and `createStore` from `redux`. With this we create our `store` object. To do this, we import our outsourced `todosReducer`, which we pass to `combineReducers()` to create a new **Root Reducer**. This would be unnecessary at this point, since we only have one **Reducer** anyway and could therefore pass it directly to `createStore()`. However, as a precautionary measure in this case we assume that our application will continue to grow and that we will certainly add more **reducers** over time.

We also import the `Provider` component from `react-redux`. We will later transfer the newly created **Store** to them. From the `TodoList.js` we import our connected component, which we ultimately use in our `App` component _within_ the `Provider` component with our `store` object.

If we now interact with the `TodoList`, we can create new todos, mark them as done via a checkbox or remove them completely via the delete button.

At this point it certainly makes the most sense to play around with the component and see how the interaction affects the store. For this you should activate the Redux-Devtools for our store. Therefore we change the line in the `index.js`:

```javascript
const store = createStore(rootReducer);
```

The following one:

```javascript
const store = createStore(rootReducer, __REDUX_DEVTOOLS_EXTENSIONS__());
```

It must be ensured that the Redux Devtools are installed in the browser used.

## Conclusion

I have to admit that I far underestimated the complexity of this chapter. **Redux** has been part of my everyday business for years in many projects and so the use of **Redux** now feels a bit very _natural_, like something I don't have to think about much about. Basically I would describe **Redux** as a tool that manages to make very complex state management very simple, comprehensible and even _predictable_.

While writing this chapter I noticed how overwhelming **Redux** can be, but especially for beginners, and so I can imagine that some of the things described in this chapter will not necessarily be immediately understandable despite my explanations. But here it should actually help to play around with the **Actions** and the **Reducers** with open devtools to understand how one influences the other and how the interaction of all components, i.e. the store, the state, the props of a component, the `connect()` function as well as the **Actions** and **Reducer** finally work.

If you still have any questions after that, please feel free to contact me with regard to this book!

