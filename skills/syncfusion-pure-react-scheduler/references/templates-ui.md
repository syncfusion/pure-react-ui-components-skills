# Templates and Header Customization

The scheduler supports flexible templates to customize events, date headers, cells, and the toolbar/header bar. Use templates to build conditional content, icons, and custom layouts that match the application style.

## Contents

- [Customizing event appearance — `eventTemplate`](#customizing-event-appearance--eventtemplate)
- [Customizing the date header — `dateHeader`](#customizing-the-date-header--dateheader)
- [Customizing the Month cell header — `cellHeader`](#customizing-the-month-cell-header--cellheader)
- [Customizing Month cell appearance — `cell`](#customizing-month-cell-appearance--cell)
- [Customizing the header indent — `headerIndent`](#customizing-the-header-indent--headerindent)
- [Agenda "no events" template — `noEventsTemplate`](#agenda-no-events-template--noeventstemplate)
- [Customizing the header bar — `header` / `SchedulerHeader`](#customizing-the-header-bar--header--schedulerheader) — properties, built-in items, layout, hiding elements

## Customizing event appearance — `eventTemplate`

Override default event rendering per view with `eventTemplate` to create custom event layouts — text, images, icons, links, badges, or buttons, with full control over colors, typography, time format, and structure:

```tsx
import { Scheduler, DayView, WeekView, WorkWeekView, MonthView, EventSettings, EventModel } from '@syncfusion/react-scheduler';

export default function App() {
    const schedulerData = [
        {
            Id: 1, Subject: 'Release Meeting',
            StartTime: new Date('2026-02-12T10:00:00+05:30'),
            EndTime: new Date('2026-02-12T12:30:00+05:30'),
            Location: 'Chennai', EventType: 'Release'
        },
        {
            Id: 2, Subject: 'Customer Escalation Review',
            StartTime: new Date('2026-02-13T08:30:00+05:30'),
            EndTime: new Date('2026-02-13T11:30:00+05:30'),
            Location: 'Kenya', EventType: 'Customer'
        },
        {
            Id: 3, Subject: 'Annual Townhall 2026',
            StartTime: new Date('2026-02-10T09:00:00+05:30'),
            EndTime: new Date('2026-02-10T11:30:00+05:30'),
            Location: 'USA', EventType: 'Annual'
        },
        {
            Id: 4, Subject: 'Leadership Meeting',
            StartTime: new Date('2026-02-09T10:00:00+05:30'),
            EndTime: new Date('2026-02-09T12:30:00+05:30'),
            Location: 'Canada', EventType: 'Leader'
        }
    ];
    const eventSettings: EventSettings = {
        dataSource: schedulerData,
        fields: { id: 'Id', subject: 'Subject', startTime: 'StartTime', endTime: 'EndTime', location: 'Location' }
    };

    // Conditional styling per custom field value (here: EventType)
    const eventTemplate = (props: EventModel & { EventType?: string }) => {
        const styleMap: Record<string, { backColor: string; accent: string }> = {
            Release: { backColor: '#6366f1', accent: '#4f46e5' },
            Customer: { backColor: '#ec4899', accent: '#db2777' },
            Annual: { backColor: '#f59e0b', accent: '#d97706' },
            Leader: { backColor: '#10b981', accent: '#059669' }
        };
        const { backColor, accent } = styleMap[props.EventType ?? ''] ?? { backColor: '#8b5cf6', accent: '#7c3aed' };

        const formatTime = (date: Date) =>
            date.toLocaleTimeString('en-US', { hour: 'numeric', minute: '2-digit', hour12: true });
        const start = props.startTime ? formatTime(new Date(props.startTime)) : '';
        const end = props.endTime ? formatTime(new Date(props.endTime)) : '';
        const timeRange = start && end && start !== end ? `${start} – ${end}` : start || 'All Day';

        return (
            <div className="height-100p overflow-hidden display-flex" style={{ backgroundColor: backColor, width: '100%' }}>
                <div className='absolute' style={{ backgroundColor: accent }} />
                <div className='width-100p overflow-hidden'>
                    <div className='sf-subject sf-ellipsis'>{props.subject}</div>
                    <div className='sf-event-time sf-ellipsis'>{timeRange}</div>
                    <div className='sf-event-location sf-ellipsis'>{props.location}</div>
                </div>
            </div>
        );
    };

    return (
        <Scheduler eventSettings={eventSettings} eventResize={false}>
            <DayView eventTemplate={eventTemplate} startHour="08:00" endHour="20:00" />
            <WeekView eventTemplate={eventTemplate} startHour="08:00" endHour="22:00" />
            <WorkWeekView eventTemplate={eventTemplate} />
            <MonthView eventTemplate={eventTemplate} />
        </Scheduler>
    );
}
```

`eventTemplate` is configured per view, so different views can render events differently. Templates receive the full event object (built-in fields plus custom fields like `EventType` above).

## Customizing the date header — `dateHeader`

Replace the default date header with custom content by combining formatted date text with icons or badges:

```tsx
import { DayView, WeekView, WorkWeekView, Scheduler, EventSettings, SchedulerDateHeaderProps } from '@syncfusion/react-scheduler';
import { formatDate } from '@syncfusion/react-base';

const dateHeader = (props: SchedulerDateHeaderProps) => {
    // e.g. append weather info or badges keyed by day
    return (
        <div className="custom-date-header display-flex">
            <div>{formatDate(props.date, { skeleton: 'Ed' })}</div>
            {/* custom icons / badges based on props.date */}
        </div>
    );
};

<Scheduler dateHeader={dateHeader} eventSettings={eventSettings}>
    <DayView /><WeekView /><WorkWeekView />
</Scheduler>
```

## Customizing the Month cell header — `cellHeader`

Customize the header area inside each Month view cell to display additional context like icons, badges, or counts. The template receives the cell's date:

```tsx
import { MonthView, Scheduler, EventSettings, SchedulerCellHeaderProps } from '@syncfusion/react-scheduler';
import { formatDate } from '@syncfusion/react-base';

const cellHeader = (props: SchedulerCellHeaderProps) => {
    if (props.date.getDate() % 2 === 0) {
        return <div className="custom-cell-header display-flex">{formatDate(props.date, { format: 'MMM dd' })}</div>;
    }
    if (props.date.getDate() % 3 === 0) {
        return <div className="custom-cell-header display-flex">{formatDate(props.date, { format: 'MMM dd' })}</div>;
    }
    return <div className="custom-cell-header display-flex">{formatDate(props.date, { format: 'MMM dd' })}</div>;
};

<Scheduler eventSettings={eventSettings}>
    <MonthView cellHeader={cellHeader} />
</Scheduler>
```

## Customizing Month cell appearance — `cell`

Style and decorate Month view cells based on the date — Syncfusion icons, color tags, or custom classes to highlight weekends, holidays, or event-driven states. Return an empty fragment for cells that should remain unchanged. Check `props.type` (e.g. `"monthCell"`) to target the right cell kind:

```tsx
import { MonthView, Scheduler, EventSettings, SchedulerCellProps } from '@syncfusion/react-scheduler';

const cell = (props: SchedulerCellProps) => {
    if (props.type === "monthCell") {
        return (
            <div className="template-wrap month-more-div">
                {/* per-date custom content: holidays, birthdays, images... */}
                {/* return <></> (empty content) for unchanged cells */}
            </div>
        );
    }
    return (<></>);
};

<Scheduler eventSettings={eventSettings} cell={cell}>
    <MonthView />
</Scheduler>
```

## Customizing the header indent — `headerIndent`

Render custom content (icons, badges, text) in the top-left indent area of the scheduler. When `showWeekNumber` is enabled, the template function receives `HeaderIndentProps` including the current week number. Example — a "more" icon opening a dropdown that switches the `timeScale` interval:

```tsx
import { MonthView, Scheduler, DayView, WeekView, TimeScaleProps } from '@syncfusion/react-scheduler';
import { useState } from 'react';

export default function App() {
    const [selectedInterval, setSelectedInterval] = useState(1);
    const [timeScale, setTimeScale] = useState<TimeScaleProps>({ enable: true, interval: 60, slotCount: 2 });

    const headerIndentTemplate = () => (
        <div className="header-indent-template">
            {/* e.g. a DropDownButton listing slot options; selecting one calls
                setTimeScale({ enable: true, interval: 60, slotCount: value }) */}
        </div>
    );

    return (
        <Scheduler
            eventSettings={eventSettings}
            headerIndent={headerIndentTemplate}
            timeScale={timeScale}
        >
            <DayView /><WeekView /><MonthView />
        </Scheduler>
    );
}
```

## Agenda "no events" template — `noEventsTemplate`

The Agenda view renders "No events" content when the active date range has no events (also for specific empty dates when `hideEmptyAgendaDays` is `false`). Replace the default message with custom icons, text, or action buttons:

```tsx
import { AgendaView, Scheduler, IScheduler } from '@syncfusion/react-scheduler';
import { useRef } from 'react';

export default function App() {
    const schedulerRef = useRef<IScheduler>(null);
    const selectedDate = new Date(2025, 0, 12);

    const onAddEvent = () => {
        const cellData = {
            startTime: new Date(selectedDate.getFullYear(), selectedDate.getMonth(), selectedDate.getDate(), 9, 0),
            endTime: new Date(selectedDate.getFullYear(), selectedDate.getMonth(), selectedDate.getDate(), 10, 0),
            isAllDay: false
        };
        schedulerRef.current?.openEditor('Add', cellData);
    };

    const noEventsTemplate = () => {
        return (
            <div className="no-events-template">
                <div className="no-events-template-content">
                    No events are currently scheduled for this period.
                </div>
                <button onClick={onAddEvent}>Add Event</button>
            </div>
        );
    };

    return (
        <Scheduler ref={schedulerRef} height='480px' defaultSelectedDate={selectedDate}>
            <AgendaView noEventsTemplate={noEventsTemplate} hideEmptyAgendaDays={true} />
        </Scheduler>
    );
}
```

## Customizing the header bar — `header` / `SchedulerHeader`

The scheduler's header bar (toolbar) manages navigation and view switching. The `header` property accepts a function returning a `SchedulerHeader` component, giving full control over the toolbar's structure — reorder default items, insert custom React nodes (buttons, images), or restyle standard navigation controls.

### SchedulerHeader properties

| Prop | Description |
|---|---|
| `children` | Renders the default toolbar items in their standard order. |
| `todayProps` | Properties for the `Today` button (icon, disabled state, variant, styles, or attributes). |
| `previousProps` | Properties for the `Previous` navigation button (icon, appearance, behavior). |
| `nextProps` | Properties for the `Next` navigation button (icon, style, functionality). |
| `dateRangeProps` | Properties for the `DateRange` display (format, style, or custom templates). |
| `dateRangeTemplate` | Custom rendering template for the date range label. |
| `viewSwitcherProps` | Properties for the `ViewSwitcher` control (appearance, available views, interaction). |
| `overflowMode` | How toolbar items behave when space is limited (scrollable, popup, etc.). |

### Built-in toolbar items

| Item | Purpose |
|---|---|
| `Today` | Instantly navigates the view to the current date. |
| `Previous` | Traverses backward through date ranges. |
| `Next` | Traverses forward through date ranges. |
| `DateRange` | A text label showing the currently visible period. |
| `ViewSwitcher` | Toggles between views such as Day, Week, WorkWeek, and Month. |

### Custom header layout

Mix built-in items with custom components using `ToolbarItem`, `ToolbarSeparator`, and `ToolbarSpacer` (from `@syncfusion/react-navigations`) — insert a company logo, add a "New Event" button, reorder navigation arrows:

```tsx
import { OverflowMode, ToolbarItem, ToolbarSeparator, ToolbarSpacer } from "@syncfusion/react-navigations";
import { DayView, IScheduler, MonthView, Scheduler, SchedulerCellDetails, SchedulerHeader, SchedulerHeaderProps, WeekView, WorkWeekView } from "@syncfusion/react-scheduler";
import { useRef } from "react";

export default function App() {
    const schedulerRef = useRef<IScheduler | null>(null);

    const cellData: Partial<SchedulerCellDetails> = {
        startTime: new Date(2025, 0, 12, 10, 0),
        endTime: new Date(2025, 0, 12, 11, 0),
        isAllDay: false
    };

    const handleAddEvent: () => void = () => {
        schedulerRef?.current?.openEditor('Add', cellData);
    };

    const customHeader = (props: SchedulerHeaderProps) => (
        <SchedulerHeader
            {...props}
            overflowMode={OverflowMode.Scrollable}
        >
            <ToolbarItem>
                <span aria-label="Logo" role="img">My Scheduler</span>
            </ToolbarItem>
            {props.previous}
            {props.dateRange}
            {props.next}
            <ToolbarSpacer />
            {props.today}
            <ToolbarSeparator />
            {props.viewSwitcher}
            <ToolbarSeparator />
            <ToolbarItem>
                <button onClick={handleAddEvent}>New Event</button>
            </ToolbarItem>
        </SchedulerHeader>
    );

    return (
        <Scheduler
            ref={schedulerRef}
            header={customHeader}
            defaultSelectedDate={new Date(2025, 0, 12)}
            eventSettings={eventSettings}
        >
            <DayView /><WeekView /><WorkWeekView /><MonthView />
        </Scheduler>
    );
}
```

### Hiding header elements

- **Hide specific items** — pass `null` to props like `todayProps` or `viewSwitcherProps` to remove those buttons while keeping the rest of the toolbar:

```tsx
const customHeader = (props: SchedulerHeaderProps) => (
    <SchedulerHeader
        {...props}
        todayProps={hideToday ? null : props.todayProps}
        previousProps={hidePrevious ? null : props.previousProps}
        nextProps={hideNext ? null : props.nextProps}
        dateRangeProps={hideDateRange ? null : props.dateRangeProps}
        viewSwitcherProps={hideViewSwitcher ? null : props.viewSwitcherProps}
    >
        {props.children}
    </SchedulerHeader>
);
```

- **Hide the entire header** — set the scheduler's `header` property to `false`:

```tsx
<Scheduler header={false} ...>
```
