# Scheduler Views

Day, Week, Work Week, Month, and Agenda views configure the core scheduler layout, with horizontally scrollable Timeline variants available as well. Views are configured as child components of `<Scheduler>`; only rendered views appear; Week is the active view by default.

View components: `DayView`, `WeekView`, `WorkWeekView`, `MonthView`, `AgendaView`, plus `TimelineDayView`, `TimelineWeekView`, `TimelineWorkWeekView`, `TimelineMonthView`.

## Setting the number of visible days

Use `interval` on a view to display multiple consecutive periods. Give each variant a distinct `name` and `displayName` so they appear separately in the view switcher:

```tsx
import { WeekView, DayView, MonthView, EventSettings, Scheduler } from '@syncfusion/react-scheduler';

export default function App() {
    const eventSettings: EventSettings = { dataSource: defaultData };
    return (
        <Scheduler height='34.375rem' eventSettings={eventSettings} defaultView='2weeks' defaultSelectedDate={new Date(2025, 5, 12)}>
            <DayView displayName='2 Days' name='2days' interval={2} />
            <DayView displayName='3 Days' name='3days' interval={3} />
            <DayView displayName='4 Days' name='4days' interval={4} />
            <WeekView displayName='2 Weeks' name='2weeks' interval={2} />
        </Scheduler>
    );
}
```

Set `interval={4}` on `MonthView` to show 4 months for multi-period planning.

## Changing the time interval

`timeScale` sets minutes per main slot (`interval`) and sub-slots within each main slot (`slotCount`), controlling grid density:

```tsx
const timeScale: TimeScaleProps = { enable: true, interval: 60, slotCount: 6 };
<Scheduler timeScale={timeScale}>...</Scheduler>
```

## Flexible working days

Define which days are working days using `workDays` with day indices 0–6, where 0 is Sunday. All other days are treated as weekends. `firstDayOfWeek` controls which day appears first in the header:

```tsx
import { WorkWeekView, Scheduler } from '@syncfusion/react-scheduler';

<Scheduler workDays={[1, 3, 5]} firstDayOfWeek={0}>
    <WorkWeekView />
</Scheduler>
```

`workDays={[1, 3, 5]}` designates Monday, Wednesday, and Friday as working days.

## Flexible working hours

`workHours` defines the business-hour range with `start`/`end` values and an optional `highlight` flag to visually distinguish working hours from non-working time:

```tsx
import { Scheduler, WorkHoursProps } from '@syncfusion/react-scheduler';

const workHours: WorkHoursProps = { highlight: true, start: '07:00', end: '20:00' };
<Scheduler workHours={workHours} startHour='07:00'>...</Scheduler>
```

## Setting visible hours

`startHour` and `endHour` restrict the calendar display to a specific time window:

```tsx
<Scheduler startHour='09:00' endHour='20:00'>...</Scheduler>
```

Combine with `workHours` to highlight core business hours within the narrower visible range.

## Show current time indicator

`showTimeIndicator` displays an animated line aligned with the system clock in Day, Week, and WorkWeek views:

```tsx
<Scheduler showTimeIndicator={true} scrollToSettings={{ enable: true }}>
    <DayView /><WeekView /><WorkWeekView /><MonthView />
</Scheduler>
```

## Month view

### Number of visible months

`MonthView interval` displays multiple consecutive months. Register variants with distinct names:

```tsx
<Scheduler defaultView='month2'>
    <MonthView interval={1} displayName='Month' name='month1' />
    <MonthView interval={2} displayName='2 Months' name='month2' />
    <MonthView interval={4} displayName='1st Quarter' name='month3' />
</Scheduler>
```

### Weeks and rows per view

`displayDate` sets the starting day, `numberOfWeeks` sets how many week rows display, and `maxEventsStack` limits the number of events per day cell:

