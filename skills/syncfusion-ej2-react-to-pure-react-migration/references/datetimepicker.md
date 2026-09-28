# DateTimePicker Migration

This section explains how to migrate the `DateTimePicker` component from EJ2 React to React. It provides a detailed comparison of APIs, including props, methods and events.

Component-specific notes for `<DateTimePicker>`.

## Properties

| EJ2 React | Pure React | Notes |
| --- | --- | --- |
| `allowEdit` | `editable` | Renamed. |
| `cssClass` | `className` | Standard rename. |
| `calendarMode` | `calendarType` | `'gregorian'` / `'islamic'`. |
| `enableRtl` | `dir` on `<Provider>` | RTL moves to wrapper. |
| `enabled` | `disabled` (inverted) | Standard rename. |
| `floatLabelType` | `labelMode` | Renamed. |
| `timeFormat` | `format` | Renamed; use the single `format` for both date + time. |
| `fullScreenMode` | `pickerVariant` | `PickerVariant.Auto`. |
| `htmlAttributes` | `inputProps` | Renamed. |
| `max` | `maxDate` | Renamed. |
| `min` | `minDate` | Renamed. |
| `readonly` | `readOnly` | Standard React prop. |
| `showClearButton` | `clearButton` | Renamed. |
| `depth` | `depth` (`CalendarView` enum) | |
| `start` | `start` (`CalendarView` enum) | |
| `dayHeaderFormat` | `weekDaysFormat` | Renamed. |
| `width` | `style` | Use `style={{ width }}`. |
| `enableMask` | `inputMask` | Renamed. |

`maxTime`, `minTime`, `firstDayOfWeek`, `format`, `inputFormats`,
`strictMode`, `openOnFocus`, `placeholder`, `step`, `value`, `weekNumber`,
`weekRule`, `zIndex`, `maskPlaceholder` carry over (use `WeekRule` enum).

## Events

| EJ2 React | Pure React | Notes |
| --- | --- | --- |
| `change` | `onChange` |
| `open` | `onOpen` |
| `close` | `onClose` |
| `navigated` | `onViewChange` |
| `renderDayCell` | `cellTemplate` (prop; React node/function) |