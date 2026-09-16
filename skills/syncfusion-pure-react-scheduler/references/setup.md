# Setup — React Scheduler Installation & First Steps

Start here if you're new to the Syncfusion React Scheduler or adding it to an existing React project.

This guide covers:
- Installing the npm package
- Adding theme CSS
- Rendering your first Scheduler
- Choosing controlled vs uncontrolled date/view modes

---

## Prerequisites

- **Node.js & npm** — A working Node.js environment
- **React 16.8+** — Functional components with hooks support
- **TypeScript (optional)** — The package includes full type definitions

---

## Step 1: Create a React Project

If you don't have a React project yet, create one using Vite (recommended for fast development):

```bash
npm create vite@latest my-scheduler -- --template react-ts
cd my-scheduler
npm install
```

Other options: Create React App (`npx create-react-app`), Next.js, Remix, etc. — the Scheduler works with all.

---

## Step 2: Install the Scheduler Package

Install the `@syncfusion/react-scheduler` package:

```bash
npm install @syncfusion/react-scheduler
```

This package includes:
- All Scheduler components and views
- Full TypeScript type definitions
- Event handling and configuration APIs
- No dependencies on external scheduling libraries

---

## Step 3: Add Theme CSS

The Scheduler requires CSS styling. Theme availability differs by package:

### Material Theme (Recommended)

Install the dedicated Material theme package:

```bash
npm install @syncfusion/react-material-theme --save
```

Add to your main CSS file (e.g., `src/App.css` or `src/index.css`):

```css
@import '@syncfusion/react-material-theme/styles/scheduler/index.css';
```

**Use the dedicated theme package for Material theme** — it keeps your theme setup consistent with Bootstrap and Tailwind (both only available as dedicated packages).

> **Note:** Material theme CSS is *also* bundled inside the components package (`@syncfusion/react-scheduler/styles/material/scheduler.css`), but this source only works for Material — Bootstrap and Tailwind do not offer it. Prefer the dedicated theme package so all three themes follow the same install pattern and if material is import from components package, then needs to import for its dependency package also.

### Bootstrap Theme

Bootstrap theme is **only available** in the dedicated package:

```bash
npm install @syncfusion/react-bootstrap-theme --save
```

Add to your main CSS file (e.g., `src/App.css` or `src/index.css`):

```css
@import '@syncfusion/react-bootstrap-theme/styles/scheduler/index.css';
```

> **Important:** Bootstrap theme is NOT available in the components package. You **must** install and import from `@syncfusion/react-bootstrap-theme`.

### Tailwind Theme

Tailwind theme is **only available** in the dedicated package:

```bash
npm install @syncfusion/react-tailwind-theme --save
```

Add to CSS:

```css
@import '@syncfusion/react-tailwind-theme/styles/scheduler/index.css';
```

> **Important:** Tailwind theme is NOT available in the components package. You **must** install and import from `@syncfusion/react-tailwind-theme`.

### Theme Availability Summary

| Theme | Components Package | Dedicated Theme Package | Install Command |
|-------|:-:|:-:|---|
| Material | ✓ (Material only) | ✓ | `npm install @syncfusion/react-material-theme` |
| Bootstrap | ✗ | ✓ | `npm install @syncfusion/react-bootstrap-theme` |
| Tailwind | ✗ | ✓ | `npm install @syncfusion/react-tailwind-theme` |

> **Troubleshooting:** If Bootstrap or Tailwind themes don't apply properly, verify you installed the **dedicated theme package**, not just tried importing from the components package. Importing from the wrong package is the most common cause of theme failures. Only Material is bundled in the components package — Bootstrap and Tailwind are exclusive to their dedicated theme packages.

---

## Step 4: Render Your First Scheduler

Create a minimal Scheduler in `src/App.tsx`:

```tsx
import { Scheduler, DayView, WeekView, WorkWeekView, MonthView, AgendaView, EventSettings } from '@syncfusion/react-scheduler';
import './App.css';

export default function App() {
  // Sample event data
  const data = [
    {
      Id: 1,
      Subject: 'Team Standup',
      StartTime: new Date(2026, 0, 12, 10, 0),
      EndTime: new Date(2026, 0, 12, 10, 30)
    },
    {
      Id: 2,
      Subject: 'Project Review',
      StartTime: new Date(2026, 0, 13, 14, 0),
      EndTime: new Date(2026, 0, 13, 15, 0)
    }
  ];

  // Configure event data
  const eventSettings: EventSettings = { dataSource: data };

  return (
    <Scheduler
      height="500px"
      eventSettings={eventSettings}
      defaultSelectedDate={new Date(2026, 0, 12)}
    >
      <DayView />
      <WeekView />
      <WorkWeekView />
      <MonthView />
      <AgendaView />
    </Scheduler>
  );
}
```