```tsx
<Scheduler defaultView='week2'>
    <MonthView displayDate={new Date(2025, 0, 12)} maxEventsStack={6} numberOfWeeks={2} displayName='2 Weeks' name='week2' />
    <MonthView displayDate={new Date(2025, 0, 12)} numberOfWeeks={4} displayName='4 Weeks' name='week4' />
</Scheduler>
```

### Week numbers

`showWeekNumber` displays ISO week numbers in the first column. The numbers are governed by `firstDayOfWeek`, and `weekRule` determines what qualifies as the first week of the year (`FirstDay`, `FirstFullWeek`, or `FirstFourDayWeek` from `@syncfusion/react-calendars`):

```tsx
import { WeekRule } from '@syncfusion/react-calendars';

<Scheduler firstDayOfWeek={1} weekRule={WeekRule.FirstDay}>
    <MonthView showWeekNumber={true} />
</Scheduler>
```

### Auto row height

`rowAutoHeight` dynamically expands rows based on the number of appointments so all events are visible without clipping:

```tsx
<Scheduler rowAutoHeight={true}>...</Scheduler>
```

### Hide leading and trailing dates

`showTrailingAndLeadingDates` toggles dates from adjacent months. Set it to `false` to show only the current month's dates:

```tsx
<MonthView showTrailingAndLeadingDates={false} />
```

## Agenda view

The Agenda view renders a compact, date-wise list of events instead of a time grid — ideal for mobile and narrow displays.

### Number of visible dates

`agendaDaysCount` specifies how many dates to render starting from the selected date (defaults to 7), while `interval` determines how many consecutive `agendaDaysCount` periods to render (defaults to 1). Example: `agendaDaysCount={3}` and `interval={2}` displays 6 dates total; navigating with the previous or next arrows jumps forward or backward by the total number of dates shown (in this case, 6 dates at a time):

```tsx
<AgendaView agendaDaysCount={3} interval={2} />
```

### Hiding empty and weekend dates

- `hideEmptyAgendaDays` (default `true`) hides dates with no scheduled events; set `false` to show every date in the range.
- `showWeekend` (default `true`) includes weekend dates; set `false` to display only the days defined in `workDays`.

```tsx
<AgendaView hideEmptyAgendaDays={true} showWeekend={true} />
```

## Timeline views

Timeline views (`TimelineDayView`, `TimelineWeekView`, `TimelineWorkWeekView`, `TimelineMonthView`) are horizontally scrollable and display events across a continuous horizontal timeline. They support the same `interval`, `timeScale`, `workDays`, `workHours`, `startHour`/`endHour`, and `rowAutoHeight` settings described above:

```tsx
import { Scheduler, TimelineDayView, TimelineWeekView, TimelineWorkWeekView, TimelineMonthView } from '@syncfusion/react-scheduler';

<Scheduler defaultView="TimelineWeek" startHour="08:00" endHour="18:00">
    <TimelineDayView displayName="2 Days" interval={2} />
    <TimelineWeekView displayName="2 Weeks" interval={2} />
    <TimelineMonthView displayName="2 Months" interval={2} />
</Scheduler>
```

### Header rows

Timeline views support extra header rows — `Year`, `Month`, `Week` — in addition to the default `Date` and `Hour` rows. This feature applies only to timeline views, and when `headerRows` is set the scheduler renders only the specified rows:

```tsx
<Scheduler defaultView="TimelineMonth" headerRows={[
    { option: 'Year' },
    { option: 'Month' },
    { option: 'Week' },
    { option: 'Date' },
    { option: 'Hour' }
]}>
    <TimelineDayView />
    <TimelineMonthView interval={6} />
</Scheduler>
```

Each row accepts a `template` for custom content (e.g., formatted text, badges). Note: the `Date` header is customized through the scheduler's `dateHeader` property; time-scale headers are customized via `majorSlot`/`minorSlot` templates inside `timeScale`. See [templates-ui.md](templates-ui.md) for details.
