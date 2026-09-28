# RadioButton Migration

This section explains how to migrate the `RadioButton` component from EJ2 React to React. It provides a detailed comparison of APIs, including props, methods and events.

Component-specific notes for `<RadioButton>`.

## Properties

| EJ2 React | Pure React | Notes |
| --- | --- | --- |
| `checked` | `defaultChecked` | Initial state. Combine with `onChange` to control. |
| `cssClass` | `className` | Standard rename. |
| `labelPosition` | `labelPlacement` | Renamed. |
| `cssClass="e-small"` | `size={Size.Small}` | Enum. |
| `cssClass="e-large"` | `size={Size.Large}` | Enum. |
| `cssClass="e-primary"` / `"e-success"` / `"e-info"` / `"e-warning"` / `"e-danger"` | `color={Color.Primary}` / `Success` / `Info` / `Warning` / `Error` | Enum. |

`disabled`, `label` carry over.

## Events

| EJ2 React | Pure React |
| --- | --- |
| `onCheckedChange` | `onChange` |
| `destroyed` | `useEffect` cleanup |