**Run the development server:**

```bash
npm run dev
```

Open `http://localhost:5173` (or the URL printed) — you should see a calendar with events and a view switcher.

---

## Step 5: Understand the Mental Model

The Scheduler works in three parts:

### 1. Views as Child Components

Render views as JSX children of `<Scheduler>`:

```tsx
<Scheduler>
  <DayView />           {/* Daily view */}
  <WeekView />          {/* 7-day week view */}
  <WorkWeekView />      {/* Mon-Fri work week */}
  <MonthView />         {/* Full month grid */}
  <AgendaView />        {/* List/agenda format */}
  <TimelineMonthView /> {/* Horizontal resource timeline */}
</Scheduler>
```

- Only rendered views appear in the view switcher
- The **Week view** is active by default
- View-specific settings (like `startHour`, `workDays`) go on the view; general settings go on `<Scheduler>`

### 2. Events from `eventSettings.dataSource`

Pass event data to the Scheduler via the `eventSettings` prop:

```tsx
const data = [
  {
    Id: 1,
    Subject: 'Meeting',
    StartTime: new Date(2026, 0, 12, 10, 0),
    EndTime: new Date(2026, 0, 12, 11, 0)
  }
];

const eventSettings: EventSettings = { dataSource: data };

<Scheduler eventSettings={eventSettings}>
  {/* views */}
</Scheduler>
```

**Default event fields:**
- `Id` — Unique identifier
- `Subject` — Event title
- `StartTime` — Start datetime (JavaScript `Date` object)
- `EndTime` — End datetime (JavaScript `Date` object)
- Optional: `Description`, `Location`, `IsAllDay`, `RecurrenceRule`, `RecurrenceID`, `RecurrenceException`, etc.

If your data uses different field names, map them in `eventSettings.fields` (see [data-model.md](data-model.md)).

### 3. Customization is Function Props

Customize the editor dialog, quick popup, header, templates, etc. by passing React components:

```tsx
<Scheduler
  eventSettings={eventSettings}
  onEventRendered={(args) => {
    // Custom event rendering logic
  }}
>
  {/* views */}
</Scheduler>
```

No internal state mutation — everything is controlled through props and callbacks.

---

## Step 6: Control Date and View (Optional)

By default, the Scheduler uses uncontrolled mode:
- **Date:** Today's date
- **View:** Week view

To control these explicitly (for React state synchronization):

### Controlled Date

```tsx
import { useState } from 'react';

export default function App() {
  const [selectedDate, setSelectedDate] = useState<Date>(new Date(2026, 0, 12));

  return (
    <Scheduler
      selectedDate={selectedDate}
      onSelectedDateChange={(args) => setSelectedDate(args.value)}
      eventSettings={eventSettings}
    >
      {/* views */}
    </Scheduler>
  );
}
```

### Controlled View

```tsx
import { useState } from 'react';

export default function App() {
  const [view, setView] = useState<string>('Week');

  return (
    <Scheduler
      view={view}
      onViewChange={(args) => setView(args.value)}
      eventSettings={eventSettings}
    >
      {/* views */}
    </Scheduler>
  );
}
```

### Both Date and View

```tsx
const [selectedDate, setSelectedDate] = useState<Date>(new Date(2026, 0, 12));
const [view, setView] = useState<string>('Week');

return (
  <Scheduler
    selectedDate={selectedDate}
    onSelectedDateChange={(args) => setSelectedDate(args.value)}
    view={view}
    onViewChange={(args) => setView(args.value)}
    eventSettings={eventSettings}
  >
    {/* views */}
  </Scheduler>
);
```

Or use **uncontrolled** defaults:

```tsx
<Scheduler
  defaultSelectedDate={new Date(2026, 0, 12)}
  defaultView="Week"
  eventSettings={eventSettings}
>
  {/* views */}
</Scheduler>
```

---

## Step 7: Next Steps

Now that you have a working Scheduler:

| Goal | Read |
|---|---|
| Configure views (hours, work days, intervals) | [views.md](views.md) |
| Bind to remote data or load on demand | [events-data.md](events-data.md) |
| Add CRUD (create, edit, delete events) | [crud-operations.md](crud-operations.md) |
| Support recurring events | [recurrence.md](recurrence.md) |
| Group by resources (rooms, staff) | [resources-grouping.md](resources-grouping.md) |
| Customize editor or quick popup | [customization.md](customization.md) |
| Add drag-and-drop or context menu | [interactions.md](interactions.md) |
| Support multiple languages or timezones | [globalization.md](globalization.md) or [timezone.md](timezone.md) |

---

## Common Setup Issues

