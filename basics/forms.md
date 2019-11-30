# Forms

Forms play a special role among the other DOM elements in React and work a little different as they have some sort of **state**. This state is not related to React.

The state of a text field results from the value entered, the state of a checkbox or a radio button results from being selected or not and `<select></select>` lists hold the state of one or more `<option></option>` selected. React does not change any of these values. If you feel comfortable using the form state, you can keep using it without problem and nothing changes for the development of your components.

React calls these components **uncontrolled components** as React does not concern itself with the state management of these components. State handling is either completely indepedent of React or only in the direction of DOM form state to React state, but **never the other way round**. A form element does not know about updates in React state and will keep showing the same value or status \(in the case of checkboxes, selects and radio buttons\) as before.

**Controlled components**, in contrast, are deeply linked to React State. An update in the React state will have an effect on the value or the status of the form element and vice versa. While **controlled components** are harder to implement, they are "safer" to use as it is less likely that both states differ from each other.

