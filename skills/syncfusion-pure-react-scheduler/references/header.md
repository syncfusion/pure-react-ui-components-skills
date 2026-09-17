# Header and Toolbar Customization

The React Scheduler's header (toolbar) is highly customizable, enabling you to control navigation, view switching, and add custom controls for domain-specific actions.

## SchedulerHeader component

The core of header customization is the Scheduler's `header` property, which accepts a function that returns a `SchedulerHeader` component. This gives you full control over the toolbar's structure, layout, and behavior.

### SchedulerHeader properties

The `SchedulerHeader` exposes several properties to configure individual toolbar sections:

| Property | Type | Description |
|---|---|---|
| `children` | ReactNode | Renders the default toolbar items in their standard order. |
| `todayProps` | ToolbarItemProps \| null | Properties for the `Today` button (icon, disabled state, variant, styles). Set to `null` to hide. |
| `previousProps` | ToolbarItemProps \| null | Properties for the `Previous` navigation button. Set to `null` to hide. |
| `nextProps` | ToolbarItemProps \| null | Properties for the `Next` navigation button. Set to `null` to hide. |
| `dateRangeProps` | ToolbarItemProps \| null | Properties for the `DateRange` display (format, style). Set to `null` to hide. |
| `dateRangeTemplate` | function | Custom render function for the date range label. |
| `viewSwitcherProps` | ToolbarItemProps \| null | Properties for the `ViewSwitcher` control. Set to `null` to hide. |
| `overflowMode` | OverflowMode | Controls behavior when space is limited: `Scrollable`, `Popup`, or `MultiRow`. |

### Built-in toolbar items

| Item | Purpose | Keyboard Shortcut |
|---|---|---|
| `Today` | Navigate to the current date | Ctrl+T |
| `Previous` | Move to the previous date range | Ctrl+← |
| `Next` | Move to the next date range | Ctrl+→ |
| `DateRange` | Display the currently visible date period | (display only) |
| `ViewSwitcher` | Toggle between views (Day, Week, Month, etc.) | Alt+[1-5] |

## Customizing header layout

Use the `header` prop on `<Scheduler>` to inject a custom render function:

```tsx
<Scheduler
  header={(props: SchedulerHeaderProps) => (
    <SchedulerHeader {...props} overflowMode={OverflowMode.Scrollable}>
      {/* Custom layout */}
    </SchedulerHeader>
  )}
>
```

### Reordering items

Reorder built-in toolbar items by accessing props and placing them in your desired order:

```tsx
const customHeader = (props: SchedulerHeaderProps) => (
  <SchedulerHeader {...props}>
    <ToolbarItem>
      <img src="logo.png" alt="Logo" style={{ height: 35 }} />
      <span>My Scheduler</span>
    </ToolbarItem>
    {props.previous}
    {props.dateRange}
    {props.next}
    <ToolbarSpacer />
    {props.today}
    <ToolbarSeparator />
    {props.viewSwitcher}
  </SchedulerHeader>
);
```

### Adding custom controls

Insert custom buttons or controls by wrapping them in `<ToolbarItem>`:

```tsx
import { ToolbarItem, ToolbarSeparator, ToolbarSpacer } from "@syncfusion/react-navigations";
import { Button } from "@syncfusion/react-buttons";
import { useRef } from "react";

const schedulerRef = useRef<IScheduler>(null);

const customHeader = (props: SchedulerHeaderProps) => (
  <SchedulerHeader {...props}>
    {props.children}
    <ToolbarSeparator />
    <ToolbarItem>
      <Button
        icon={<CircleAddIcon />}
        onClick={() => {
          schedulerRef.current?.openEditor('Add', {
            startTime: new Date(),
            endTime: new Date(new Date().getTime() + 60 * 60 * 1000),
          });
        }}
      >
        New Event
      </Button>
    </ToolbarItem>
    <ToolbarItem>
      <Button icon={<SettingsIcon />} onClick={onOpenSettings}>
        Settings
      </Button>
    </ToolbarItem>
  </SchedulerHeader>
);
```

## Hiding header elements

Minimize the toolbar by hiding specific items or the entire header:

### Hide specific items

Pass `null` to individual props to remove buttons while keeping others:

```tsx
const minimalHeader = (props: SchedulerHeaderProps) => (
  <SchedulerHeader
    {...props}
    todayProps={null}              // Hide Today button
    previousProps={null}           // Hide Previous button
    nextProps={null}               // Hide Next button
    viewSwitcherProps={null}       // Hide View Switcher
  >
    {props.dateRange}
  </SchedulerHeader>
);

<Scheduler header={minimalHeader}>
```

### Hide entire header

Set the Scheduler's `header` property to `false` to remove the toolbar completely:

```tsx
<Scheduler header={false}>
  <DayView />
  <WeekView />
</Scheduler>
```

Alternatively, conditionally apply the header based on state:

