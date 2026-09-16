# Event Fields and Data Mapping

Event fields define how appointment data is structured and bound to the Scheduler. Use built-in fields for standard properties, map custom data keys to the Scheduler schema (via `eventSettings.fields`), or extend events with custom fields for application-specific logic.

## Built-in Event Fields

The Scheduler event object supports these built-in fields. When your data uses different field names, map them in `eventSettings.fields` (see **Mapping custom field names** below).

| Field Name | Type | Required | Purpose |
|---|---|---|---|
| **id** | number \| string | ✅ Yes | Unique identifier for each event |
| **subject** | string | ❌ No | Event title/summary text |
| **startTime** | Date | ✅ Yes | Event start datetime |
| **endTime** | Date | ✅ Yes | Event end datetime |
| **location** | string | ❌ No | Event location (displays on event card) |
| **description** | string | ❌ No | Event details/notes |
| **isAllDay** | boolean | ❌ No | `true` for all-day events (default: `false`) |
| **isReadonly** | boolean | ❌ No | `true` prevents editing (default: `false`) |
| **isBlock** | boolean | ❌ No | `true` blocks time slots for new events (default: `false`) |
| **recurrenceRule** | string | ❌ No | iCalendar recurrence rule (e.g., `"FREQ=DAILY;BYDAY=MO,WE,FR"`) |
| **recurrenceException** | string | ❌ No | Comma-separated dates of recurrence exceptions |
| **recurrenceID** | number \| string | ❌ No | ID of the parent recurrence series (for exception events) |
| **resourceId** | number \| string | ❌ No | ID of the associated resource (for grouped/multi-resource scheduling) |

## Using Default Field Names

When your data matches the Scheduler's default field names (id, subject, startTime, endTime, etc.), no field mapping is required:

```tsx
import { DayView, WeekView, WorkWeekView, MonthView, Scheduler, EventSettings } from '@syncfusion/react-scheduler';

export default function App() {
  // Data with default field names — no mapping needed
  const events = [
    {
      id: 1,
      subject: 'Team Meeting',
      startTime: new Date(2026, 0, 15, 10, 0),
      endTime: new Date(2026, 0, 15, 11, 0),
      location: 'Conference Room A',
      isAllDay: false
    },
    {
      id: 2,
      subject: 'Holiday',
      startTime: new Date(2026, 0, 16),
      endTime: new Date(2026, 0, 16),
      isAllDay: true
    }
  ];

  const eventSettings: EventSettings = { dataSource: events };

  return (
    <Scheduler
      height="34.375rem"
      eventSettings={eventSettings}
      defaultSelectedDate={new Date(2026, 0, 15)}
    >
      <DayView />
      <WeekView />
      <WorkWeekView />
      <MonthView />
    </Scheduler>
  );
}
```

## Mapping Custom Field Names

If your data uses field names that differ from the Scheduler defaults, map them using `eventSettings.fields`. This is common when:
- Your API returns data with different naming conventions (e.g., `TicketId` instead of `id`)
- You're integrating with an existing database schema
- You want to avoid reshaping data before binding

**Example: Travel booking system with custom field names**

```tsx
import { DayView, WeekView, WorkWeekView, MonthView, Scheduler, EventSettings } from '@syncfusion/react-scheduler';

export default function App() {
  // Data with custom field names (e.g., from external API)
  const events = [
    {
      TicketId: 1,
      TravelSummary: 'Flight to NYC',
      DepartureTime: new Date(2026, 5, 15, 14, 0),
      ArrivalTime: new Date(2026, 5, 15, 18, 0),
      Source: 'Boston',
      Destination: 'New York',
      TravelClass: 'Business',
      FullDay: false
    },
    {
      TicketId: 2,
      TravelSummary: 'Conference Attendance',
      DepartureTime: new Date(2026, 0, 20),
      ArrivalTime: new Date(2026, 0, 22),
      Source: 'NYC',
      Destination: 'Las Vegas',
      TravelClass: 'Economy',
      FullDay: true
    }
  ];

  // Map custom field names to Scheduler defaults
  const eventSettings: EventSettings = {
    dataSource: events,
    fields: {
      id: 'TicketId',               // Map TicketId → id
      subject: 'TravelSummary',     // Map TravelSummary → subject
      startTime: 'DepartureTime',   // Map DepartureTime → startTime
      endTime: 'ArrivalTime',       // Map ArrivalTime → endTime
      location: 'Source',           // Map Source → location
      description: 'Destination',   // Map Destination → description
      isAllDay: 'FullDay'           // Map FullDay → isAllDay
    }
  };

  return (
    <Scheduler eventSettings={eventSettings} defaultSelectedDate={new Date(2026, 0, 15)}>
      <WeekView />
    </Scheduler>
  );
}
```