### Issue: Calendar renders empty

**Cause:** Event field names don't match. The Scheduler expects `Id`, `Subject`, `StartTime`, `EndTime` by default.

**Solution:** If your data uses different names (e.g., `event_id`, `title`, `begin_time`, `finish_time`), map them:

```tsx
const eventSettings: EventSettings = {
  dataSource: data,
  fields: {
    id: 'event_id',
    subject: 'title',
    startTime: 'begin_time',
    endTime: 'finish_time'
  }
};
```

See [data-model.md](data-model.md) for the complete field mapping reference.

### Issue: CSS not loading or styles look broken

**Cause:** Theme CSS not imported or conflicting with other stylesheets.

**Solution:**
1. **Verify you're using the correct package for your theme:**
   - **Material:** Recommended — `@syncfusion/react-material-theme` (also bundled in the components package, but only for this theme)
   - **Bootstrap:** Must use `@syncfusion/react-bootstrap-theme` (NOT available in components package)
   - **Tailwind:** Must use `@syncfusion/react-tailwind-theme` (NOT available in components package)
   
   Check installed packages: `npm ls | grep syncfusion`

2. Verify the CSS import is in your main CSS file (e.g., `src/App.css` or `src/index.css`), not a component file

3. Check browser DevTools to confirm the CSS file is loading (Network tab)

4. If Bootstrap or Tailwind aren't applying:
   - Confirm the dedicated theme package is installed: `npm ls @syncfusion/react-bootstrap-theme` or `npm ls @syncfusion/react-tailwind-theme`
   - Verify the import path matches the package: `@syncfusion/react-bootstrap-theme/styles/scheduler/index.css` (not the components package)
   - Clear cache: `npm cache clean --force` and rebuild

5. If conflicts with other CSS, load theme CSS **before** your own CSS in the import order

### Issue: Date/time formats are wrong or in wrong timezone

**Cause:** Locale or timezone not configured.

**Solution:** See [globalization.md](globalization.md) for locale setup or [timezone.md](timezone.md) for timezone handling.

### Issue: Can't edit events / editor doesn't open

**Cause:** CRUD not set up, or editor customization is overriding default behavior.

**Solution:** See [crud-operations.md](crud-operations.md) for how to enable editing.

---

## TypeScript Types

If using TypeScript, import types from `@syncfusion/react-scheduler`:

```tsx
import {
  Scheduler,
  EventSettings,
  EventModel,
  SchedulerDateChangeEvent,
  SchedulerViewChangeEvent,
  IScheduler,
  SchedulerCellDetails,
  DayView,
  WeekView,
  WorkWeekView,
  MonthView,
  AgendaView
} from '@syncfusion/react-scheduler';

const eventSettings: EventSettings = { dataSource: data };

const handleDateChange = (args: SchedulerDateChangeEvent) => {
  console.log('New date:', args.value);
};

const handleViewChange = (args: SchedulerViewChangeEvent) => {
  console.log('New view:', args.value);
};
```

---

## What's Included in the Package

The `@syncfusion/react-scheduler` package provides:

**Components:**
- `<Scheduler>` — Main container
- Views: `<DayView />`, `<WeekView />`, `<WorkWeekView />`, `<MonthView />`, `<AgendaView />`
- Timeline views: `<TimelineDayView />`, `<TimelineWeekView />`, `<TimelineMonthView />`
- `<RecurrenceEditor />` — Standalone recurrence rule editor

**Configuration:**
- `EventSettings` — Data binding configuration
- `SchedulerResource` — Resource/grouping configuration
- Event callbacks: `onEventRendered`, `onEventClick`, `onDataChangeStart`, etc.
- Styling and theming via CSS imports

**Methods (via ref):**
- `openEditor()` — Programmatically open event editor
- `deleteEvent()` — Delete event(s)
- `saveEvent()` — Save event programmatically
- `addEvent()` — Add new event
- `getEventDetails()` — Retrieve event data
- `getCellDetails()` — Get cell/slot details

See [events-api.md](../references/events-api.md) for complete API reference.

---

## Version & Support

- **Package:** `@syncfusion/react-scheduler` (latest stable)
- **React:** 16.8+ (hooks support required)
- **Themes:** Material, Bootstrap, Tailwind available via npm
- **Licensing:** Syncfusion license required; trial available at [syncfusion.com](https://www.syncfusion.com/react-components/react-scheduler)

---

## Summary

You now have:
- ✓ Scheduler package installed
- ✓ Theme CSS loaded
- ✓ First Scheduler rendering
- ✓ Sample data displaying
- ✓ View switcher functional

Next, configure your specific use case using the reference files linked above. For a complete end-to-end example, see [examples.md](examples.md).
