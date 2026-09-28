# Form Validator Migration

This section explains how to migrate the `Form` component from EJ2 React to React. It provides a detailed comparison of APIs, including props, methods and events.

Component-specific notes for `<Form>`.

> Note: in EJ2 React, validation is typically set up with `new FormValidator('#form1', options)`. In Pure React, the `<Form>` component takes the validation rules as a `rules` prop and exposes `reset`/`validate` via a typed ref.

## Properties

| EJ2 React | Pure React | Notes |
| --- | --- | --- |
| `rules` | `rules` | Same prop name; pass validation rules. |
| _(none)_ | `validateOnChange` | New prop — validate on every keystroke. |

## Methods

| EJ2 React | Pure React |
| --- | --- |
| `destroy()` (on FormValidator) | `useEffect` cleanup |
| `reset()` | `reset()` on the typed ref |
| `validate()` | `validate()` on the typed ref |

## Events

| EJ2 React | Pure React |
| --- | --- |
| `submit` (on `<form>`) | `onSubmit` (on `<Form>`) |