**Common mapping scenarios:**

| Backend Field | Maps To | Notes |
|---|---|---|
| `eventId` / `eventCode` | `id` | Must be unique per event |
| `title` / `name` / `summary` | `subject` | Displayed as event title |
| `begin` / `start` / `from` | `startTime` | Convert to Date if string |
| `finish` / `end` / `until` | `endTime` | Must be after startTime |
| `allDay` / `fullDay` / `wholeDay` | `isAllDay` | Boolean flag |
| `readOnly` / `locked` / `protected` | `isReadonly` | Boolean flag |
| `blackout` / `blocked` / `unavailable` | `isBlock` | Boolean flag |
| `repeat` / `rule` / `pattern` | `recurrenceRule` | iCalendar format string |

## Adding custom fields

Extend events with application-specific fields beyond the built-ins. Custom fields are not mapped in `eventSettings.fields`—just add them directly to your event data:

```tsx
import { Scheduler, WeekView, EventSettings, EventModel } from '@syncfusion/react-scheduler';

export default function App() {
  // Events with custom fields (Priority, Status, AssignedTo, Cost)
  const events = [
    {
      id: 1,
      subject: 'Project Review',
      startTime: new Date(2026, 0, 15, 10, 0),
      endTime: new Date(2026, 0, 15, 11, 0),
      // Custom fields (no mapping required)
      priority: 'High',
      status: 'In Progress',
      assignedTo: 'John Doe',
      cost: 500
    },
    {
      id: 2,
      subject: 'Team Training',
      startTime: new Date(2026, 0, 16, 14, 0),
      endTime: new Date(2026, 0, 16, 16, 0),
      priority: 'Medium',
      status: 'Planned',
      assignedTo: 'Jane Smith',
      cost: 200
    }
  ];

  const eventSettings: EventSettings = { dataSource: events };

  // Use custom fields in templates or event handlers
  const eventTemplate = (event: EventModel) => (
    <div style={{
      padding: '5px',
      backgroundColor: event.priority === 'High' ? '#ffcccc' : '#ccffcc'
    }}>
      <div style={{ fontWeight: 'bold' }}>{event.subject}</div>
      <div style={{ fontSize: '12px' }}>
        Priority: {(event as any).priority}
      </div>
      <div style={{ fontSize: '12px' }}>
        Assigned: {(event as any).assignedTo}
      </div>
    </div>
  );

  return (
    <Scheduler eventSettings={eventSettings} defaultSelectedDate={new Date(2026, 0, 15)}>
      <WeekView eventTemplate={eventTemplate} />
    </Scheduler>
  );
}
```

**Custom field examples:**

| Custom Field | Use Case | Type |
|---|---|---|
| `priority` | Task/ticket priority level | 'High' \| 'Medium' \| 'Low' |
| `status` | Event workflow state | 'Draft' \| 'Scheduled' \| 'Completed' |
| `assignedTo` / `ownerID` | Resource assignment | string \| number |
| `category` / `type` | Event classification | string |
| `cost` / `budget` | Financial tracking | number |
| `tags` | Flexible labeling | string[] |
| `customColor` | Event styling | hex color code |
| `metadata` | Application-specific data | JSON object |

## Accessing custom fields in handlers

Custom fields are accessible in all event handlers and templates:

