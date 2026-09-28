# Menu Migration

This section explains how to migrate the `Menu` component from EJ2 React to React. It provides a detailed comparison of APIs, including props, methods and events.

Component-specific notes for `<Menu>`.

The EJ2 `items` array + `fields` mapping is replaced by JSX composition.
Menu items become `<MenuItem>` children with `<MenuItemLabel>` descendants:

```tsx
// EJ2 React
const menuItems = [{ text: 'File' }, { text: 'Edit' }];
<MenuComponent items={menuItems} />

// Pure React
<Menu>
  <MenuItem><MenuItemLabel>File</MenuItemLabel></MenuItem>
  <MenuItem><MenuItemLabel>Edit</MenuItemLabel></MenuItem>
</Menu>
```

## Properties

| EJ2 React | Pure React | Notes |
| --- | --- | --- |
| `orientation` | `orientation` (`Orientation` enum) | Same name. |
| `animationSettings` | `animation` | Renamed. Similar shape. |
| `showItemOnClick` | `itemOnClick` | Renamed. |
| `hoverDelay` | `hoverDelay` | Same name. |
| `enableScrolling` | Not applicable | Use `className` with `max-height` + `overflow-y: auto`. |
| `cssClass` | `className` | Standard rename. |
| `enableRtl` | `dir` on `<Provider>` | RTL moves to wrapper. |
| `items` | Not applicable | Use JSX `<MenuItem>` composition with `map()`. |
| `fields` | Not applicable | Use JSX composition. |

## Events

| EJ2 React | Pure React | Notes |
| --- | --- | --- |
| `onOpen` | `onOpen` | Fires when menu opens. |
| `onClose` | `onClose` | Fires when menu closes. |
| `select` | `onSelect` | Renamed. |
| `beforeOpen` | Not directly available | Use `onOpen` for similar pre-open logic. |
| `beforeClose` | Not directly available | Use `onClose` for similar pre-close logic. |
| `beforeItemRender` | Not available | Handle render-time logic directly in JSX. |
| `created` | `useEffect` with no deps | Lifecycle via hook. |