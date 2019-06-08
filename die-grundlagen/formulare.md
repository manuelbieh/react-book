# Forms

Forms have a small special place in React and work somewhat differently than other DOM elements, because forms have a kind of **own state** that has nothing in common with the React state. 

The state of text fields, for example, consists of the entered value, the state of checkboxes or radio buttons results from the fact whether these are selected or not, selection lists \(`<select></select>`\) hold the selected value as the state or, in the case of multiple selection, the selected values. React does not change this behaviour for the time being. If you want to, you can keep it that way and don't have to worry about anything else. 

In React jargon, we are then talking about **Uncontrolled Components****. Uncontrolled because React does not take care of the state management of these components. The state handling is either completely independent of React or only works from the direction of the DOM form elements towards React State, but **not in the opposite direction**. A form element gets nothing from an update at the react state and shows the same value \(or status for checkboxes, selects and radio buttons\) as before.

On the other hand there are the **Controlled Components****. Here an update at the react state updates the value \(or status\) of the form element and likewise an update at the respective form element updates the react state. **Controlled Components** are somewhat more complex to implement, but are also the "safer" variant, since we do not run the risk of both states differing from each other.

## Uncontrolled Components / Uncontrolled Components

**Uncontrolled components** can essentially occur in two different forms. The first variant simply renders form elements that are processed on the server side and do not interact with React in any way. A completely static form, if you will. React **doesn't take care of the connection to the React-State by itself** but **leaves the developer all freedom**!

In the second variant, changes to a form element **are written to the react state**, e.g. to validate the data in the background or to output the entered data elsewhere. A change to the react state elsewhere in the application has no direct effect on the form fields. 

An example of such an uncontrolled component: 

```jsx
class Uncontrolled extends React.Component {
  state = {
    username: '',
    isValid: false,
  };

  changeUsername = e => {
    const { value } = e.target;
    this.setState(() => ({
      username: value,
      isValid: value.length > 3,
    }));
  };

  submitForm = e => {
    e.preventDefault();
    alert(`Hello ${this.state.username}`);
  };

  render() {
    return (
      <form method="post" onSubmit={this.submitForm}>
        <p>Your username: {this.state.username}</p>
        <p>
          <input 
            type="text" 
            name="username" 
            onChange={this.changeUsername} />
          <input type="submit" disabled={!this.state.isValid} />
        </p>
      </form>
    );
  }
}
```

Here we see a simple text field in which the user can enter a desired user name. The `Uncontrolled` component is notified of every change via `onChange` event and can further process the user name. Since React only acts **passively** here, i.e. it is informed about the new value when the text field is changed, we are still in the range of **Uncontrolled Components**.

This is sufficient in some cases, especially if the forms are not yet too complex. However, the react state here is **decoupled** from the DOM state or only **works in one direction**. The react state is updated as soon as the `onChange` event of the text field is triggered. However, this means that our text field is not updated at the same time if the value in the react state has been changed elsewhere, e.g. because the response of an asynchronous request arrives after some time.

A form field is considered **controlled** when a `value` attribute is set. From this moment on, React expects us as developers to take care of synchronizing the React state with the form field ourselves. But if we want to set an initial value only once without making the whole component a **Controlled Component**, we have the possibility to set the React own `defaultValue` attribute instead of the `value` attribute \(`defaultChecked` for checkboxes and radio buttons\). The element then remains **uncontrolled**, but still displays a prefilled value \(or status\).

## Controlled Components

To map state updates to form fields as well as to transfer user-side changes to form fields to the react state on the other side, we need a **Controlled Component**. Here we leave the state handling of a form element completely to React. This means that we fill the `value` attribute with a value that we get from the react state and at the same time transfer a changed value back into the react state.

The goal of this approach is to consider the React-State \(or another state container like Redux\) as **Single Source of Truth**, the _only source of truth_. Relevant is the value in the state managed by React, the respective input field then reflects the value from this state at any time.

Let's have a look at an example here:

```jsx
class Controlled extends React.Component {
  state = {
      username: '',
      isValid: false,
  };

  changeUsername = (e) => {
    const { value } = e.target;
    this.setState(() => ({
      username: value,
      isValid: value.length > 3
    }));
  };

  submitForm = (e) => {
    e.preventDefault();
    alert(`Hello ${this.state.username}`);
  };

  render() {
    return (
      <form method="post" onSubmit={this.submitForm}>
        <p>{username}</p>
        <p>
          <input
            type="text"
            name="username"
            onChange={this.changeUsername}
            value={this.state.username}/>
          <input type="submit" disabled={!this.state.isValid} />
        </p>
      </form>
    );
  }
}
```

