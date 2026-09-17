# Event Data, Field Mapping, and Data Binding

Event fields define how appointment data is structured and bound to the scheduler. Bind a local array, React state, or a remote service through `eventSettings.dataSource`, handle change events, and load data on demand for large datasets.

## Contents

- [Built-in event fields](#built-in-event-fields)
- [Binding different field names](#binding-different-field-names)
- [Adding custom fields](#adding-custom-fields)
- [Local data binding](#local-data-binding) — JSON, useEffect, React state
- [Remote data binding](#remote-data-binding) — Fetch API, DataManager, adaptors, headers, cross-domain
- [Load on demand](#load-on-demand) — onDataRequest, DataManager
- [Handling data changes](#handling-data-changes)

## Built-in event fields

| Field name | Description |
| --- | --- |
| id | Mandatory. Assigns a unique ID value to each event. |
| subject | Optional. Assigns the summary text of the event. |
| startTime | Mandatory. Defines the start time of the event. |
| endTime | Mandatory. Defines the end time of the event. |
| location | Optional. Maps the location text value displayed over events. |
| description | Optional. Maps the event description. |
| isAllDay | Denotes whether an event spans an entire day (`true`) or a specific time alone. |
| isReadonly | Makes specific appointments read-only when set to `true`. |
| isBlock | Blocks particular time ranges, preventing event creation on those slots. |

## Binding different field names

If the data source uses field names that differ from the scheduler defaults, map them with `eventSettings.fields` instead of reshaping the data:

```tsx
import { DayView, WeekView, WorkWeekView, MonthView, EventSettings, Scheduler } from '@syncfusion/react-scheduler';

export default function App() {
    const scheduleData = [
        {
            TicketId: 1,
            TravelSummary: 'The Art Walk',
            DepartureTime: new Date(2026, 5, 15, 2, 0),
            ArrivalTime: new Date(2026, 5, 15, 3, 30),
            FullDay: false,
            Source: 'London',
            Comments: 'Summer vacation planned for outstation.',
        },
        {
            TicketId: 2,
            TravelSummary: 'Not available',
            DepartureTime: new Date(2026, 5, 17, 3, 0),
            ArrivalTime: new Date(2026, 5, 17, 5, 0),
            FullDay: false,
            Source: 'Beijing',
            Comments: 'Conference on emerging technologies.',
            IsDisabled: true
        }
    ];
    const eventSettings: EventSettings = {
        dataSource: scheduleData,
        fields: {
            id: 'TicketId',
            subject: 'TravelSummary',
            startTime: 'DepartureTime',
            endTime: 'ArrivalTime',
            isAllDay: 'FullDay',
            location: 'Source',
            description: 'Comments',
            isBlock: 'IsDisabled'
        }
    };

    return (
        <Scheduler height='34.375rem' defaultSelectedDate={new Date(2026, 5, 15)} eventSettings={eventSettings}>
            <DayView />
            <WeekView />
            <WorkWeekView />
            <MonthView />
        </Scheduler>
    );
}
```

Here `TravelSummary` maps to `subject`, `DepartureTime` to `startTime`, and `IsDisabled` to `isBlock` to block those time ranges.

## Adding custom fields

Events can carry custom fields (e.g. `Status`, `Priority`, `IsHoliday`) beyond the built-ins — added directly to the data, no mapping in `eventSettings` required. Custom fields are accessible for internal processing and application logic (including templates, see [templates-ui.md](templates-ui.md)):

```tsx
import { WeekView, EventSettings, Scheduler, EventModel } from '@syncfusion/react-scheduler';

export default function App() {
    const scheduleData = [
        {
            Id: 1,
            Subject: 'Service Check',
            StartTime: new Date(2026, 1, 13, 3, 0),
            EndTime: new Date(2026, 1, 13, 4, 0),
            IsAllDay: false,
            Notes: 'Non-veg',
            Priority: 'Low'
        },
        {
            Id: 2,
            Subject: 'Event Arrangement',
            StartTime: new Date(2026, 1, 8, 1, 0),
            EndTime: new Date(2026, 1, 8, 2, 0),
            IsAllDay: false,
            Notes: 'Veg',
            Priority: 'High'
        }
    ];
    const eventSettings: EventSettings = {
        dataSource: scheduleData,
        fields: { id: 'Id', subject: 'Subject', startTime: 'StartTime', endTime: 'EndTime', isAllDay: 'IsAllDay' }
    };
    const eventTemplate = (event: EventModel) => (
        event.Priority === 'High'
            ? <div className="time-slot-event-template"><div className="sf-subject">{event.subject}</div></div>
            : <div className="time-slot-event"><div className="sf-subject">{event.subject}</div></div>
    );

    return (
        <Scheduler eventSettings={eventSettings} eventDrag={false} eventResize={false}>
            <WeekView eventTemplate={eventTemplate} />
        </Scheduler>
    );
}
```

## Local data binding

### Assigning local JSON data

Bind a JSON array of appointment objects directly to `eventSettings.dataSource` — no network requests, suitable for static datasets and offline applications:

```tsx
import { DayView, WeekView, WorkWeekView, MonthView, Scheduler, EventSettings } from '@syncfusion/react-scheduler';
import { ScheduleJsonData } from './scheduleJsonData';

export default function App() {
    const eventSettings: EventSettings = { dataSource: ScheduleJsonData };
    return (
        <Scheduler defaultSelectedDate={new Date(2025, 6, 15)} eventSettings={eventSettings}>
            <DayView /><WeekView /><WorkWeekView /><MonthView />
        </Scheduler>
    );
}
```

### Loading data with useEffect

Prepare or fetch data after mount, then assign it to `eventSettings.dataSource`:

```tsx
import { useEffect, useState } from 'react';
import { DayView, WeekView, WorkWeekView, MonthView, Scheduler, EventSettings } from '@syncfusion/react-scheduler';

export default function App() {
    const [scheduleData, setScheduleData] = useState([]);
    useEffect(() => {
        const processedData = [
            { Id: 1, Subject: 'Meeting - 1', StartTime: new Date(2025, 9, 30, 10, 0), EndTime: new Date(2025, 9, 30, 12, 30) },
            { Id: 2, Subject: 'Meeting - 2', StartTime: new Date(2025, 9, 30, 11, 0), EndTime: new Date(2025, 9, 30, 14, 30) }
        ];
        setScheduleData(processedData);
    }, []);
    const eventSettings: EventSettings = { dataSource: scheduleData };
    return (
        <Scheduler defaultSelectedDate={new Date(2025, 9, 28)} eventSettings={eventSettings}>
            <DayView /><WeekView /><WorkWeekView /><MonthView />
        </Scheduler>
    );
}
```

### Binding with React state

Connect `dataSource` to a state variable for real-time synchronization; state changes are reflected automatically:

```tsx
import { useState } from 'react';
import { DayView, WeekView, WorkWeekView, MonthView, EventSettings, Scheduler } from '@syncfusion/react-scheduler';
import { meetingEvents } from './dataSource';

export default function App() {
    const [scheduleData, setScheduleData] = useState(meetingEvents);
    const eventSettings: EventSettings = { dataSource: scheduleData };
    return (
        <Scheduler height='34.375rem' eventSettings={eventSettings} defaultSelectedDate={new Date(2025, 9, 30)} startHour='08:00'>
            <DayView /><WeekView /><WorkWeekView /><MonthView />
        </Scheduler>
    );
}
```

> When user edits flow through `onDataChangeStart`, apply `addedRecords`/`changedRecords`/`deletedRecords` back to state — see "Handling data changes" below.

## Remote data binding

### Fetch API with useEffect

Fetch from external services (e.g. `OData`) and assign the result to state:

```tsx
import { DayView, WeekView, WorkWeekView, MonthView, Scheduler, EventSettings } from '@syncfusion/react-scheduler';
import { useEffect, useState } from 'react';
import { Fetch } from '@syncfusion/react-base';

interface Appointment {
    Id: number;
    Subject: string;
    IsAllDay: boolean;
    StartTime: Date;
    EndTime: Date;
}

export default function App() {
    const [events, setEvents] = useState<Appointment[]>([]);
    const url = 'https://ej2services.syncfusion.com/react/hotfix/api/schedule';
    useEffect(() => {
        Fetch(url, 'GET', 'application/json').send?.()
            .then((response: Response) => response as object as Promise<Appointment[]>)
            .then((data) => setEvents(data))
            .catch((error) => console.log(error));
    }, []);
    const eventSettings: EventSettings = { dataSource: events as [] };

    return (
        <Scheduler height='34.375rem' defaultSelectedDate={new Date(2026, 0, 7)} readOnly={true} eventSettings={eventSettings}>
            <DayView /><WeekView /><WorkWeekView /><MonthView />
        </Scheduler>
    );
}
```

### Syncfusion DataManager

`DataManager` (from `@syncfusion/react-data`) connects to remote sources with built-in adaptors that translate between the UI and the data service. Configure a `url` and assign an `adaptor`:

```tsx
import { DataManager, ODataV4Adaptor } from '@syncfusion/react-data';

const data = new DataManager({
    url: 'https://ej2services.syncfusion.com/react/hotfix/api/schedule',
    adaptor: new ODataV4Adaptor()
});
const eventSettings: EventSettings = { dataSource: data };
```

DataManager handles queries for filtering, sorting, and paging, and performs CRUD actions against the service; with remote sources adaptors manage requests and responses, while local arrays are processed directly.

**Custom headers**: add authentication tokens, API keys, or anti-forgery (CSRF) tokens to every request via the `headers` property:

```tsx
const data = new DataManager({
    url: '...',
    adaptor: new WebApiAdaptor(),
    headers: [{ 'Syncfusion': 'true' }]
});
```

**Cross domain**: enable CORS via `crossDomain: true` for third-party services or microservices on different domains.

**Available adaptors** (choose by server contract):

| Adaptor | Server contract |
| --- | --- |
| `UrlAdaptor` | Data operations and CRUD sent in LINQ query format; server returns `{ result: [...], count: N }` |
| `ODataV4Adaptor` | OData V4 endpoints; server returns `{ value: [...], '@odata.count': N }` |
| `WebApiAdaptor` | Web API endpoints (extension of ODataAdaptor), JSON over HTTP; same request format as ODataV4 |

```tsx
// UrlAdaptor example with custom field mapping
import { DataManager, UrlAdaptor } from '@syncfusion/react-data';

const data = new DataManager({
    url: 'https://services.syncfusion.com/js/production/api/UrlDataSource',
    adaptor: new UrlAdaptor()
});
const eventSettings: EventSettings = {
    dataSource: data,
    fields: {
        id: 'EmployeeID',
        subject: 'Employees',
        location: 'ShipCountry',
        description: 'Address',
        startTime: 'OrderDate',
        endTime: 'RequiredDate',
    }
};
```

```tsx
// WebApiAdaptor example (cross-domain)
import { DataManager, WebApiAdaptor } from '@syncfusion/react-data';

const data = new DataManager({
    url: 'https://ej2services.syncfusion.com/react/hotfix/api/schedule',
    adaptor: new WebApiAdaptor(),
    crossDomain: true
});
```

## Load on demand

Load events dynamically based on the active view's date range instead of loading the entire dataset — a significant optimization for large event resources. Two implementations:

### `onDataRequest` event

`onDataRequest` fires before data loads and provides the active view's `startDate`/`endDate`. Fetch only relevant records and assign them to `event.result`; persist add/edit/delete via `onDataChangeStart` (setting `event.cancel = true` on failure reverts changes):

```tsx
import { SchedulerDataRequestEvent, SchedulerDataChangeEvent, Scheduler, DayView, WeekView, WorkWeekView, MonthView } from "@syncfusion/react-scheduler";
import { useCallback } from "react";
import { getCurrentViewEvents, updateEvents } from "./event";

export default function App() {
    const onDataRequest = useCallback(async (event: SchedulerDataRequestEvent) => {
        if (event.startDate && event.endDate) {
            const MS_PER_DAY: number = 86400000;
            event.endDate = new Date(event.endDate.getTime() + MS_PER_DAY);
            event.result = await getCurrentViewEvents(event.startDate, event.endDate);
        }
    }, []);

    const onDataChangeStart = useCallback(async (event: SchedulerDataChangeEvent) => {
        const { addedRecords, changedRecords, deletedRecords } = event;
        try {
            await updateEvents({ addedRecords, changedRecords, deletedRecords });
        } catch (error) {
            event.cancel = true;
            console.error("Data update failed:", error);
        }
    }, []);

    return (
        <Scheduler onDataRequest={onDataRequest} onDataChangeStart={onDataChangeStart}>
            <DayView /><WeekView /><WorkWeekView /><MonthView />
        </Scheduler>
    );
}
```

### DataManager

When configured with a remote service, DataManager automatically sends requests (including the active view's `startDate`/`endDate`) whenever the scheduler needs updated data — on date or view navigation and CRUD operations. Filter server-side using those dates and return only the events belonging to the current view.

## Handling data changes

For any add, edit, or delete action the scheduler invokes `onDataChangeStart` (at the start of modification) and `onDataChangeComplete` (after it finishes or data has been loaded). Use them to update state/UI, show messages, persist changes, hide loaders, and handle success or errors. Both receive:

- `addedRecords` — newly created event records from add operations.
- `changedRecords` — modified event records from update operations.
- `deletedRecords` — deleted event records from delete operations.
- `cancel` — set to `true` to cancel the entire modification and revert all changes.

A typical local-state sync applies deletes, applies changes, and appends adds:

```tsx
const onDataChangeStart = (args: SchedulerDataChangeEvent) => {
    if (args) {
        setDataSource((data: Record<string, any>[]) => data
            .filter((item) => args.deletedRecords?.find((current) => current.Id === item.Id) === undefined)
            .map((item) => args.changedRecords?.find((current) => current.Id === item.Id) || item)
            .concat(args.addedRecords?.map((item) => item) || [])
        );
    }
};
```

Showing success feedback after changes complete:

```tsx
import { Message, Severity } from '@syncfusion/react-notifications';

const dataChangeComplete = (args: SchedulerDataChangeEvent) => {
    if (args.addedRecords && args.addedRecords.length > 0) setMessage('Records added successfully.');
    else if (args.changedRecords && args.changedRecords.length > 0) setMessage('Records changed successfully.');
    else if (args.deletedRecords && args.deletedRecords.length > 0) setMessage('Records deleted successfully.');
    setState(Severity.Success);
};
```
