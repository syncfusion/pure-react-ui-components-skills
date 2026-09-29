# Floating Action Button (Fab) Migration

This section explains how to migrate the `Fab` component from EJ2 React to React. It provides a detailed comparison of APIs, including props, methods and events.

Component-specific notes for `<Fab>`.

## Properties

| EJ2 React | Pure React | Notes |
| --- | --- | --- |
| `position="BottomRight"` | `position={FabPosition.BottomRight}` | Enum replaces string. |
| `cssClass` | `className` | Standard rename. |
| `iconCss` | `icon` | Accepts string or React node. |
| `iconPosition="Right"` | `iconPosition={Position.Right}` | Enum replaces string. |
| `cssClass="e-flat"` / `"e-outline"` | `variant={Variant.Standard}` / `Variant.Outlined` | Enum. |
| `cssClass="e-primary"` / `"e-success"` / `"e-info"` / `"e-warning"` / `"e-danger"` | `color={Color.Primary}` / `Success` / `Info` / `Warning` / `Error` | Enum. |
| `cssClass="e-small"` | `size={Size.Small}` | Enum. |
| `cssClass="e-large"` | `size={Size.Large}` | Enum. |

`visible`, `toggleable`, `disabled` carry over.

## Methods

| EJ2 React | Pure React | Notes |
| --- | --- | --- |
| `show()` | Not applicable | Use `visible` prop + state. |
| `hide()` | Not applicable | Use `visible` prop + state. |
| `toggle()` | Not applicable | Integrated with `toggleable` and `visible` state. |

## Events

| EJ2 React | Pure React |
| --- | --- |
| `click` | `onClick` |