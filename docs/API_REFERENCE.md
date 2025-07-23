# ETS Tools – API & Component Reference

This document provides detailed documentation for the JavaScript APIs exposed by **ETS Tools** (the `ETS Tools-2.html` application).  All functions are attached to the global `window` object, so they can be invoked directly from the browser console or from other scripts loaded **after** the page.

---

## Table of Contents
1. Theme Management
2. Navigation
3. Notification System
4. EDI Formatter
5. EDI Comparator
6. IDoc Utility
7. URL Encoder / Decoder
8. Generic Utilities
9. Team Contacts Component

---

## 1  Theme Management

### `initTheme()`
Initialises the page theme based on the user’s previously-saved preference in `localStorage`.

| Parameter | Type | Description |
|-----------|------|-------------|
| — | — | — |

Returns `void`.

**Example**
```js
// Re-initialise the theme – useful after manually changing localStorage.
initTheme();
```

---

### `toggleTheme()`
Toggles between **light** and **dark** mode, persists the choice to `localStorage`, and updates the theme icon.

| Parameter | Type | Description |
|-----------|------|-------------|
| — | — | — |

Returns `void`.

**Example**
```js
// Bind to a custom keyboard shortcut, e.g. Cmd/Ctrl + J.
document.addEventListener('keydown', e => {
  if (e.metaKey && e.key === 'j') toggleTheme();
});
```

---

### `updateThemeIcon(isDark)`
Internal helper used by `initTheme()` and `toggleTheme()` to show / hide the corresponding Lucide icon.

| Parameter | Type | Description |
|-----------|------|-------------|
| `isDark` | `boolean` | `true` → dark mode enabled; `false` → light mode. |

Returns `void`.

> Although intended for internal use, you can call it directly if you need to sync the icons after DOM manipulation.

---

## 2  Navigation

### `scrollToSection(sectionId)`
Smooth-scrolls the viewport so that the element with `id=sectionId` becomes visible at the top.

| Parameter | Type | Description |
|-----------|------|-------------|
| `sectionId` | `string` | Target element’s `id` attribute. |

Returns `void`.

**Example**
```js
// Jump to the “EDI Comparator” section programmatically.
scrollToSection('edi-comparator');
```

---

## 3  Notification System

### `showNotification(message, isError = false)`
Displays a transient toast-like banner in the bottom-right corner.

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `message` | `string` | — | Text displayed to the user. |
| `isError` | `boolean` | `false` | Render as an error (red banner) instead of success (green banner). |

Returns `void`.

### `hideNotification()`
Immediately hides the notification banner. Called automatically by `showNotification()` after 3 seconds.

| Parameter | Type | Description |
|-----------|------|-------------|
| — | — | — |

Returns `void`.

**Example**
```js
showNotification('Everything saved successfully!');
// …later, cancel early if needed
hideNotification();
```

---

## 4  EDI Formatter

### `handleFileUpload(event)`
Reads the first file selected in an `<input type="file">` element and injects its contents into the **EDI Input** textarea `#ediContent`.

| Parameter | Type | Description |
|-----------|------|-------------|
| `event` | `Event` | Standard DOM `change` event from a file input. |

Returns `void`.

### `convertEDI()`
Re-formats the raw EDI text in `#ediContent` by replacing the `~` delimiter with newlines, placing the result in `#ediOutput`.

Returns `Promise<void>` (resolves after formatting completes).

**Example (headless)**
```js
// Provide sample payload
document.getElementById('ediContent').value = 'ISA*00*          ~…';
await convertEDI();
console.log(document.getElementById('ediOutput').value);
```

---

## 5  EDI Comparator

### `compareFiles()`
Highlights character-level differences between the two multi-line strings held in `#compareFile1` and `#compareFile2`.  The results are injected as HTML into `#differences` and the results panel is shown.

Returns `Promise<void>`.

### `compareLine(line1, line2)` *(helper)*
Produces HTML highlighting differences between two strings.

| Parameter | Type | Description |
|-----------|------|-------------|
| `line1` | `string` | First line. |
| `line2` | `string` | Second line. |

Returns `string` (HTML).

### `escapeHTML(text)` *(helper)*
Escapes HTML meta-characters to prevent XSS in diff output.

| Parameter | Type | Description |
|-----------|------|-------------|
| `text` | `string` | Raw text. |

Returns `string` (escaped).

---

## 6  IDoc Utility

### `handleIdocSubmit(event)`
Intercepts the **Re-process IDoc** form submission, opens the downstream webMethods endpoint in a new tab, and passes the entered IDoc numbers in a hidden POST field.

| Parameter | Type | Description |
|-----------|------|-------------|
| `event` | `Event` | Form `submit` event. Call `preventDefault()` internally. |

Returns `void`.

---

## 7  URL Encoder / Decoder

### `encode()`
Encodes the value in the `#dencoder` textarea using `encodeURIComponent`, making it safe for inclusion in URLs.

Returns `void`.

### `decode()`
Decodes the current `#dencoder` value using `decodeURIComponent`.

Returns `void`.

---

## 8  Generic Utilities

### `copyToClipboard(elementId)`
Copies the `.value` of the element with the supplied `id` to the system clipboard.

| Parameter | Type | Description |
|-----------|------|-------------|
| `elementId` | `string` | The element’s `id` attribute. |

Returns `void`.

### `downloadFile(elementId, filename)`
Creates a text file from the `.value` of `#elementId` and triggers a download.

| Parameter | Type | Description |
|-----------|------|-------------|
| `elementId` | `string` | Source element id. |
| `filename` | `string` | Download filename (e.g. `output.txt`). |

Returns `void`.

---

## 9  Team Contacts Component
The **Teams** section lets users jump directly to pre-defined Slack channels.  It is powered by four public functions:

### `renderTeams(filteredTeams = teams)`
Renders a (sub-set of) the `teams` array as a list of buttons.

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `filteredTeams` | `Array<Team>` | `teams` | Optional subset to render. |

Returns `void`.

*`Team` type*
```ts
interface Team {
  name: string;
  url: string;     // Slack deep-link
  icon: string;    // Lucide icon name
  color: string;   // Tailwind bg-color class
}
```

### `filterTeams()`
Reads the search box `#teamSearch`, filters the global `teams` list by name, and re-renders.

Returns `void`.

### `openTeamChannel(url, teamName)`
Opens the given Slack `url` in a new tab and shows a notification that the redirect is occurring.

| Parameter | Type | Description |
|-----------|------|-------------|
| `url` | `string` | Fully-qualified Slack URL. |
| `teamName` | `string` | Display name used in the success message. |

Returns `void`.

---

## Usage from External Scripts
Because all functions live in the global scope, you can import **ETS Tools** into another page via an `<iframe>` or `<script src="ETS Tools-2.html">` and call any API documented above.  For a more modular approach, consider extracting the script portion into its own `ets-tools.js` module and importing it using standard ES modules.

---

## Contributing to the Docs
If you extend or modify any functions, please update this file accordingly and bump the section numbers where needed.