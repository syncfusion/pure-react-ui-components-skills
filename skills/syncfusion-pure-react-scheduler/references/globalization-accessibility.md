# Globalization, Localization, Timezone, and Accessibility

Adapt the scheduler to different languages, cultural preferences, and timezones, and make it accessible to all users.

## Localization

The scheduler includes built-in localization to customize text for different languages (e.g. Arabic, German, French) by setting the `locale` property in a custom `Provider` with translation objects.

### Configuration steps

1. Choose a [locale](https://github.com/syncfusion/react-locale) from Syncfusion's React localization resources.
2. Install the CLDR data package for culture-specific formatting:

```bash
npm i @syncfusion/react-cldr-data
```

3. Import `L10n` and `loadCldr` from `@syncfusion/react-base`.
4. Import localization files and CLDR data for your chosen languages:

```tsx
import { L10n, loadCldr } from "@syncfusion/react-base";
import Localization from './locale.json';
import * as enAllData from '@syncfusion/react-cldr-data/main/en/all.json';
import * as deAllData from '@syncfusion/react-cldr-data/main/de/all.json';
import * as frAllData from '@syncfusion/react-cldr-data/main/fr/all.json';
import * as arAllData from '@syncfusion/react-cldr-data/main/ar/all.json';

import * as numberingSystems from '@syncfusion/react-cldr-data/supplemental/numberingSystems.json';
import * as weekData from '@syncfusion/react-cldr-data/supplemental/weekData.json';
import * as currencyData from '@syncfusion/react-cldr-data/supplemental/currencyData.json';

import * as enUSLocalization from '@syncfusion/react-locale/src/en-US.json';
import * as frFRLocalization from '@syncfusion/react-locale/src/fr.json';
import * as arFRLocalization from '@syncfusion/react-locale/src/ar.json';
import * as deDELocalization from '@syncfusion/react-locale/src/de.json';
```

5. Call `loadCldr` with the imported data and load translations with `L10n.load`:

```tsx
loadCldr(enAllData, deAllData, frAllData, arAllData, numberingSystems, weekData, currencyData);
L10n.load({
    ...enUSLocalization,
    ...frFRLocalization,
    ...arFRLocalization,
    ...deDELocalization
});
```

6. Set the `locale` property on the `Provider` to activate the chosen language.

### Switching cultures at runtime

Wrap the scheduler in `Provider` (from `@syncfusion/react-base`) and re-call `loadCldr` for the active culture when it changes (enable RTL for Arabic):

```tsx
import { useState } from 'react';
import { Scheduler, DayView, WeekView, WorkWeekView, MonthView } from '@syncfusion/react-scheduler';
import { L10n, loadCldr, Provider } from '@syncfusion/react-base';
// ...same CLDR/localization imports as above...

L10n.load({ ...enUSLocalization, ...deDELocalization, ...frFRLocalization, ...arFRLocalization });
loadCldr(enAllData, frAllData, arAllData, deAllData, numberingSystems, weekData, currencyData);

export default function App() {
    const [locale, setLocale] = useState('en');
    const [rtl, setRtl] = useState(false);

    const changeCulture = (args?: ChangeEvent) => {
        const locale = args?.value as string;
        setLocale(locale);
        switch (locale) {
            case 'de':
                setRtl(false);
                loadCldr(deAllData, numberingSystems, currencyData);
                break;
            case 'fr':
                setRtl(false);
                loadCldr(frAllData, numberingSystems, currencyData);
                break;
            case 'ar':
                setRtl(true);
                loadCldr(arAllData, numberingSystems, currencyData);
                break;
            default:
                setRtl(false);
                loadCldr(enAllData, numberingSystems, currencyData);
                break;
        }
    };

    return (
        <Provider locale={locale} dir={rtl ? 'rtl' : 'ltr'}>
            <Scheduler height='34.375rem' defaultSelectedDate={new Date(2025, 0, 10)} startHour='09:00'>
                <DayView /><WeekView /><WorkWeekView /><MonthView />
            </Scheduler>
        </Provider>
    );
}
```

## Date format

The scheduler supports all valid date formats. Default is `MM/dd/yyyy`; if `dateFormat` is not specified, the format is derived from the current `locale`:

```tsx
<Scheduler dateFormat='yyyy/MM/dd' ...>
```

## Time format

Time display (12-hour vs 24-hour) also derives from the locale (12-hour for `en-US`). Override with the `timeFormat` property, which accepts only valid time format patterns:

```tsx
<Scheduler timeFormat="HH:mm" ...>
```

## First day of the week

Set the week's starting day with `firstDayOfWeek`. Days map as: Sunday = 0, Monday = 1, Tuesday = 2, and so on:

```tsx
<Scheduler firstDayOfWeek={1} ...>   {/* Monday */}
```

## RTL mode

Enable right-to-left layout by wrapping the app with `Provider` and setting `dir={'rtl'}`; the scheduler UI automatically flips to RTL conventions:

```tsx
import { Provider } from '@syncfusion/react-base';

<Provider dir={'rtl'}>
    <Scheduler...>
        <DayView /><WeekView /><WorkWeekView /><MonthView />
    </Scheduler>
</Provider>
```

## Timezone

Built-in timezone support ensures accurate event display and synchronization across geographical regions.

- Use the scheduler's `timezone` property to set a **global timezone** — all events display consistently in it regardless of the local timezone.
- Define per-event timezones with the `startTimezone` and `endTimezone` fields.

> **Notes:**
> - Timezone settings are **not** applied to all-day events — they are treated as date-based entries and displayed consistently without conversion.
> - When no global `timezone` is specified, the scheduler uses the **browser/system timezone** to display event times.

### Displaying events from different timezones in one target timezone

Events carrying `startTimezone`/`endTimezone` values (e.g. `America/Chicago`) are automatically converted and displayed in the scheduler's target `timezone`. In this pattern a match at June 16, 2026 8:00 PM **America/Chicago** displays at 1:00 AM UTC on June 17 when `timezone='UTC'`. Let users switch target timezones by binding `timezone` to state:

```tsx
import { DayView, EventSettings, MonthView, Scheduler, TimezoneFields, WeekView } from '@syncfusion/react-scheduler';
import { useState } from 'react';

export default function App() {
    const [timezone, setTimezone] = useState('UTC');
    const timezoneData: TimezoneFields[] = [
        { text: '(UTC-05:00) Eastern Time', value: 'America/New_York' },
        { text: 'Coordinated Universal Time', value: 'UTC' },
        { text: '(UTC+03:00) Moscow+00 - Moscow', value: 'Europe/Moscow' },
        { text: '(UTC+05:30) India Standard Time', value: 'Asia/Kolkata' },
        { text: '(UTC+08:00) Western Time - Perth', value: 'Australia/Perth' }
    ];
    const eventSettings: EventSettings = { dataSource: fifaEventData };

    // Change handler wiring: a DropDownList bound to timezoneData sets `timezone`.

    return (
        <Scheduler
            height={'550px'}
            defaultSelectedDate={new Date(2026, 5, 15)}
            eventSettings={eventSettings}
            timezone={timezone}
        >
            <DayView /><WeekView /><MonthView />
        </Scheduler>
    );
}
```

### Customizing timezone options in the editor

Provide your own list via the `timezoneDataSource` property to replace the built-in options in the Start/End Timezone dropdowns of the editor:

```tsx
import { DayView, MonthView, Scheduler, WeekView, TimezoneFields, EventSettings } from '@syncfusion/react-scheduler';

const editorTimezoneData: TimezoneFields[] = [
    { value: 'America/New_York', text: '(UTC-05:00) Eastern Time' },
    { value: 'UTC', text: 'UTC' },
    { value: 'Asia/Kolkata', text: '(UTC+05:30) India Standard Time' }
];

export default function App() {
    const eventSettings: EventSettings = { dataSource: overviewDataSource };

    return (
        <Scheduler
            height={'550px'}
            eventSettings={eventSettings}
            timezoneDataSource={editorTimezoneData}
            defaultSelectedDate={new Date(2026, 1, 11)}
        >
            <DayView /><WeekView /><MonthView />
        </Scheduler>
    );
}
```

## Accessibility

The scheduler complies with major accessibility standards: [ADA](https://www.ada.gov/), [Section 508](https://www.section508.gov/), and [WCAG 2.2](https://www.w3.org/TR/WCAG22/).

| Accessibility Criteria | Compatibility |
| -- | -- |
| WCAG 2.2 Support | ✅ |
| Section 508 Support | ✅ |
| Screen Reader Support | ✅ |
| Right-To-Left Support | ✅ |
| Color Contrast | ✅ |
| Mobile Device Support | ✅ |
| Keyboard Navigation Support | ✅ |
| Accessibility Validation | ✅ |

### WAI-ARIA attributes

| Selector | Attributes | Purpose |
| --- | --- | --- |
| .sf-scheduler | `role="application"` | Identifies the component as an interactive application. |
| | `aria-label="Scheduler"` | Provides an accessible name for screen readers. |
| | `tabindex="0"` | Enables keyboard focus on the scheduler component. |
| .sf-appointment | `role="button"` | Marks appointments as interactive controls that open event details when activated. |
| | `aria-label` | Announces appointment details (subject, location, time) to assistive technologies. |
| | `tabindex` | Includes appointments in keyboard tab order. |

The scheduler toolbar follows the ToolBar accessibility specification.

### Keyboard interaction

Users can navigate and manage events entirely via keyboard:

| Windows | macOS | Action |
| --- | --- | --- |
| `Shift + Alt + Y` | `Shift + Option + Y` | Jump to today's date. |
| `Ctrl + Left Arrow` | `Command + Left Arrow` | Move to the previous date period. |
| `Ctrl + Right Arrow` | `Command + Right Arrow` | Move to the next date period. |
| `Alt + Number (1–6)` | `Option + Number (1–6)` | Switch between scheduler views. |
| `Home` | `Fn + ←` | Select the first cell in the scheduler. |
| `PageUp` | `←` | Scroll up through the work cells area. |
| `PageDown` | `→` | Scroll down through the work cells area. |
| `Tab` | `Tab` | Move focus through the header bar and event elements. In dialogs, cycle forward through controls. |
| `Shift + Tab` | `Shift + Tab` | Move focus backward through event elements and header bar. In dialogs, cycle backward through controls. |
| `Enter` | `Enter` | Open the quick info popup for the selected cell or event. |
| `Ctrl + Enter` | `Command + Enter` | Save the appointment when the editor window is open. |
| `Delete` | `Delete` | Delete the selected event. |
| `Escape` | `Escape` | Close any open popup or dialog. |

### Disabling keyboard navigation

Set `keyboardNavigation` to `false` to disable all built-in keyboard interactions.