At first glance, the `Controlled` component does not differ much from the `Uncontrolled` component in the paragraph above. And indeed, the decisive factor that turns the uncontrolled component into a controlled one is solely the `value` attribute of the `<input />` element. If one exists, **React** controls the form element and expects changes to the input field to be reflected in the state accordingly. The `onChange` handler is also important to transfer the value to the react state when there is a change. A common mistake when working with forms in React for the first time is not to synchronize corresponding input fields with the React state by writing the new value to the state. The input field then does not change and continues to display the old value from `this.state`.

Here are some more things to keep in mind. So the value of the `value` attribute may always only be a **string**, never `undefined` or `null`.

![Warning for a controlled textfield with the value &quot;null&quot;](../.gitbook/assets/react-uncontrolled-null.png)

An exception are `select` elements that have a `multiple` attribute. Here, the `value` attribute must be an **array**. 

Wait a minute, are you thinking. Which `value` attribute for `<select>`? I select options by setting the `selected` attribute at the respective `<option>`! And yes, that's correct in HTML, in React it works a little different. Here the controlled value is also set via the `value` attribute. The same is true for the `<textarea>` element, whose initial value is usually determined by its `textContent`. Not so in React. 

React unifies the mechanism for changing values a bit and requires for the three elements `input` \(all types except `checkbox` and `radio`\), `textarea` and `select` a `value` attribute! For simple values this must **always be a string**, for a selection list with the `multiple` attribute as just mentioned an **array consisting of strings**!

In addition, a change to a form element must **always be transferred back to the react state**. This can sometimes be a bit tedious, especially with checkboxes and radio buttons where not only a value is changed, but the status \(`checked`\) to a value.

In the following I would like to show a fully controlled component that contains all basic types of form elements that include HTML \(other `input` elements of type `email`, `date`, `range`, etc. work identically to input fields of type `text`\).

```jsx
class FullyControlledComponent extends React.Component {
  state = {
    text: "",
    textarea: "",
    checkbox: false,
    singleSelect: "",
    multipleSelect: [],
  };

  changeValue = ({ target: { name, value } }) => {
    this.setState(() => ({
      [name]: value,
    }));
  };

  changeCheckbox = ({ target: { name, checked } }) => {
    this.setState(() => ({
      [name]: checked,
    }));
  };

  changeSelect = ({ target: { name, value, selectedOptions, multiple } }) => {
    if (multiple) {
      value = Array.from(selectedOptions).map((option) => option.value);
    }

    this.setState(() => ({
      [name]: value,
    }));
  };

  render() {
    return (
      <form>
        <input
          type="text"
          name="text"
          value={this.state.text}
          onChange={this.changeValue}
        />

        <textarea
          name="textarea"
          value={this.state.textarea}
          onChange={this.changeValue}
        />

        <input
          type="checkbox"
          name="checkbox"
          checked={this.state.checkbox}
          onChange={this.changeCheckbox}
        />

        <input
          type="radio"
          name="radio"
          value="1"
          &lt;font color="#ffff00"&gt;-==- proudly presents
          onChange={this.changeValue}
        />
        <input
          type="radio"
          name="radio"
          value="2"
          &lt;font color="#ffff00"&gt;-==- proudly presents
          onChange={this.changeValue}
        />

        <select
          name="singleSelect"
          value={this.state.singleSelect}
          onChange={this.changeValue}
        >
          <option value="">Please select</option>
          <option value="1">One</option>
          <option value="2">Two</option>
        </select>

        <select
          name="multipleSelect"
          value={this.state.multipleSelect}
          onChange={this.changeSelect}
          multiple
        >
          <option value="1">One</option>
          <option value="2">Two</option>
        </select>

        <pre>{JSON.stringify(this.state, null, 2)}</pre>
      </form>
    );
  }
}
```

At the core of the form are the three event handler methods for the different types of form elements: `changeValue`, `changeCheckbox` and `changeSelect`.

They are called at the `onChange` event of the respective form elements and receive an object of the type `SyntheticEvent`. From its `target` property, we use **ES2015 Object Destructuring** to pick individual properties and then update the state accordingly.

