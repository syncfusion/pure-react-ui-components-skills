# Architecture Overview

The React Scheduler is a comprehensive event management component with flexible, configurable views, powerful data binding, and rich customization capabilities. This guide explains the core architecture and mental model.

## Component hierarchy

```
Scheduler (Container)
├── Views (Children)
│   ├── DayView
│   ├── WeekView
│   ├── MonthView
│   ├── AgendaView
│   ├── TimelineDayView
│   ├── TimelineWeekView
│   ├── TimelineMonthView
│   └── TimelineWorkWeekView
├── EventSettings (Data binding)
│   └── dataSource (Event array or DataManager)
├── Resources (Optional grouping)
│   └── SchedulerResource[] (Room, Team, Staff, etc.)
├── RecurrenceEditor (Built-in recurrence UI)
├── EventTemplates (Custom rendering)
│   ├── eventTemplate
│   ├── headerTemplate
│   ├── cellTemplate
│   ├── dateHeaderTemplate
│   └── tooltipTemplate
└── Callbacks (Lifecycle)
    ├── onDataChangeStart
    ├── onDataChangeComplete
    ├── onSelectedDateChange
    ├── onDataBound
    └── [20+ other event handlers]
```

## Core concepts

### 1. Views as Children

Unlike most libraries, Scheduler views are declarative child components:

```tsx
// ✅ Correct: Views as children
<Scheduler defaultView="Week">
  <DayView />
  <WeekView />
  <MonthView />
</Scheduler>

// ❌ Incorrect: Don't pass views as array prop
<Scheduler views={[DayView, WeekView]} />
```

**Why?** This pattern:
- Allows view-specific prop configuration (different `startHour` per view)
- Enables lazy loading of view modules
- Supports dynamic view registration
- Keeps component tree predictable

### 2. Event data flow

```
Data Source (Array / DataManager)
           ↓
    EventSettings.fields (Mapping)
           ↓
    Normalized Event Models
           ↓
    View Rendering Engine
           ↓
    Templated Event Display
           ↓
    User Interaction (Drag, Edit, Delete)
           ↓
    onDataChangeStart/Complete (Callbacks)
           ↓
    Back to Data Source (Persistence)
```

### 3. Time zones and localization

Scheduler respects browser timezone but allows explicit overrides:

```tsx
<Scheduler
  timezone="Asia/Kolkata"        // Explicit timezone
  locale="en-IN"                  // Localization
  rtl={true}                      // Right-to-left layout
>
  {/* Views */}
</Scheduler>
```

All dates internally use ISO 8601 format. Conversion to user's timezone happens at display time.

## Five view categories

| Category | Views | Layout | Best For |
|----------|-------|--------|----------|
| **Time-based** | Day, Week, WorkWeek | Grid: Time slots × Resource rows | Hourly scheduling |
| **Calendar** | Month | Grid: Weeks × Days | Overview planning |
| **List** | Agenda | Vertical list | Event browsing |
| **Timeline (Horizontal)** | TimelineDay, TimelineWeek, TimelineMonth | Horizontal scrolling timeline | Resource planning, Gantt-like |
| **Custom** | Your own template | Any | Specialized needs |

**Selecting views:**

```tsx
// Common combination: Overview + Detailed
<Scheduler defaultView="Month">
  <MonthView />        {/* High-level overview */}
  <WeekView />         {/* Weekly drilldown */}
  <DayView />          {/* Hourly detail */}
</Scheduler>

// Project timeline focus
<Scheduler defaultView="TimelineMonth">
  <TimelineMonthView interval={3} />  {/* 3-month view */}
  <TimelineWeekView interval={2} />   {/* 2-week drill-in */}
  <TimelineDayView />                 {/* Daily detail */}
</Scheduler>
```

## Data binding architecture

### Two binding modes:

**1. Local data (State-managed)**
```tsx
const [events, setEvents] = useState(initialData);
<Scheduler eventSettings={{ dataSource: events }}>
```
- Events stored in React state
- Changes via `setEvents()`
- Manual persistence to backend

**2. Remote data (Auto-managed)**
```tsx
const data = new DataManager({
  url: 'https://api.example.com/events',
  adaptor: new WebApiAdaptor()
});
<Scheduler eventSettings={{ dataSource: data }}>
```
- Automatic API calls on CRUD
- Handles pagination and filtering
- Real-time synchronization

**Load-on-demand optimization:**
```tsx
// Fetch events only for visible date range
<Scheduler onDataRequest={(args) => {
  fetchEvents(args.startDate, args.endDate);
}}>
```

## Customization layers

```
Layer 1: Configuration (Props)
├── startHour, endHour
├── workDays, workHours
├── timeScale
└── locale, timezone

Layer 2: Templates (Render functions)
├── eventTemplate
├── cellTemplate
├── dateHeaderTemplate
└── tooltipTemplate

Layer 3: Event handlers (Callbacks)
├── onEventClick
├── onCellClick
├── onEventRender
└── onDataChangeStart

Layer 4: Custom components (External)
├── External form (replaces editor)
├── Custom context menu
└── Integration with other libraries
```

**Example: Three-layer customization**

