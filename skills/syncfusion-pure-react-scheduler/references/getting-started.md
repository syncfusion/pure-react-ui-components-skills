# Getting Started with the React Scheduler

Set up the Syncfusion® React Scheduler (`@syncfusion/react-scheduler`) in a React project, then control its date and view.

## Installing the package

```bash
npm install @syncfusion/react-scheduler
```

## Adding the theme CSS

Scheduler themes ship as separate npm theme packages, e.g. Material:

```bash
npm install @syncfusion/react-material-theme --save
```

Reference the theme CSS in `src/App.css`:

```css
@import '../node_modules/@syncfusion/react-material-theme/styles/scheduler/index.css';
```

## Rendering the scheduler

Import `Scheduler` and the view components, then render views as children:

```tsx
import { DayView, WeekView, WorkWeekView, MonthView, Scheduler } from '@syncfusion/react-scheduler';
import './app.css';

export default function App() {
    return (
        <Scheduler>
            <DayView />
            <WeekView />
            <WorkWeekView />
            <MonthView />
        </Scheduler>
    );
}
```

## Populating appointments

Bind appointment data through `eventSettings.dataSource`:

```tsx
import { DayView, WeekView, WorkWeekView, MonthView, Scheduler, EventSettings } from '@syncfusion/react-scheduler';

export default function App() {
    const data = [
        {
            Id: 1,
            Subject: 'Weekly Team Status Meeting',
            StartTime: new Date(2025, 10, 17, 0, 30),
            EndTime: new Date(2025, 10, 17, 2, 0),
            Location: 'Conference Room A',
            Description: 'Regular team status meeting'
        }
    ];
    const eventSettings: EventSettings = { dataSource: data };
    return (
        <Scheduler eventSettings={eventSettings}>
            <DayView />
            <WeekView />
            <WorkWeekView />
            <MonthView />
        </Scheduler>
    );
}
```

## Setting the selected date

The scheduler shows the current system date by default. Both uncontrolled and controlled modes are supported.

**Uncontrolled** — set the initial visible date with `defaultSelectedDate`:

```tsx
<Scheduler defaultSelectedDate={new Date(2025, 10, 18)}>...</Scheduler>
```

**Controlled** — drive the date with `selectedDate` and `onSelectedDateChange`:

```tsx
import { useState } from 'react';
import { Scheduler, SchedulerDateChangeEvent } from '@syncfusion/react-scheduler';

const [selectedDate, setSelectedDate] = useState(new Date(2025, 10, 18));

const onDateChange = (args: SchedulerDateChangeEvent) => setSelectedDate(args.value as Date);

<Scheduler selectedDate={selectedDate} onSelectedDateChange={onDateChange}>...</Scheduler>
```

## Setting the active view

The Week view is active by default. Both modes are supported here too.

**Uncontrolled** — assign a view name to `defaultView`:

```tsx
<Scheduler defaultView='Month'>...</Scheduler>
```

**Controlled** — use `view` with `onViewChange` to tailor the layout based on context:

```tsx
import { useState } from 'react';
import { Scheduler, SchedulerViewChangeEvent } from '@syncfusion/react-scheduler';

const [view, setView] = useState('Week');
const onViewChange = (args: SchedulerViewChangeEvent) => setView(args.value);

<Scheduler view={view} onViewChange={onViewChange}>...</Scheduler>
```

## Complete runnable example

```tsx
import { DayView, WeekView, WorkWeekView, MonthView, Scheduler, EventSettings } from '@syncfusion/react-scheduler';

export default function App() {
    const defaultData = [
        {
            Id: 1,
            Subject: 'Normal: Team Meeting',
            StartTime: new Date(2025, 10, 9, 0, 30),
            EndTime: new Date(2025, 10, 9, 2, 0),
            Location: 'Conference Room A',
            Description: 'Regular team status meeting'
        },
        {
            Id: 2,
            Subject: 'Project Review',
            StartTime: new Date(2025, 10, 14, 1, 30),
            EndTime: new Date(2025, 10, 14, 3, 0),
            Location: 'Conference Room B',
            Description: 'Review project progress and discuss next steps'
        }
    ];
    const eventSettings: EventSettings = { dataSource: defaultData };

    return (
        <Scheduler height='34.375rem' defaultView='Month' eventSettings={eventSettings} defaultSelectedDate={new Date(2025, 10, 11)}>
            <DayView />
            <WeekView />
            <WorkWeekView />
            <MonthView />
        </Scheduler>
    );
}
```

Run the project with `npm run dev`.

## Overview of capabilities

Use these pointers to jump to the right reference when a request goes beyond basic setup:

- **Views:** Day, Week, Work Week, Month, Agenda, and horizontally scrollable Timeline variants, each with configurable intervals and per-view settings (see [views.md](views.md)).
- **Data binding:** local arrays, React state, remote APIs (Fetch/DataManager/OData/Web API/URL adaptors), and load-on-demand (see [events-data.md](events-data.md)).
- **Recurrence:** daily/weekly/monthly/yearly repeat patterns with end conditions, occurrence-level edits, and series splitting (see [recurrence.md](recurrence.md)).
- **CRUD:** dialog/quick-popup editing with field validation, drag-and-drop, and event resizing (see [crud-external-forms.md](crud-external-forms.md) and [editor-and-popups.md](editor-and-popups.md)).
- **Resources:** color-coded resource collections and group-by-resource layouts including multi-level hierarchies (see [resources-grouping.md](resources-grouping.md)).
- **Timezone:** global scheduler timezone and per-event timezone fields (see [globalization-accessibility.md](globalization-accessibility.md)).
- **Globalization:** full localization via L10n/CLDR, custom date/time formats, first day of week, and RTL (see [globalization-accessibility.md](globalization-accessibility.md)).
- **Accessibility:** ARIA attributes, keyboard navigation, and compliance with WCAG 2.2/Section 508/ADA (see [globalization-accessibility.md](globalization-accessibility.md)).