For elements of type `<input type="text" />`, `<input type="radio" />` and `<textarea />` these are `name` and `value`, for `<input type="checkbox" />` we are interested in the `name` and `checked` property, for `select` elements we are also interested in the `name` and then, depending on whether it is a simple selection list or a multiple selection list, again the `value` or the `selectedOptions`. Whether we are dealing with a single or multiple pick list, we find out by means of the `multiple` property, which we also pick out of the `e.target` property.

### Change of values

If a value is changed, as is the case with text input or radio buttons, we set a state property of the same name to the respective value that the user entered and used to trigger the `onChange` event. Since we are in a controlled component, the following now works:

1. the user changes the value by means of text input
2. a `onChange` event is triggered and processed in the event handler
3. the event handler sets the state property to the new value
React re-renders the user interface and sets the `value` property of the input field to the new value from `this.state`.
5 The user sees his newly entered value. 

For the user this is **Business as Usual**. He doesn't notice that the form works differently than he knows from the browser. And yet React took care of the logic in the background and drew a new "frame" in the user interface.

### Change of states of checkboxes and radio buttons

Checkboxes \(`<input type="checkbox" />`\) work in the same way in the background, but with the difference that their value remains the same. For checkboxes, instead of the value, the state of their `checked` property changes from `true` to `false` or vice versa. They are therefore considered to be controlled when their `checked` property is controlled by React. The `onChange` event for checkboxes informs us by means of `e.target.checked` whether the checkbox just changed is now activated \(`true`\) or not activated \(`false`\). We pass this status unchanged to the React state, React then takes care in the rendering that the new status of the checkbox is displayed.

Radio buttons are a kind of hybrid element. Like checkboxes, they are also controlled if their `checked` attribute is managed by React. However, there are usually several radio buttons with the same name, but with different values in one document. So it wouldn't make sense to set the value of a name to `true` or `false`, because we are interested in the actual value of the selected radio button. So here we write the value of the radio button into the state like with text elements and then check during the rendering of the respective radio button whether the selected value from the state corresponds to the own value: `checked={this.state.radio === "1"}`. So in this example: set `checked` to `true` if the value of a radio button with the name `radio` equals `1`.

### Changing the status of single or multiple selection lists

Let's start with the simple case: simple `<select>` selection lists also change their value like text fields, trigger a rendering and display the selected value in the newly drawn user interface. Multiple selection lists are an exception here.

Unlike text fields or simple selection lists, multiple selection lists do not expect a string as a value, but an **array of strings**. However, we have to build this ourselves, because `e.target.value` contains only one value for multiple selection lists, even when multiple options are selected. This is where `e.target.selectedOptions` helps us. This property is an object of type `HTMLCollection`, with the `<option>` elements currently selected. We can easily convert this object into an array with the static array method `Array.from()` from ES2015. By iterating over this using `Array.map()`, we can also create a new array containing all the values relevant to us: 

```jsx
Array.from(selectedOptions).map((option) => option.value);
```

We then write the new array created in this way as a new value in our state. Before we do this we first use `e.target.multiple` to see if it is a `<select>` with multiple selection, because only this one expects an array as `value`.

Alternatively it would also be possible to pass the `changeValue` method as event handler to simple selection lists and the `changeSelect` method to multiple selection lists only. Then we could save ourselves the check whether it is a `multiple` select. To solve it in the above way has the advantage that later the type could be changed from multiple to simple without having to change the event handler. But in the end, of course, that's up to you.

### Special features of controlled components

In the above examples I used the `name` attribute of each element as a key to store its value in the state. This is particularly useful when working with server-side React and, for example, automatically generating and processing forms on the basis of a database schema. However, this is not a prerequisite for a functioning controlled component. Theoretically, neither a `name` attribute is required, nor does the name of the state property have to match the `name` attribute. 

You can also nest the stored values, which is useful if you have multiple forms in one component \(**Note:** React Antipattern!\). In addition, it is not even necessary to use the react state to map **Controlled Components**. On the contrary, in practice an external state container such as **Redux**, **Unstated** or **MobX** is often used instead. 

## Conclusion

{% hint style="info" %}
Forms in React may occur in \(by React\) controlled or uncontrolled form.

**Uncontrolled components** are often sufficient for simple forms, but it is recommended to let React control form components to have a **Single Source of Truth**. The `value` or `checked` attribute of React must be managed. The developer then has to react manually to changes.

Unlike traditional HTML, React expects the value of texttareas, selects, and input fields with text input in the `value` attribute.
{% endhint %}



