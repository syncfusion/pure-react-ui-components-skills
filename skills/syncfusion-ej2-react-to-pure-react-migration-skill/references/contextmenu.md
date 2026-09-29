# ContextMenu Migration

This section explains how to migrate the `ContextMenu` component from EJ2 React to React. It provides a detailed comparison of APIs, including props, methods and events.

Component-specific notes for `<ContextMenu>`.

The EJ2 `items` array + `itemTemplate` is replaced by JSX composition.
Menu items become `<MenuItem>` children, each with a `<MenuItemLabel>`
descendant:

```tsx
// EJ2 React
const menuItems = [{ text: 'Complete' }, { text: 'Incomplete' }];
<ContextMenuComponent items={menuItems} />

// Pure React
<ContextMenu>
  <MenuItem><MenuItemLabel>Complete</MenuItemLabel></MenuItem>
  <MenuItem><MenuItemLabel>Incomplete</MenuItemLabel></MenuItem>
</ContextMenu>
```

## Properties

| EJ2 React | Pure React | Notes |
| --- | --- | --- |
| `animationSettings={{ duration, effect }}` | `animation={{ effect, duration, easing }}` | Renamed; new `MenuAnimationProps` shape. |
| `items` array + `itemTemplate` | JSX `<MenuItem>` composition | Replacement pattern. |
| `cssClass` | `className` on `<ContextMenu>` or `<MenuItem>` | Standard rename. |
| `enableScrolling` | Not available | Use `className` with `max-height` and `overflow-y: auto`. |
| `showItemOnClick` | `itemOnClick` | Renamed. |
| `hoverDelay` | `hoverDelay` | Same name. |
| `target` (CSS selector string) | `targetRef` (React `RefObject<HTMLElement>`) | String selector → ref. |
| `enableRtl` | `dir` on `<Provider>` | RTL moves to wrapper. |

## Methods

| EJ2 React | Pure React |
| --- | --- |
| `destroy()` | `useEffect` cleanup |

## Events

| EJ2 React | Pure React | Notes |
| --- | --- | --- |
| `onClose` | `onClose` | Receives a native `Event` object. |
| `onOpen` | `onOpen` | Receives a native `Event` object. |
| `onSelect` | `onSelect` | `MenuSelectEvent` — payload includes `event` (SyntheticEvent) and `item` (MenuItemProps). |