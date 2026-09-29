# Switch Migration

This section explains how to migrate the `Switch` component from EJ2 React to React. It provides a detailed comparison of APIs, including props, methods and events.

Component-specific notes for `<Switch>`.

## Properties

| EJ2 React | Pure React | Notes |
| --- | --- | --- |
| `checked` | `checked` (controlled) or `defaultChecked` (uncontrolled) | Both supported in Pure React. |
| `cssClass` | `className` | Standard rename. |
| `onLabel` | `onTrackLabel` | Renamed. |
| `offLabel` | `offTrackLabel` | Renamed. |
| `enableRtl` | `dir` on `<Provider>` | RTL moves to wrapper. |

`disabled`, `value` carry over.

## Events

| EJ2 React | Pure React | Notes |
| --- | --- | --- |
| `change` | `onChange` | Renamed. Arg shape `.value` (`event.value`). |
| `beforeChange` | Not directly available | Use `onChange` for pre-toggle logic. |
| `created` | `useEffect` with no deps | Lifecycle via hook. |