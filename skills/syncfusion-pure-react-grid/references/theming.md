---
name: theming
description: Theme application and CSS-variable customization for the Syncfusion React Data Grid — Material / Bootstrap / Tailwind built-in themes, light/dark mode, `--sf-*` variables (per-theme variable catalogs), custom-theme pattern, and the CSS import line. Load when applying or overriding a theme, switching light/dark, or generating a custom theme.
---

# Theming

The grid ships with three built-in theme packages and a unified `--sf-*` CSS-variable convention. Theme switching is one CSS import away; deeper customization is via the same variables.

## Built-in themes

| Package | Variants |
|---|---|
| `@syncfusion/react-material-theme` | Material **light** + Material **dark** |
| `@syncfusion/react-bootstrap-theme` | Bootstrap light + dark |
| `@syncfusion/react-tailwind-theme` | Tailwind light + dark |

Default theme is **Material** (light).

## Install and import

```bash
npm install @syncfusion/react-material-theme --save
# or @syncfusion/react-bootstrap-theme / @syncfusion/react-tailwind-theme
```

In `src/App.css`:

```tsx
@import "../node_modules/@syncfusion/react-material-theme/styles/grid/index.css";
```

Per the getting-started docs:

> Ensure CSS styles are imported in the correct dependency order. CSS must be imported once at the app entry.

## Light/dark toggle

Toggle the theme variant via the CSS file's `data-theme` (theme libraries ship the CSS scoped):

```tsx
<html data-theme="dark">
  <App />
</html>
```

Or via a CSS class on a wrapper:

```tsx
<div className={isDark ? 'sf-dark-mode' : 'sf-light-mode'}>
  <Grid ... />
</div>
```

The exact mechanism depends on the theme package; consult `@syncfusion/react-material-theme` docs for runtime switching.

## Customizing via `--sf-*` variables

All themes expose CSS variables for runtime override. Scope variables to a parent CSS class to localize.

### Material

```css
:root {
  --sf-font-family: 'Inter', sans-serif;
  --sf-font-size: 14px;

  --sf-color-primary: #1f6feb;
  --sf-color-primary-container: #dde7ff;
  --sf-color-background: #ffffff;
  --sf-color-on-surface: #1f2328;
  --sf-color-line-color: #e6e8eb;
  --sf-color-surface: #f7f8fa;
  --sf-color-primary-text-color: #ffffff;
  --sf-color-primary-light: #cfe1ff;

  --scrollbar-thumb: #cccfd4;
  --scrollbar-thumb-hover: #b6b9be;
}
```

### Bootstrap

```css
:root {
  --sf-font-family: 'Roboto', sans-serif;
  --sf-font-size: 14px;

  --sf-color-primary: #0d6efd;
  --sf-color-emphasis-color: #0a58ca;
  --sf-color-body-bg: #ffffff;
  --sf-color-body-color: #212529;
  --sf-color-border-light: #dee2e6;
  --sf-color-primary-bg-color-hover: #cfe2ff;
  --sf-color-primary-border-color-hover: #9ec5fe;
  --sf-color-primary-color: #084298;
  --sf-color-primary-text-color: #ffffff;
  --sf-color-primary-light: #cfe2ff;
  --sf-color-table-bg-color-selected-hover: #cfe2ff;

  --scrollbar-thumb: #ced4da;
  --scrollbar-thumb-hover: #adb5bd;
}
```

### Tailwind

```css
:root {
  --sf-font-family: 'Inter', sans-serif;
  --sf-font-size: 14px;

  --sf-color-primary: #2563eb;
  --sf-color-content-bg-color: #ffffff;
  --sf-color-surface: #f9fafb;
  --sf-color-surface-alt2: #f3f4f6;
  --sf-color-content-bg-color-alt3: #e5e7eb;
  --sf-color-content-bg-color-hover: #f3f4f6;
  --sf-color-content-bg-color-pressed: #e5e7eb;
  --sf-color-table-bg-color-selected-hover: #dbeafe;
  --sf-color-content-text-color: #111827;
  --sf-color-content-text-color-alt1: #6b7280;
  --sf-body-color: #1f2937;
  --sf-color-primary-text-color: #ffffff;
  --sf-color-primary-light: #bfdbfe;

  --scrollbar-thumb: #d1d5db;
  --scrollbar-thumb-hover: #9ca3af;
}
```

## Custom-theme pattern

```tsx
import './grid-custom-theme.css';   // overrides --sf-* variables

<div className="custom-grid-css">
  <Grid ... />
</div>
```

```css
.custom-grid-css {
  --sf-color-primary: #5b21b6;
  --sf-color-primary-text-color: #ffffff;
  --sf-color-primary-light: #ede9fe;
  --sf-color-background: #fbfaff;
}

.custom-grid-css .sf-cell {
  font-family: 'JetBrains Mono', monospace;
  font-size: 13px;
}
```

Scope your overrides to a top-level wrapper so you can theme one grid in your app without leaking.

## Per-cell override via CSS classes

Combine `--sf-*` variables with a `cellClass` callback for fine-grained control:

```tsx
const getStatusClass = (props?: CellClassProps) => {
  if (props?.cellType !== CellType.Content) return '';
  switch ((props?.data as Record)?.status) {
    case 'Critical': return 'status-critical';
    case 'Warning':  return 'status-warning';
    default:         return 'status-default';
  }
};

<Column field="status" cellClass={getStatusClass} />
```

```css
.sf-grid .status-critical {
  color: #dc2626;
  font-weight: 600;
  background: var(--sf-color-primary-light);  /* uses current theme's primary-light */
}
```

## Common patterns

| Need | Configuration |
|---|---|
| Material light | `@syncfusion/react-material-theme/styles/grid/index.css` |
| Material dark | Same import + `<html data-theme="dark">` (or wrapper class) |
| Custom palette | Override `--sf-color-primary` etc. on a parent class |
| Per-cell accent | `cellClass` callback + CSS rules using `--sf-*` variables |
| Brand fonts | `--sf-font-family` override |
| Custom hover colors | `--sf-color-content-bg-color-hover` etc. |

## Constraints & guardrails

- **CSS import order matters** — theme styles must come after framework resets. Put the `@import` at the top of `App.css`.
- **Override scope carefully** — placing `--sf-*` variables on `:root` affects **every** Syncfusion grid in the app. Use a parent CSS class for surgical theming.
- **Dark mode toggle** requires the theme package's `data-theme` mechanism; verify with the official docs per theme.
- **`cellClass` callbacks run on every render** — keep them pure and cheap, and ensure the CSS class names map to short, stable themes (avoid dynamic class strings from JSON lookups unless memoized).
- **Integration: when adding a theme**, also verify all toolbar buttons (incl. iconography), the pager, and the checkbox column respect the override.
- **High-risk action**: don't ship non-WCAG-compliant colors. Confirm contrast (≥ 4.5:1 for body text) per WCAG 1.4.3.
- **Configuration protection**: do not edit `@syncfusion/react-material-theme/dist/*.css` directly; instead, wrap and override via your own CSS files. (Per the project's `guardrails` skill: "Config Protection".)
- **Detecting deployment gaps**: missing the `@import` line in `App.css` is the most common visual bug — verify the import exists before shipping.