# Toolbar Migration

This section explains how to migrate the `Toolbar` component from EJ2 React to React. It provides a detailed comparison of APIs, including props, methods and events.

Component-specific notes for `<Toolbar>`.

In EJ2 React, `<ItemsDirective>` / `<ItemDirective>` populate the toolbar.
In Pure React, `<ToolbarItem>` children do the same:

```tsx
// EJ2 React
<ToolbarComponent>
  <ItemsDirective>
    <ItemDirective text="Cut" />
    <ItemDirective text="Copy" />
  </ItemsDirective>
</ToolbarComponent>

// Pure React
<Toolbar>
  <ToolbarItem><Button title="Cut" /></ToolbarItem>
  <ToolbarItem><Button title="Copy" /></ToolbarItem>
</Toolbar>
```

## Properties

| EJ2 React | Pure React | Notes |
| --- | --- | --- |
| `allowKeyboard` | `keyboardNavigation` | Renamed. |
| `cssClass` | `className` | Standard rename. |
| `enableCollision` | `collision` | Renamed. |
| `overflowMode` | `overflowMode` (`OverflowMode` enum) | Enum replaces string. |

`scrollStep` carries over.

## Methods

| EJ2 React | Pure React |
| --- | --- |
| `destroy()` | `useEffect` cleanup |
| `refreshOverflow()` | `refreshOverflow()` on typed ref |