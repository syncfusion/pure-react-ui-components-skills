# ListView Migration

This section explains how to migrate the `ListView` component from EJ2 React to React. It provides a detailed comparison of APIs, including props, methods and events.

Component-specific notes for `<ListView>`.

## Properties

| EJ2 React | Pure React | Notes |
| --- | --- | --- |
| `cssClass` | `className` | Standard rename. |
| `enabled` | `disabled` (inverted) | Standard rename. |
| `height` / `width` | `style={{ height }}` / `style={{ width }}` | Use style object (or pass `inputProps` style). |
| `htmlAttributes` | Pass DOM attributes directly on `<ListView>` | e.g. `id="..."` or `data-*`. |
| `headerTitle` | `headerTemplate` | Renamed (use a render function/node). |
| `enableVirtualization` | `virtualization={{ itemSize, pageSize, overscanCount }}` | Object replaces boolean. |
| `enableRtl` | `dir` on `<Provider>` | RTL moves to wrapper. |
| `showCheckBox` | Not applicable | Render a checkbox inside `itemTemplate`, manage state yourself. |
| `fields.selected` / `fields.isChecked` / `fields.checked` | Not applicable | No built-in selection; emulate with state + itemTemplate. |

`dataSource`, `fields`, `query`, `sortOrder`, `headerTemplate`, `footerTemplate`,
`itemTemplate`, `groupTemplate` carry over.

## Events

| EJ2 React | Pure React |
| --- | --- |
| `actionBegin` | `onDataRequest` |
| `dataBound` | `onDataLoad` |
| `scroll` | `onScroll` |