```tsx
// Layer 1: Configuration
<Scheduler
  startHour="06:00"
  endHour="22:00"
  workDays={[1,2,3,4,5]}
  timeScale={{ interval: 30, slotCount: 2 }}
>
  
  {/* Layer 2: Template for event display */}
  <WeekView
    eventTemplate={(props) => (
      <div className={`priority-${props.priority}`}>
        {props.subject}
      </div>
    )}
  />
</Scheduler>

// Layer 3: Callbacks for interactions
<Scheduler
  onEventClick={handleEventClick}
  onDataChangeStart={persistToBackend}
>
```

## Resource and grouping architecture

Resources enable multi-user/multi-facility scheduling:

```
Scheduler
├── Resource 1 (Rooms)
│   ├── Room 1 → [Events for Room 1]
│   ├── Room 2 → [Events for Room 2]
│   └── Room 3 → [Events for Room 3]
├── Resource 2 (Staff)
│   ├── John → [John's events]
│   └── Jane → [Jane's events]
└── Resource 3 (Equipment)
    ├── Projector → [Projector bookings]
    └── Whiteboard → [Whiteboard bookings]
```

**Grouping modes:**

| Mode | Layout | Query |
|------|--------|-------|
| **Vertical** | One resource = one row; stacked vertically | `groupBy="Rooms"` |
| **Horizontal** | One resource = one column; side-by-side | `groupBy="Rooms"` |
| **Hierarchical** | Resources grouped by category | `groupBy="['Rooms', 'Staff']"` |

## CRUD operation flow

```
User Action
   ↓
Scheduler intercepts (onEventClick, onCellClick, etc.)
   ↓
Editor dialog opens (built-in or external)
   ↓
User saves changes
   ↓
onDataChangeStart triggered (validation, persistence)
   ↓
Change applied to Scheduler state
   ↓
onDataChangeComplete triggered (notification, sync)
   ↓
UI re-renders with updated events
```

## Event recurring architecture

```
Recurring Event (parent)
├── recurrenceRule: "FREQ=DAILY;COUNT=5"
├── recurrenceException: "20260202T100000Z"  (skip Feb 2)
└── Instances
    ├── Jan 31 occurrence (normal)
    ├── Feb 1 occurrence (normal)
    ├── Feb 2 occurrence (SKIPPED by exception)
    ├── Feb 3 occurrence (normal)
    └── Feb 4 occurrence (normal)

Exception Event (child)
├── recurrenceID: 1  (references parent)
├── startTime: Feb 2, 11:00 AM  (moved time)
└── Replaces Feb 2 normal occurrence
```

## Performance considerations

### 1. Event rendering optimization
- Only visible events render (virtualization)
- Templates re-render on data change
- Use `useMemo` for expensive templates

### 2. Data binding performance
- Load-on-demand for large datasets
- Client-side caching (Map-based)
- Pagination for remote APIs
- Avoid loading all 10,000 events at once

### 3. View complexity
- Fewer resources = faster rendering
- Simpler templates = faster re-render
- Disable animations for large datasets

## Integration patterns

### With form libraries (react-hook-form)
```tsx
const { control, handleSubmit } = useForm();
// Use Scheduler's onEventClick to populate form
// Use form submission to call addEvent/saveEvent
```

### With state management (Redux/Zustand)
```tsx
// Dispatch to store on onDataChangeStart
// Subscribe to store changes → sync to Scheduler
```

### With third-party APIs (Google Calendar, Outlook)
```tsx
// Fetch events via API
// Map API fields to Scheduler fields
// Two-way sync on changes
```

## Browser compatibility

- **Desktop:** Chrome, Firefox, Safari, Edge (latest 2 versions)
- **Mobile:** iOS Safari, Chrome Mobile
- **Responsive:** Adapts to screen size (full layout shift not needed)
- **Accessibility:** WCAG 2.2 AA compliant with full keyboard navigation

## Memory model

Scheduler maintains:
1. **Event array** — All events in memory
2. **View state** — Current date, selected view type
3. **UI state** — Expanded/collapsed groups, editor dialog state
4. **Cache** — Sorted/filtered event subsets for fast rendering

**Large dataset handling:**
- Virtual scrolling for lists (AgendaView)
- Viewport-based rendering for timeline
- Client-side caching with expiration

## Related concepts

- **Mental model:** See [getting-started.md](./getting-started.md)
- **Available views:** See [views.md](./views.md)
- **Data binding options:** See [events-data.md](./events-data.md)
- **Event field structure:** See [event-fields.md](./event-fields.md)
- **Resource grouping:** See [resources-grouping.md](./resources-grouping.md)
- **Customization:** See [templates-ui.md](./templates-ui.md) and [editor-and-popups.md](./editor-and-popups.md)

## Quick reference: Common tasks

| Task | Approach |
|------|----------|
| Change default date | `selectedDate={date}` prop |
| Restrict working hours | `startHour="09:00"` + `workHours={{ start, end }}` |
| Add custom event fields | Extend event data, access in templates |
| Prevent conflicts | Validate in `onDataChangeStart`, set `event.cancel = true` |
| Resource-based booking | Use `resources` array with `groupBy` mode |
| Replace editor dialog | External form + `addEvent()` / `saveEvent()` refs |
| Performance: large dataset | Load-on-demand + client-side cache |
| Timezone handling | Set `timezone` prop, dates auto-convert |