```tsx
const [showHeader, setShowHeader] = useState(true);

<Scheduler header={showHeader ? customHeader : false}>
```

## Overflow handling

When the toolbar doesn't fit in the available space, use `overflowMode` to control behavior:

- **`OverflowMode.Scrollable`** - Toolbar scrolls horizontally (default for desktop)
- **`OverflowMode.Popup`** - Overflow items appear in a dropdown menu
- **`OverflowMode.MultiRow`** - Items wrap to multiple rows

```tsx
const customHeader = (props: SchedulerHeaderProps) => (
  <SchedulerHeader 
    {...props} 
    overflowMode={OverflowMode.Popup}
  >
    {props.children}
  </SchedulerHeader>
);
```

## Styling the header

Apply theme-aware and custom styles to header items:

```tsx
import { Variant } from "@syncfusion/react-base";

const styledHeader = (props: SchedulerHeaderProps) => (
  <SchedulerHeader
    {...props}
    todayProps={{ variant: Variant.Outlined }}
    previousProps={{ variant: Variant.Text }}
    nextProps={{ variant: Variant.Text }}
    viewSwitcherProps={{ variant: Variant.Filled }}
  >
    {props.children}
  </SchedulerHeader>
);
```

Wrap the toolbar in a container div to apply CSS classes:

```tsx
const customHeader = (props: SchedulerHeaderProps) => (
  <div style={{ 
    backgroundColor: '#f5f5f5', 
    padding: '8px',
    borderBottom: '1px solid #e0e0e0'
  }}>
    <SchedulerHeader {...props}>{props.children}</SchedulerHeader>
  </div>
);
```

## DateRange template customization

Format or customize the date range display using `dateRangeTemplate`:

```tsx
const customHeader = (props: SchedulerHeaderProps) => {
  const formatDateRange = () => {
    const startDate = props.startDate?.toLocaleDateString();
    const endDate = props.endDate?.toLocaleDateString();
    return `${startDate} → ${endDate}`;
  };

  return (
    <SchedulerHeader
      {...props}
      dateRangeTemplate={() => <span>{formatDateRange()}</span>}
    >
      {props.children}
    </SchedulerHeader>
  );
};
```

## Complete example

Here's a fully customized header combining multiple techniques:

```tsx
import { useRef, useState } from "react";
import { 
  DayView, WeekView, MonthView, WorkWeekView,
  Scheduler, SchedulerHeader, SchedulerHeaderProps, IScheduler
} from "@syncfusion/react-scheduler";
import { OverflowMode, ToolbarItem, ToolbarSeparator, ToolbarSpacer } from "@syncfusion/react-navigations";
import { Button } from "@syncfusion/react-buttons";
import { CircleAddIcon, SettingsIcon } from "@syncfusion/react-icons";
import { Variant } from "@syncfusion/react-base";
import { defaultData } from './dataSource';

export default function App() {
  const schedulerRef = useRef<IScheduler>(null);
  const [showEventCount, setShowEventCount] = useState(false);

  const customHeader = (props: SchedulerHeaderProps) => (
    <SchedulerHeader
      {...props}
      overflowMode={OverflowMode.Scrollable}
      todayProps={{ variant: Variant.Standard }}
      viewSwitcherProps={{ variant: Variant.Standard }}
    >
      <ToolbarItem>
        <span style={{ fontWeight: 'bold', color: '#1a73e8' }}>
          📅 My Scheduler
        </span>
      </ToolbarItem>
      <ToolbarSeparator />
      {props.previous}
      {props.dateRange}
      {props.next}
      <ToolbarSpacer />
      {props.today}
      <ToolbarSeparator />
      {props.viewSwitcher}
      <ToolbarSeparator />
      <ToolbarItem>
        <Button
          variant={Variant.Outlined}
          icon={<CircleAddIcon />}
          onClick={() => schedulerRef.current?.openEditor('Add', {})}
        >
          New Event
        </Button>
      </ToolbarItem>
      <ToolbarItem>
        <Button
          variant={Variant.Text}
          icon={<SettingsIcon />}
          onClick={() => setShowEventCount(!showEventCount)}
        >
          Show Count
        </Button>
      </ToolbarItem>
    </SchedulerHeader>
  );

  return (
    <Scheduler
      ref={schedulerRef}
      height="550px"
      eventSettings={{ dataSource: defaultData }}
      header={customHeader}
    >
      <DayView />
      <WeekView />
      <WorkWeekView />
      <MonthView />
    </Scheduler>
  );
}
```

## Related APIs

- [Scheduler.header](https://react.syncfusion.com/react-ui/scheduler/#prop-header)
- [SchedulerHeader props](https://react.syncfusion.com/react-ui/scheduler/#prop-schedulerheader)
- [ToolbarItem documentation](https://react.syncfusion.com/react-ui/toolbar/#tools)
- [Keyboard navigation shortcuts](./globalization-accessibility.md#keyboard-navigation)
