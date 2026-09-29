# ChipList Migration

This section explains how to migrate the `ChipList` component from EJ2 React to React. It provides a detailed comparison of APIs, including props, methods and events.

Component-specific notes for `<ChipList>`.

The EJ2 directive pair `<ChipsDirective>` / `<ChipDirective>` is gone —
chips are passed as a `chips` array prop on `<ChipList>`.

## Properties

| EJ2 React | Pure React | Notes |
| --- | --- | --- |
| `enabled` (boolean toggle) | `disabled` (inverted) | Standard rename. |
| `cssClass` | `className` | Standard rename. |
| `enableDelete` | `removable` | Renamed. |
| `selectedChips` | `selectedChips` | Same name. |
| `selection` | `selection` | Same name. |

## Methods

| EJ2 React | Pure React |
| --- | --- |
| `created` | `useEffect` with no deps |
| `destroy()` | `useEffect` cleanup |
| `getSelectedChips()` | `getSelectedChips()` on the typed ref (e.g. `useRef<ChipListRef<T>>`) |

## Events

| EJ2 React | Pure React |
| --- | --- |
| `delete` | `onDelete` |