```tsx
import { Scheduler, WeekView, EventSettings } from '@syncfusion/react-scheduler';

export default function App() {
  const events = [
    {
      id: 1,
      subject: 'Critical Update',
      startTime: new Date(2026, 0, 15, 10, 0),
      endTime: new Date(2026, 0, 15, 11, 0),
      severity: 'Critical',
      department: 'IT'
    }
  ];

  const handleEventClick = (args: any) => {
    const event = args.data;
    console.log(`Event: ${event.subject}`);
    console.log(`Severity: ${(event as any).severity}`); // Access custom field
    console.log(`Department: ${(event as any).department}`);
  };

  const handleDataChangeComplete = (args: any) => {
    if (args.addedRecords) {
      args.addedRecords.forEach((event: any) => {
        console.log(`New event: ${event.subject}, Severity: ${event.severity}`);
      });
    }
  };

  return (
    <Scheduler
      eventSettings={{ dataSource: events }}
      onEventClick={handleEventClick}
      onDataChangeComplete={handleDataChangeComplete}
      defaultSelectedDate={new Date(2026, 0, 15)}
    >
      <WeekView />
    </Scheduler>
  );
}
```

## Field mapping for remote data

When binding to remote APIs, map fields based on the API response structure:

```tsx
import { Scheduler, WeekView, EventSettings } from '@syncfusion/react-scheduler';
import { DataManager, WebApiAdaptor } from '@syncfusion/react-data';

export default function App() {
  // API returns: eventCode, eventName, startDate, endDate, location
  const data = new DataManager({
    url: 'https://api.example.com/events',
    adaptor: new WebApiAdaptor(),
    crossDomain: true
  });

  const eventSettings: EventSettings = {
    dataSource: data,
    fields: {
      id: 'eventCode',           // API field: eventCode
      subject: 'eventName',      // API field: eventName
      startTime: 'startDate',    // API field: startDate (must be Date)
      endTime: 'endDate',        // API field: endDate (must be Date)
      location: 'location'       // API field: location
    }
  };

  return (
    <Scheduler eventSettings={eventSettings} defaultSelectedDate={new Date(2026, 0, 15)}>
      <WeekView />
    </Scheduler>
  );
}
```

## Nullable and optional fields

Only `id`, `startTime`, and `endTime` are mandatory. All others can be `null` or `undefined`:

```tsx
const events = [
  {
    id: 1,
    subject: 'Untitled Event', // Can be empty string or omitted
    startTime: new Date(2026, 0, 15, 10, 0), // Required
    endTime: new Date(2026, 0, 15, 11, 0),   // Required
    location: null,           // Optional, can be null
    description: undefined,   // Optional, can be undefined
    isAllDay: false,          // Optional, defaults to false
    isReadonly: false,        // Optional, defaults to false
    isBlock: false            // Optional, defaults to false
  }
];
```

## Type checking custom fields

For TypeScript projects, extend the EventModel interface for type safety:

```tsx
import { EventModel } from '@syncfusion/react-scheduler';

interface CustomEvent extends EventModel {
  priority?: 'High' | 'Medium' | 'Low';
  status?: 'Draft' | 'Scheduled' | 'Completed';
  assignedTo?: string;
  cost?: number;
}

const events: CustomEvent[] = [
  {
    id: 1,
    subject: 'Meeting',
    startTime: new Date(2026, 0, 15, 10, 0),
    endTime: new Date(2026, 0, 15, 11, 0),
    priority: 'High',       // Type-checked custom field
    status: 'Scheduled',    // Type-checked custom field
    assignedTo: 'John',
    cost: 500
  }
];
```

## Field mapping validation checklist

Before finalizing your data binding:

- [ ] All events have a unique `id`
- [ ] All events have `startTime` and `endTime` (not null/undefined)
- [ ] All date/time values are JavaScript `Date` objects (or parseable strings)
- [ ] Field mapping keys match actual data keys (case-sensitive)
- [ ] Custom fields don't conflict with built-in field names
- [ ] API responses include all mapped fields
- [ ] Read-only fields (id, startTime, endTime) are not null

## Related APIs

- [EventSettings.fields](https://react.syncfusion.com/react-ui/scheduler/#eventsettings)
- [EventModel interface](https://react.syncfusion.com/react-ui/scheduler/#eventmodel)
- [Scheduler.eventSettings](https://react.syncfusion.com/react-ui/scheduler/#eventsettings)
- [See events-data.md](./events-data.md) for data binding patterns
- [See external-forms.md](./external-forms.md) for creating events programmatically with custom fields
