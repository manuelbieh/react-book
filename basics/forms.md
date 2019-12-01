# Forms

Forms play a special role among the other DOM elements in React and work a little different as they have some sort of **state**. This state is not related to React.

The state of a text field results from the value entered, the state of a checkbox or a radio button results from being selected or not and `<select></select>` lists hold the state of one or more `<option></option>` selected. React does not change any of these values. If you feel comfortable using the form state, you can keep using it without problem and nothing changes for the development of your components.

React calls these components **uncontrolled components** as React does not concern itself with the state management of these components. State handling is either completely indepedent of React or only in the direction of DOM form state to React state, but **never the other way round**. A form element does not know about updates in React state and will keep showing the same value or status \(in the case of checkboxes, selects and radio buttons\) as before.

**Controlled components**, in contrast, are deeply linked to React State. An update in the React state will have an effect on the value or the status of the form element and vice versa. While **controlled components** are harder to implement, they are "safer" to use as it is less likely that both states differ from each other.

## Uncontrolled Components

**Uncontrolled components** can take two different forms. First, plain form elements can be rendered which are processed server-side and do not interact with React at any time. The form is completely static so to say. React does **not intervene** if this is what's wanted and allows the developer to develop **freely**.

But uncontrolled components could also still interact with React, which is the second form of an **uncontrolled component**. This variation of an uncontrolled component writes changes of the form element **into React state** either to validate data in the background or to render the data in a different place. Changes that have been made to the state in React in different parts of the application do not directly influence the form fields.

An example for an controlled component:

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
    alert(`Hallo ${this.state.username}`);
  };

  render() {
    return (
      <form method="post" onSubmit={this.submitForm}>
        <p>Dein Benutzername: {this.state.username}</p>
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

The user can enter their username into a simple text field. The `uncontrolled` component is notified of a change via the `onChange` event and can process the username further if necessary. As React only reacts **passively** and is simply notified of changes in the text field, we still refer to these types of components as **uncontrolled components**.

In most cases, it is sufficient to define these type of **uncontrolled components** if forms are not overly complex. However, we have to remember that the react state and DOM state are completely **decoupled** from each other and only works **in one direction**. Once the `onChange` event has been triggered, the React state can update safely. However, the text field would not update if changes to value of the React state had been made elsewhere in the application \(for example due to a response in an asynchronous request\).

