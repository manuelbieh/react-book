# Typechecking with PropTypes, Flow and TypeScript

**Typechecking** is a simple method to avoid potential errors in an application. We have talked about basic principles to avoid potential errors, one of the most important being the existence of "pure" components meaning that components do not have side effects. The non-existence of side effects describe the situation in which the **same input parameters** \(in the case of React we are talking about **props** and **state**\) should always render the **same output**.

We should be able to foresee and be clear about which props are passed into a component and how they are being processed. In order to achieve this, we can employ **typechecking** to make this process easier. JavaScript is slightly unusual in the sense that a variable which used to be **String** can easily be converted into a **Number** or even an **Object** without the JavaScript interpreter even complaining.

Even if this sounds useful for development, it opens doors for errors and also forces us to manually check for the correct types. When we want to access nested properties such as `user.settings.notifications.newMessages` we have to check user is actually an Object and not null, subsequently perform the same check for settings and so on. Otherwise, we might run into type errors:

{% hint style="danger" %}
TypeError: Cannot read property 'settings' of undefined
{% endhint %}

**Typechecking** can help us to catch such errors before they actually happen. Apart from **Flow** and **TypeScript** which allow for their own static typing, React provides its own simple solution called **PropTypes**. While **Flow** and **TypeScript** can also be used in JavaScript applications, **PropTypes** are exclusively used in React components. If you are interested into static types, you might like exploring **Flow** or **TypeScript** in greater detail.

