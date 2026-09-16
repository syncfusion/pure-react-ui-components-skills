# Recurring Events and the Recurrence Editor

Recurring events repeat at regular intervals — daily, weekly, monthly, or yearly — based on a recurrence rule following the [iCalendar](https://tools.ietf.org/html/rfc5545#section-3.3.10) specification. Recurring events display a repeat marker in the lower-right corner.

## Contents

- [Creating a recurring event](#creating-a-recurring-event)
- [Recurrence options and rules](#recurrence-options-and-rules) — rule properties, daily/weekly/monthly/yearly examples
- [Adding exceptions to a recurring event](#adding-exceptions-to-a-recurring-event)
- [Editing an occurrence in a series](#editing-an-occurrence-in-a-series)
- [Editing current and following events](#editing-current-and-following-events)
- [Recurrence validation](#recurrence-validation)
- [Recurrence Editor component](#recurrence-editor-component) — generating rules, setting rules, customization, date generation
- [Remote data with recurring events](#remote-data-with-recurring-events)

## Creating a recurring event

Assign an iCalendar rule string to the event's `recurrenceRule` field. The example below repeats daily, ending after five occurrences:

```tsx
const schedulerData = [
    {
        Id: 1,
        Subject: 'Scrum Meeting',
        StartTime: new Date(2026, 0, 25, 11, 0),
        EndTime: new Date(2026, 0, 25, 11, 30),
        RecurringRule: 'FREQ=DAILY;INTERVAL=2;COUNT=3'
    }
];

const eventSettings: EventSettings = {
    dataSource: schedulerData,
    fields: {
        id: 'Id',
        subject: 'Subject',
        startTime: 'StartTime',
        endTime: 'EndTime',
        recurrenceRule: 'RecurringRule'
    }
};
```

## Recurrence options and rules

A rule combines:

- The repeat type (daily, weekly, monthly, or yearly)
- The end type (count, until, or never)
- The interval between occurrences
- The time period during which appointments should be generated

| Recurrence Type | Description |
|-----------------|-------------|
| Daily       | Creates recurring instances every day. |
| Weekly      | Creates recurring instances every week on selected days. |
| Monthly     | Creates recurring instances each month based on selected dates or recurrence patterns. |
| Yearly      | Creates recurring instances once a year. |

### Recurrence rule properties

| Property     | Purpose | Example |
|--------------|---------|---------|
| `FREQ`       | Specifies the repeat frequency (Daily, Weekly, Monthly, Yearly). | FREQ=DAILY;INTERVAL=1 |
| `INTERVAL`   | Defines the interval between instances. | FREQ=DAILY;INTERVAL=2 |
| `COUNT`      | Specifies the total number of occurrences. | FREQ=DAILY;INTERVAL=1;COUNT=10 |
| `UNTIL`      | Indicates the end date of the recurrence series (ISO format). | FREQ=DAILY;INTERVAL=1;UNTIL=20260530T041343Z |
| `BYDAY`      | Specifies the day(s) on which weekly recurrences occur. | FREQ=WEEKLY;INTERVAL=1;BYDAY=MO,WE |
| `BYMONTHDAY` | Specifies the day of the month for monthly recurrences. | FREQ=MONTHLY;BYMONTHDAY=3;INTERVAL=1 |
| `BYMONTH`    | Specifies the month index for yearly recurrences. | FREQ=YEARLY;BYMONTHDAY=16;BYMONTH=6;INTERVAL=1 |
| `BYSETPOS`   | Specifies the week index in a month (e.g., 2nd, 3rd). | FREQ=MONTHLY;BYDAY=MO;BYSETPOS=2 |

### Daily frequency examples

| Description | Example |
|------------|---------|
| Daily recurring event that never ends | FREQ=DAILY;INTERVAL=1 |
| Daily recurring event that ends after 5 occurrences | FREQ=DAILY;INTERVAL=1;COUNT=5 |
| Daily recurring event that ends exactly on 12/12/2026 | FREQ=DAILY;INTERVAL=1;UNTIL=20261212T041343Z |
| Daily event that recurs on alternate days and repeats for 10 occurrences | FREQ=DAILY;INTERVAL=2;COUNT=10 |
| Daily recurring appointment that ends on 12/12/2026 by excluding a single occurrence on 12/10/2026 | FREQ=DAILY;INTERVAL=2;UNTIL=20261212T041343Z |

### Weekly frequency examples

| Description | Example |
|-------------|---------|
| Repeats every Monday, Wednesday, and Friday; never ends | FREQ=WEEKLY;INTERVAL=1;BYDAY=MO,WE,FR |
| Repeats every Thursday; ends after 10 occurrences | FREQ=WEEKLY;INTERVAL=1;BYDAY=TH;COUNT=10 |
| Repeats every Monday; ends on 12/12/2026 | FREQ=WEEKLY;INTERVAL=1;BYDAY=MO;UNTIL=20261212T041343Z |
| Repeats on Monday, Wednesday, and Friday on alternate weeks; ends after 10 occurrences | FREQ=WEEKLY;INTERVAL=2;BYDAY=MO,WE,FR;COUNT=10 |
| Repeats every weekday; ends on 12/12/2026 | FREQ=WEEKLY;BYDAY=MO,TU,WE,TH,FR;UNTIL=20261212T041343Z |

### Monthly frequency examples

| Description | Example |
|-------------|---------|
| Repeats on the 15th day of each month; never ends | FREQ=MONTHLY;BYMONTHDAY=15;INTERVAL=1 |
| Repeats on the 16th day of each month; ends after 10 occurrences | FREQ=MONTHLY;BYMONTHDAY=16;INTERVAL=1;COUNT=10 |
| Repeats on the 17th day of each month; ends on 12/12/2026 | FREQ=MONTHLY;BYMONTHDAY=17;INTERVAL=1;UNTIL=20261212T041343Z |
| Repeats every 2nd Friday of each month; never ends | FREQ=MONTHLY;BYDAY=FR;BYSETPOS=2;INTERVAL=1 |
| Repeats every 4th Wednesday of each month; ends after 10 occurrences | FREQ=MONTHLY;BYDAY=WE;BYSETPOS=4;INTERVAL=1;COUNT=10 |
| Repeats every 4th Friday of each month; ends on 12/12/2026 | FREQ=MONTHLY;BYDAY=FR;BYSETPOS=4;INTERVAL=1;UNTIL=20261212T041343Z |

### Yearly frequency examples

| Description | Example |
|-------------|---------|
| Repeats on the 15th of December every year; never ends | FREQ=YEARLY;BYMONTHDAY=15;BYMONTH=12;INTERVAL=1 |
| Repeats on the 10th of December; ends after 10 occurrences | FREQ=YEARLY;BYMONTHDAY=10;BYMONTH=12;INTERVAL=1;COUNT=10 |
| Repeats on the 12th of December; ends on 12/12/2026 | FREQ=YEARLY;BYMONTHDAY=12;BYMONTH=12;INTERVAL=1;UNTIL=20261212T041343Z |
| Repeats on the 3rd Friday of December; never ends | FREQ=YEARLY;BYDAY=FR;BYMONTH=12;BYSETPOS=3;INTERVAL=1 |
| Repeats on the 3rd Tuesday of December; ends after 10 occurrences | FREQ=YEARLY;BYDAY=TU;BYMONTH=12;BYSETPOS=3;INTERVAL=1;COUNT=10 |
| Repeats on the 4th Wednesday of December; ends on 12/12/2026 | FREQ=YEARLY;BYDAY=WE;BYMONTH=12;BYSETPOS=4;INTERVAL=1;UNTIL=20261212T041343Z |

## Adding exceptions to a recurring event

Exclude specific occurrences by adding their ISO-formatted datetime values to the `recurrenceException` field. Format: no hyphens, time in UTC with `Z` appended and no spaces. Example — 22nd February 2026 is `20260222`, and 07:30:00 UTC becomes `073000Z`:

```tsx
const schedulerData = [
    {
        Id: 1,
        Subject: 'Scrum Meeting',
        StartTime: new Date(2026, 0, 28, 10, 0),
        EndTime: new Date(2026, 0, 28, 11, 0),
        RecurrenceRule: 'FREQ=DAILY;INTERVAL=1;COUNT=8',
        RecurrenceException: '20260129T043000Z,20260131T043000Z,20260202T043000Z'
    }
];

const eventSettings: EventSettings = {
    dataSource: schedulerData,
    fields: {
        id: 'Id',
        subject: 'Subject',
        startTime: 'StartTime',
        endTime: 'EndTime',
        recurrenceRule: 'RecurrenceRule',
        recurrenceException: 'RecurrenceException'
    }
};
```

## Editing an occurrence in a series

To modify a single occurrence and display it when the scheduler loads, add the edited occurrence as a **separate event** in the `dataSource` with a `recurrenceID` referencing the Id of the original recurring event. The occurrence's original date must also be added to the parent's `recurrenceException`:

```tsx
const schedulerData = [
    {
        Id: 1,
        Subject: 'Scrum Meeting',
        StartTime: new Date(2026, 0, 28, 10, 0),
        EndTime: new Date(2026, 0, 28, 11, 0),
        RecurrenceRule: 'FREQ=DAILY;INTERVAL=1;COUNT=8',
        // The edited occurrence's date (Jan 30) is excluded from the parent series
        RecurrenceException: '20260129T043000Z,20260131T043000Z,20260202T043000Z'
    },
    {
        Id: 2,
        Subject: 'Board Meeting',
        StartTime: new Date(2026, 0, 28, 12, 0),
        EndTime: new Date(2026, 0, 28, 12, 30),
        // Links this edited occurrence to the parent series (Id: 1)
        RecurrenceId: 1
    }
];

const eventSettings: EventSettings = {
    dataSource: schedulerData,
    fields: {
        id: 'Id',
        subject: 'Subject',
        startTime: 'StartTime',
        endTime: 'EndTime',
        recurrenceRule: 'RecurrenceRule',
        recurrenceID: 'RecurrenceId',
        recurrenceException: 'RecurrenceException'
    }
};
```

## Editing current and following events

When `eventSettings.editFollowingEvents` is enabled, the recurrence alert popup offers an option to edit the current and **following** occurrences. The scheduler divides the original series into two parts:

1. **Original series:** adds or updates the `UNTIL` value in the `recurrenceRule` to end the series before the changes start.
2. **New series:** a new event is created from the selected occurrence, with updated details and a `recurrenceRule` inherited from the parent.
3. **Data link:** the new series gets a `followingId` referencing the original event's `id`.

In the example below, the original series ends June 3 and a new series continues from June 4 with updated timings:

```tsx
import { DayView, WeekView, WorkWeekView, MonthView, Scheduler, EventSettings, SchedulerDataChangeEvent } from '@syncfusion/react-scheduler';
import { useState } from 'react';

export default function App() {
    const scheduleData: Record<string, any>[] = [
        {
            Id: 1,
            Subject: 'Doctor Consultation (check-up)',
            StartTime: new Date(2026, 5, 1, 11, 0),
            EndTime: new Date(2026, 5, 1, 12, 0),
            IsAllDay: false,
            // The original series is concluded on June 3rd
            RecurrenceRule: 'FREQ=DAILY;INTERVAL=1;UNTIL=20260603T063000Z;',
        },
        {
            Id: 2,
            Subject: 'Doctor Consultation (check-up) - Rescheduled',
            StartTime: new Date(2026, 5, 4, 12, 0),
            EndTime: new Date(2026, 5, 4, 13, 0),
            IsAllDay: false,
            // The new series continues the pattern from June 4th
            RecurrenceRule: 'FREQ=DAILY;INTERVAL=1;UNTIL=20260605T113000Z;',
            // Links this new series to the original parent series (Id: 1)
            FollowingId: 1
        }
    ];
    const [dataSource, setDataSource] = useState<Record<string, any>[]>(scheduleData);
    const [enableFollowingEdit, setEnableFollowingEdit] = useState<boolean>(true);
    const eventSettings: EventSettings = { dataSource: dataSource, editFollowingEvents: enableFollowingEdit };

    const onDataChangeStart = (args: SchedulerDataChangeEvent) => {
        if (args) {
            setDataSource((data: Record<string, any>[]) => data
                .filter((item) => args.deletedRecords?.find((current) => current.Id === item.Id) === undefined)
                .map((item) => args.changedRecords?.find((current) => current.Id === item.Id) || item)
                .concat(args.addedRecords?.map((item) => item) || [])
            );
        }
    };

    return (
        <Scheduler
            height={'550px'}
            eventSettings={eventSettings}
            defaultSelectedDate={new Date(2026, 5, 5)}
            scrollToSettings={{ enable: true }}
            onDataChangeStart={onDataChangeStart}
        >
            <DayView /><WeekView /><WorkWeekView /><MonthView />
        </Scheduler>
    );
}
```

With `editFollowingEvents` enabled, double-clicking any occurrence shows the **Following Events** option.

## Recurrence validation

Built-in validation runs during creation, editing, dragging, and resizing of recurring appointments:

| Validation message | Description |
|--------------------|-------------|
| The recurrence pattern is not valid. | The selected recurrence rule is invalid — e.g. an "Until" date chosen before the start date. |
| The changes made to specific instances of this series will be cancelled and those events will match the series again. | Editing an entire series when one or more occurrences were already modified individually. |
| The duration of the event must be shorter than how frequently it occurs. Shorten the duration, or change the recurrence pattern in the recurrence event editor. | The event's duration exceeds the selected repeat frequency — e.g. a two-day appointment with a Daily pattern and no interval. |
| Some months have fewer than the selected date. For these months, the occurrence will fall on the last date of the month. | Creating a recurring appointment on the 31st of each month; shorter months fall back to their last day. |
| Two occurrences of the same event cannot occur on the same day. | Moving or editing a single occurrence onto a date where another occurrence of the same series already exists. |

## Recurrence Editor component

The standalone `RecurrenceEditor` builds and customizes repeat patterns interactively, also following the iCalendar specification.

### Generate a recurrence rule

When any field changes, `onChange` fires with the updated rule in `e.value`:

```tsx
import { RecurrenceChangeEvent, RecurrenceEditor } from "@syncfusion/react-scheduler";
import { useState } from "react";

export default function App() {
    const [ruleOutput, setRuleOutput] = useState<string>('');
    const onChange = (e: RecurrenceChangeEvent): void => setRuleOutput(e.value);

    return (
        <RecurrenceEditor value={ruleOutput} onChange={onChange} startDate={new Date(2026, 0, 17, 10, 0, 0)} />
    );
}
```

### Set a recurrence rule

Provide an iCalendar string via `value` and the editor loads its initial settings automatically (use a `key` to force re-init when switching preset rules):

```tsx
const [ruleOutput, setRuleOutput] = useState<string>('FREQ=DAILY;INTERVAL=2;COUNT=8');

const ruleData = [
    'FREQ=DAILY;INTERVAL=1',
    'FREQ=DAILY;INTERVAL=2;UNTIL=20410606T000000Z',
    'FREQ=DAILY;INTERVAL=2;COUNT=8',
    'FREQ=WEEKLY;BYDAY=MO,TU,WE,TH,FR;INTERVAL=1;UNTIL=20410729T000000Z',
    'FREQ=MONTHLY;BYDAY=FR;BYSETPOS=2;INTERVAL=1;UNTIL=20410729T000000Z',
    'FREQ=MONTHLY;BYDAY=FR;BYSETPOS=2;INTERVAL=1',
    'FREQ=YEARLY;BYDAY=MO;BYSETPOS=-1;INTERVAL=1;COUNT=5'
];

<RecurrenceEditor key={ruleOutput} value={ruleOutput} startDate={new Date(2026, 1, 2, 10, 0, 0)} />
```

### Editor customization

Limit the repeat-type dropdown with `frequencies` (defaults: `DAILY`, `WEEKLY`, `MONTHLY`, `YEARLY`) and the end-type dropdown with `endTypes` (defaults: `Never`, `Count`, `Until`), keeping the editor simple:

```tsx
<RecurrenceEditor value={ruleOutput} onChange={onChange} startDate={new Date(2026, 1, 17, 10, 0, 0)} frequencies={['WEEKLY', 'MONTHLY']} />
```

```tsx
<RecurrenceEditor value={ruleOutput} onChange={onChange} startDate={new Date(2026, 1, 17, 10, 0, 0)} endTypes={['Count', 'Until']} />
```

### Recurrence date generation

Two utilities turn a rule into actual dates:

- `getRecurrenceDates(startDate, rule, excludeDate?, maximumCount?, viewDate?)` — returns all matching occurrence dates.
- `getRecurrenceSummary(rule, locale)` — returns a human-readable summary of the rule.

| Prop | Type | Description |
|---|---|---|
| `startDate` | `Date` | The start date for recurrence generation. |
| `rule` | `String` | The recurrence rule determining frequency, pattern, and end conditions. |
| `excludeDate` | `String` | Optional. Collection of ISO-formatted dates to exclude from the recurrence. |
| `maximumCount` | `Number` | Optional. How many recurrence dates should be generated. |
| `viewDate` | `Date` | Optional. Starting date of the current view range. |

```tsx
import { getRecurrenceDates, getRecurrenceSummary, RecurrenceChangeEvent, RecurrenceEditor } from '@syncfusion/react-scheduler';
import { useMemo, useState } from 'react';

export default function App() {
    const locale: string = "en-US";
    const startDate: Date = new Date(2026, 1, 17, 10, 0, 0);
    const [ruleOutput, setRuleOutput] = useState<string>('FREQ=DAILY;INTERVAL=1;COUNT=5;');
    const onChange = (e: RecurrenceChangeEvent): void => setRuleOutput(e.value);

    const recurrenceDateCollection = useMemo(() => {
        const dates: number[] = getRecurrenceDates(startDate, ruleOutput, '', 0);
        return dates.map(d => new Date(d).toDateString()).join('\n');
    }, [startDate, ruleOutput]);

    // Render: rule output, getRecurrenceSummary(ruleOutput, locale), and recurrenceDateCollection
}
```

### Remote data with recurring events

When using a remote DataManager (see [events-data.md](events-data.md)), recurrence fields `recurrenceRule`, `recurrenceID`, and `recurrenceException` map through `eventSettings.fields` exactly as with local data:

```tsx
import { DataManager, WebApiAdaptor } from '@syncfusion/react-data';

const data = new DataManager({
    url: 'https://ej2services.syncfusion.com/react/hotfix/api/schedule',
    adaptor: new WebApiAdaptor(),
    crossDomain: true
});
const eventSettings: EventSettings = { dataSource: data };
```
