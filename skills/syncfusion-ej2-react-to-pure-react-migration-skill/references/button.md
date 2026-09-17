# Button Migration

This section explains how to migrate the `Button` component from EJ2 React to React. It provides a detailed comparison of APIs, including props, methods and events.

Component-specific notes for `<Button>`.

## Properties

| EJ2 React | Pure React | Notes |
| --- | --- | --- |
| `cssClass` | `className` | Standard rename. |
| `iconCss` | `icon` | Accepts string or `React.ReactNode` (SVG/component). |
| `iconPosition` | `iconPosition` | Use `Position` enum (`Position.Left`, `Position.Right`). |
| `isToggle` | `toggleable` | Renamed. |
| `checked` | `selected` | Toggle state renamed. |
| `enableRtl` | `dir` on `<Provider>` | RTL moves to wrapper. |
| `cssClass="e-link"` | `isLink={true}` | Semantic link styling. |
| `cssClass="e-small"` | `size={Size.Small}` | Enum replaces CSS string. |
| `cssClass="e-large"` | `size={Size.Large}` | Enum replaces CSS string. |
| `cssClass="e-flat"` / `"e-outline"` | `variant={Variant.Standard}` / `variant={Variant.Outlined}` | Enum replaces CSS string. |
| `cssClass="e-primary"` / `"e-success"` / `"e-info"` / `"e-warning"` / `"e-danger"` | `color={Color.Primary}` / `Success` / `Info` / `Warning` / `Error` (note `Error` for danger) | Enum replaces CSS string. |

`disabled` carries over.

## Events

| EJ2 React | Pure React | Notes |
| --- | --- | --- |
| `clicked` | `onClick` | Standard rename. |