# Chip Migration

This section explains how to migrate the `Chip` component from EJ2 React to React. It provides a detailed comparison of APIs, including props, methods and events.

Component-specific notes for `<Chip>`.

In EJ2 React `<ChipDirective text="..." />` lives inside the parent
`<ChipListComponent>`. In Pure React chip instances are `<Chip>` siblings or
can be embedded anywhere with content children.

## Properties

| EJ2 React | Pure React | Notes |
| --- | --- | --- |
| `avatarIconCss` | `avatar` | Accepts React node or string. |
| `cssClass` | `className` | Standard rename. |
| `enabled` (toggle) | `disabled` (inverted) | Standard rename. |
| `leadingIconCss` | `leadingIcon` | Renamed. Accepts React node. |
| `leadingIconUrl` | `leadingIconUrl` | Same name. |
| `enableDelete` | `removable` | Renamed. |
| `trailingIconUrl` | `trailingIconUrl` | Same name. |
| `text` | `text` | Same name (or use children). |
| `trailingIconCss` | `trailingIcon` | Renamed. Accepts React node. |
| `cssClass="e-flat"` / `"e-outline"` | `variant={Variant.Standard}` / `Variant.Outlined` | Enum replaces CSS string. |
| `cssClass="e-primary"` … `"e-danger"` | `color={Color.Primary}` … `Color.Error` | Enum replaces CSS string. |

## Methods

| EJ2 React | Pure React |
| --- | --- |
| `created` | `useEffect` with no deps |
| `destroy()` | `useEffect` cleanup |

## Events

| EJ2 React | Pure React |
| --- | --- |
| `delete` | `onDelete` |