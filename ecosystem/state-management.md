# State Management

The more an application grows in the size, the more its complexity and data grows. Managing **application state** becomes increasingly difficult and cumbersome to do. When do I pass props to which component and how? How do the props influence the state of my components and what happens if I change the state of a component?

React has introduced great tooling in the last few years to deal with this increase of complexity in application state. The **Context API** as well as the **useReducer** **hook** have been great additions to the React developer's toolbox and allow us to work more comfortably with complex data. However, in some cases it is still very difficult to keep track of all the different pieces of data and how they transform. To deal with this type of problem, external tools for **global state management** can be a great choice. The React ecosystem has seen a number of players enter this space giving us lots of libraries to choose from.

One of the more commonly known tools is **MobX** which describes itself as a tool _"Simple, scalable state management"_ solution. The other well-known tool in the industry is **Redux**. Redux was in part developed by Dan Abramov and Andrew Clark who are now part of the official React Core Team. It positions itself as _"A predictable state container for JavaScript apps"._ Redux is arguably the more dominating tool boasting with higher usage stats and downloads.

This chapter will dive deeper into **Redux.** This is due to a number of reasons: first of all, I have experience with **Redux** myself and have used in it a number of different projects. Additionally, my experience with Redux has been very pleasant. But those are not the only reasons. Redux still has around **4 million weekly installations**. Compared to **MobX** which still has a very respectable number of **200.000 installations** per week, Redux is the dominant force in the React ecosystem though.

Despite a number of voices which keep proclaiming Redux's death, the interest in **Redux** has remained steady over the years and the number of downloads is still growing. When the **Context API** was introduced with **React 16.3.0**, **Redux** was said to be obsolete. Once **React 16.8.0** introduced Hooks, more specifically the **useReducer hook**, the critics' voices were raised again. Despite all this criticism, the number of installations of Redux keep growing.

In fact, **Redux** makes use of **Context** and **Hooks** under the hood and uses these principles to optimise its own performance and simplify its API. Moreover, **Redux** has seen a great increase of custom addons and tools for their own ecosystem which is enhanced rather than replaced by the new functionality offered by React.



