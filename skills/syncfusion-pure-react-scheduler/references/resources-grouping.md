# Resources and Resource Grouping

Resources assign events to entities such as people, rooms, doctors, or equipment. Configure resource collections with `resources`, then optionally organize the schedule into structured sections with the `group` property.

## Contents

- [Configuring resources](#configuring-resources)
- [Resources without grouping](#resources-without-grouping)
- [Resource grouping basics](#resource-grouping-basics) — the `group` settings
- [Horizontal grouping](#horizontal-grouping) — hierarchical, date-first, sequential
- [Timeline grouping](#timeline-grouping) — hierarchical, sequential
- [Multi-resource shared events](#multi-resource-shared-events)

## Configuring resources

| Property | Type | Description |
| ---------- | ---- | ----------- |
| `name` | `string` | A unique identifier for each resource collection, used for resource grouping. |
| `field` | `string` | Name of the field in the event data that stores the resource ID. |
| `dataSource` | `Object[]` | The list of resource data objects. |
| `multiple` | `boolean` | Allows selecting multiple resources; creates separate events for each selected resource. |
| `title` | `string` | The label text displayed for the resource input in the event editor window. |
| `idField` | `string` | Name of the field that contains the unique ID of the resource. |
| `textField` | `string` | Name of the field that contains the display label for the resource. |
| `colorField` | `string` | Name of the field that contains the color for events associated with the resource. |
| `groupIDField` | `string` | Name of the field that stores the parent resource ID, for parent-child structures. |
| `cssClassField` | `string` | Name of the field that specifies a custom `className` for styling the resource. |

Events link to resources when the event's `field` value matches a resource's `idField` value:

```tsx
const resources: SchedulerResource[] = [{
    name: 'Owners', field: 'OwnerId', title: 'Owner',
    textField: 'OwnerText', idField: 'Id', colorField: 'OwnerColor',
    dataSource: [
        { OwnerText: 'Nancy', Id: 1, OwnerColor: '#ffaa00' },
        { OwnerText: 'Steven', Id: 2, OwnerColor: '#f8a398' }
    ]
}];

const schedulerData = [
    {
        Id: 1,
        Subject: 'Meeting',
        StartTime: new Date(2025, 9, 30, 10, 0),
        EndTime: new Date(2025, 9, 30, 12, 30),
        // Matches the 'Id' value (idField) of a resource -> links this event to Nancy
        OwnerId: 1
    }
];
```

## Resources without grouping

When `resources` is configured without the `group` property, the scheduler renders all resources' events together in a single, default calendar layout (no resource-specific columns/rows). Each resource uses a unique color so users can quickly identify which resource each event belongs to:

```tsx
import { DayView, WeekView, WorkWeekView, MonthView, AgendaView, Scheduler, SchedulerResource, SchedulerDataChangeEvent } from '@syncfusion/react-scheduler';

export default function App() {
    const resources: SchedulerResource[] = [
        {
            name: 'Owners',
            dataSource: [
                { OwnerText: 'Nancy', Id: 1, OwnerColor: '#ffaa00' },
                { OwnerText: 'Steven', Id: 2, OwnerColor: '#f8a398' },
                { OwnerText: 'Michael', Id: 3, OwnerColor: '#7499e1' }
            ],
            field: 'OwnerId',
            title: 'Owner',
            textField: 'OwnerText',
            idField: 'Id',
            colorField: 'OwnerColor',
            multiple: false
        }
    ];

    return (
        <Scheduler
            eventSettings={{ dataSource: resourceData }}
            resources={resources}
            startHour='09:00'
        >
            <DayView /><WeekView /><WorkWeekView /><MonthView /><AgendaView />
        </Scheduler>
    );
}
```

## Resource grouping basics

Define the `group` property to organize the layout by resource. Its settings:

- `resources`: an array of resource category **names**. The scheduler creates the grouping hierarchy based on the order of names in this array. Default: `[]`.
- `byDate`: when `true`, dates are prioritized in the visual hierarchy (dates on top, resource collections underneath). Default: `false`.
- `byGroupID`: controls how relationships between resource levels are created. When `true` (default), child resources are associated with parent resources by matching `groupIDField` and `idField` values. When `false`, every child resource is repeated under each parent resource.

Simple grouping by one resource collection:

```tsx
<Scheduler
    eventSettings={{ dataSource: resourceData }}
    resources={resources}  // the 'Owners' collection defined above
    group={{ resources: ['Owners'] }}
>
    <DayView /><WeekView /><WorkWeekView /><MonthView /><AgendaView />
</Scheduler>
```

## Horizontal grouping

Horizontal grouping organizes resources into structured sections across Day, Week, Work Week, Month, and Agenda views, so appointments display based on the associated resource or combination.

### Hierarchical grouping

Set `byGroupID` to `true` (the default) for a parent-child hierarchy: give each parent a unique `idField` value and include a matching `groupIDField` value in the child-level resource. The scheduler detects these links and displays a nested structure. Note that `idField` values need only be unique within their own collection:

```tsx
import { Scheduler, DayView, WeekView, MonthView, WorkWeekView, AgendaView, SchedulerResource, EventSettings, SchedulerResourceHeaderProps } from '@syncfusion/react-scheduler';

const resources: SchedulerResource[] = [
    {
        name: 'Departments',
        dataSource: [
            { DepartmentText: 'Cardiology', Id: 1, DepartmentColor: '#e3165b' },
            { DepartmentText: 'Neurology', Id: 2, DepartmentColor: '#1aaa55' }
        ],
        field: 'DepartmentId',
        title: 'Department',
        textField: 'DepartmentText',
        idField: 'Id',
        colorField: 'DepartmentColor'
    },
    {
        name: 'Doctors',
        dataSource: [
            { DoctorText: 'Dr. Alice', Id: 1, GroupId: 1, DoctorColor: '#ff6e40' },
            { DoctorText: 'Dr. Bob', Id: 2, GroupId: 2, DoctorColor: '#7cb342' },
            { DoctorText: 'Dr. Charlie', Id: 3, GroupId: 1, DoctorColor: '#29b6f6' }
        ],
        field: 'DoctorId',
        title: 'Specialist',
        textField: 'DoctorText',
        idField: 'Id',
        groupIDField: 'GroupId',
        colorField: 'DoctorColor'
    }
];
const groupedResources: string[] = ['Departments', 'Doctors'];

// Optional: customize resource headers (last level gets an avatar image)
const resourceHeader = (props: SchedulerResourceHeaderProps) => {
    const resourceName: string = props.resourceData[props.resource.textField];
    const isLastLevelResource: boolean = props.resource.name === groupedResources[groupedResources.length - 1];
    return (
        <div className='resource-header sf-align-center sf-ellipsis gap-8'>
            {isLastLevelResource && <img src={`.../${props.resourceData.avatar}.png`} alt={resourceName} />}
            <span>{resourceName}</span>
        </div>
    );
};

<Scheduler
    eventSettings={eventSettings}
    resources={resources}
    resourceHeader={resourceHeader}
    group={{ resources: groupedResources }}
>
    <DayView /><WeekView /><WorkWeekView /><MonthView /><AgendaView />
</Scheduler>
```

### Date-first grouping

Set `byDate: true` to place dates at the primary, top level of the hierarchy with resource collections displayed underneath — ideal for date-first layouts:

```tsx
<Scheduler
    eventSettings={{ dataSource: movieEvents, fields: { subject: 'MovieName' } }}
    resources={[
        {
            name: 'Theatres',
            dataSource: [
                { Id: 1, Theatre: 'PVR Cinemas', Color: '#1e3a8a' },
                { Id: 2, Theatre: 'Regal', Color: '#d97706' }
            ],
            field: 'TheatreId', title: 'Theatre', textField: 'Theatre',
            idField: 'Id', colorField: 'Color'
        },
        {
            name: 'Screens',
            dataSource: [
                { Id: 1, Screen: 'IMAX', GroupID: 1, Color: '#10b981' },
                { Id: 2, Screen: 'VIP', GroupID: 2, Color: '#84cc16' },
                { Id: 3, Screen: 'Elite', GroupID: 1, Color: '#a855f7' }
            ],
            field: 'ScreenId', title: 'Screen', textField: 'Screen', groupIDField: 'GroupID',
            idField: 'Id', colorField: 'Color'
        }
    ]}
    group={{ resources: ['Theatres', 'Screens'], byDate: true }}
    defaultView="Day"
>
    <DayView /><WeekView /><WorkWeekView /><MonthView /><AgendaView />
</Scheduler>
```

### Sequential grouping

Set `byGroupID: false` for stacked header rows based purely on collection order: the first resource collection becomes the top-level header, and every item of the next collection repeats under each top-level resource — with no parent-child ID mapping:

```tsx
<Scheduler
    eventSettings={{
        dataSource: airportGroupingData,
        fields: {
            id: 'FlightId',
            subject: 'FlightName',
            description: 'AircraftDetails',
            startTime: 'ArrivalTime',
            endTime: 'DepartureTime'
        }
    }}
    resources={[
        {
            name: 'Terminals',
            dataSource: [
                { TerminalText: 'Terminal 1', Id: 1, TerminalColor: '#005ea6' },
                { TerminalText: 'Terminal 2', Id: 2, TerminalColor: '#e67e22' }
            ],
            field: 'TerminalId', title: 'Terminal Hub',
            textField: 'TerminalText', idField: 'Id', colorField: 'TerminalColor'
        },
        {
            name: 'Gates',
            dataSource: [
                { GateText: 'Gate 1', Id: 1, GateColor: '#2ecc71' },
                { GateText: 'Gate 2', Id: 2, GateColor: '#3498db' }
            ],
            field: 'GateId', title: 'Arrival Gate',
            textField: 'GateText', idField: 'Id', colorField: 'GateColor'
        }
    ]}
    group={{ resources: ['Terminals', 'Gates'], byGroupID: false }}
    defaultView="Day"
    startHour='06:00' endHour='22:00'
>
    <DayView /><WeekView /><WorkWeekView /><MonthView /><AgendaView />
</Scheduler>
```

## Timeline grouping

Timeline resource grouping displays resources in **rows**, with dates and time slots extending horizontally — useful for comparing events across resources over a continuous time range. Timeline views use the same grouping configuration (`TimelineDayView`, `TimelineWeekView`, `TimelineWorkWeekView`, `TimelineMonthView`):

```tsx
<Scheduler
    eventSettings={eventSettings}
    resources={resources}
    group={{ resources: ['Departments', 'Doctors'] }}
    defaultView="TimelineWeek"
>
    <TimelineDayView />
    <TimelineWeekView />
    <TimelineWorkWeekView />
    <TimelineMonthView />
</Scheduler>
```

- **Hierarchical**: `byGroupID: true` (default) with matching `groupIDField`/`idField` — nested resource rows (the same Departments/Doctors hierarchy from horizontal grouping works unchanged in timeline views).
- **Sequential**: `byGroupID: false` — stacked resource rows by collection order, children repeated under each parent (the same Terminals/Gates pattern, on a horizontal timeline).

## Multi-resource shared events

A single shared event can be associated with multiple resources while maintaining **one event record** in the data source — useful for shared appointments, meetings, activities, or tasks.

Requirements:

1. Set the resource's `multiple` property to `true` so users can assign multiple resources in the event editor.
2. Set the `groupEdit` property to `true` in the `group` configuration. The scheduler then stores a single event object and displays it separately for each assigned resource.
3. Store an array of resource IDs in the event's resource field.

Any change to a shared event — editing, dragging, resizing, or deleting — is automatically synchronized across all displayed instances:

```tsx
import { DayView, WeekView, WorkWeekView, MonthView, AgendaView, Scheduler, SchedulerResource } from '@syncfusion/react-scheduler';
import { resourceConferenceData } from './dataSource';

const resources: SchedulerResource[] = [
    {
        name: 'Conferences',
        dataSource: [
            { Text: 'Margaret', Id: 1, Color: '#1aaa55' },
            { Text: 'Robert', Id: 2, Color: '#357cd2' },
            { Text: 'Laura', Id: 3, Color: '#7fa900' }
        ],
        field: 'ConferenceId',
        title: 'Attendees',
        textField: 'Text',
        idField: 'Id',
        colorField: 'Color',
        multiple: true
    }
];

export default function App() {
    return (
        <Scheduler
            height={'550px'}
            defaultSelectedDate={new Date(2026, 6, 23)}
            defaultView="WorkWeek"
            startHour="04:00"
            eventSettings={{ dataSource: resourceConferenceData }}
            resources={resources}
            group={{
                resources: ['Conferences'],
                groupEdit: true
            }}
        >
            <DayView /><WeekView /><WorkWeekView /><MonthView /><AgendaView />
        </Scheduler>
    );
}
```

Here `ConferenceId: [1, 2, 3]` on an event assigns it to Margaret, Robert, and Laura — the same shared event appears across all three resources while remaining a single record.

> The `multiple` property also works without `groupEdit`: setting `multiple` to `true` lets users select several resources in the editor, and the scheduler creates a **separate copy** of the event for each selected resource.
