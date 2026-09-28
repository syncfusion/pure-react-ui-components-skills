# DateRangePicker Migration

This section explains how to migrate the `DateRangePicker` component from EJ2 React to React. It provides a detailed comparison of APIs, including props, methods and events. 

Component-specific notes for `<DateRangePicker>`.

## Properties

| EJ2 React | Pure React | Notes |
| --- | --- | --- |
| `allowEdit` | `editable` | Renamed. |
| `cssClass` | `className` | Standard rename. |
| `dayHeaderFormat` | `weekDaysFormat` | Renamed. |
| `enableRtl` | `dir` on `<Provider>` | RTL moves to wrapper. |
| `enabled` | `disabled` (inverted) | Standard rename. |
| `startDate` / `endDate` | `defaultValue={[start, end]}` | Combined to a default value. |
| `floatLabelType` | `labelMode` | Renamed. |
| `locale` | `locale` on `<Provider>` | Locale moves to wrapper. |
| `max` | `maxDate` | Renamed. |
| `maxDays` | `maxRangeDays` | Renamed. |
| `min` | `minDate` | Renamed. |
| `minDays` | `minRangeDays` | Renamed. |
| `readonly` | `readOnly` | Standard React prop. |
| `showClearButton` | `clearButton` | Renamed. |
| `fullScreenMode` | `pickerVariant` | `PickerVariant.Auto`. |
| `width` | `style` | Use `style={{ width }}`. |

`depth`, `firstDayOfWeek`, `format`, `inputFormats`, `openOnFocus`,
`placeholder`, `presets`, `separator`, `start`, `strictMode`, `value`,
`weekNumber`, `weekRule`, `zIndex` carry over (use `WeekRule` enum).

## Events

| EJ2 React | Pure React | Notes |
| --- | --- | --- |
| `change` | `onChange` | Value change. |
| `select` | `onChange` | Range end chosen. |
| `navigated` | `onViewChange` | Calendar view change. |
| `open` | `onOpen` | Popup open. |
| `close` | `onClose` | Popup close. |
| `renderDayCell` | `cellTemplate` (prop) | Switched from event handler to template prop. |