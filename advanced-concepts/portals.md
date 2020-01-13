# Portals

**Portals** allow us to render components in DOM nodes which are _outside ****_of the parent-node of the current component hierarchy but still have access to the current component environment. A common example \(but of course not the only one\) is an overlay which is rendered in its own `<div>` outside of the actual application.

The portal remains in the context of the component that has created it and thus has access to all data that is also available to the parent component such as its props and state. However, they are placed in entirely different locations in the rendered HTML compared to the rest of the application. Being able to access props and state is crucial for **Portals**, as they allow us to access common Context such as translations.

