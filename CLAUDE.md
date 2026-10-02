# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**나의 버킷 리스트** (My Bucket List) is a lightweight, framework-free web application for managing life goals. It's built with vanilla HTML5, CSS3, and JavaScript (ES6+), using Tailwind CSS for styling and LocalStorage for data persistence.

**Key Characteristics:**
- No build tools, bundlers, or npm dependencies
- No framework (pure Vanilla JavaScript with Object and Class patterns)
- Direct browser execution: open `index.html` in any modern browser
- Data persists in LocalStorage; no backend required

## Architecture

### Separation of Concerns

The codebase follows a clear **two-module pattern**:

**`js/storage.js`** — Data Layer (Object Pattern)
- Handles all LocalStorage CRUD operations
- Independent of UI; can be used standalone
- Methods: `load()`, `save()`, `addItem()`, `updateItem()`, `deleteItem()`, `toggleComplete()`, `getStats()`, `getFilteredList()`
- Data structure: Array of objects with `id`, `title`, `completed`, `createdAt`, `completedAt`
- Error handling: try-catch wraps LocalStorage access to gracefully handle quota/permission issues

**`js/app.js`** — Presentation Layer (Class Pattern)
- Single class `BucketListApp` manages all UI logic and DOM interactions
- Caches DOM elements in `cacheElements()` for performance
- Event delegation via `bindEvents()` (form submit, filter clicks, modal interactions)
- Renders entire UI via `render()` method; re-renders on every state change
- Modal handling for edit operations

**`index.html`** — Structure & Markup
- Semantic HTML5 structure
- Static UI template; dynamic content inserted via `innerHTML`
- Inline event handlers (`onclick`) in generated HTML (see `createBucketItemHTML()`)
- Tailwind CSS classes for styling; CDN import in `<head>`

**`css/styles.css`** — Custom Styling
- Supplements Tailwind with animations, responsive tweaks, and dark mode
- Animations: slideIn (items), fadeIn (empty state), scaleIn (modal)
- Responsive breakpoints for mobile (<640px), tablet, desktop

### Data Flow

```
User Interaction → App Event Handler → BucketStorage Operation → Render
```

Example: Checkbox click → `handleToggle(id)` → `BucketStorage.toggleComplete(id)` → `render()`

## Development Guide

### Running the App

**Option 1: Direct (Simplest)**
```bash
open index.html
# or drag into browser
```

**Option 2: Live Server (VS Code)**
1. Install Live Server extension
2. Right-click `index.html` → "Open with Live Server"

**Option 3: Python HTTP Server**
```bash
python -m http.server 8000
# Then visit http://localhost:8000
```

### Making Changes

**Adding a new feature:**
1. If it's data-related (storage, filtering), add to `BucketStorage` methods in `js/storage.js`
2. If it's UI-related (new button, modal), update `index.html` and add event handler in `js/app.js`
3. Call `this.render()` from the event handler to update the UI
4. For styling, add to `css/styles.css` (or modify Tailwind classes in HTML)

**Modifying existing features:**
1. Check `js/storage.js` for data logic; `js/app.js` for UI logic
2. Both modules have clear method names and comments
3. Always maintain the separation: Storage shouldn't reference DOM, App shouldn't directly manipulate storage outside BucketStorage methods

**Testing changes:**
- Manually test in browser (no automated tests currently)
- Open DevTools Console to catch JavaScript errors
- Test across mobile, tablet, desktop viewports (Chrome DevTools Device Emulation)
- Check LocalStorage in DevTools (Application → LocalStorage → `bucketList` key)

### Important Patterns

**XSS Prevention:**
- `createBucketItemHTML()` uses `escapeHtml()` to prevent injection attacks via item titles
- Always call `escapeHtml()` when inserting user input into HTML strings

**LocalStorage Error Handling:**
- `BucketStorage.load()` and `save()` wrap access in try-catch
- Gracefully returns empty array if storage fails, preventing app crashes

**State Reconstruction:**
- App state is derived from LocalStorage every render (`BucketStorage.load()`)
- No in-memory state cache; this keeps logic simple but means every render fetches from storage
- For performance optimizations, consider caching the list in memory if the codebase grows

**Inline Event Handlers:**
- Event handlers in generated HTML (e.g., `onclick="app.handleToggle('${id}')"`) require a global `app` variable
- `app` is created and exposed globally in `BucketListApp` constructor
- This pattern works for small apps but would be refactored in larger projects

## File Structure & Responsibilities

```
index.html        Entry point; defines static structure and Tailwind markup
js/
  storage.js      Data persistence layer (LocalStorage)
  app.js          UI logic layer (event handling, rendering)
css/
  styles.css      Custom animations, responsive tweaks, dark mode
README.md         User-facing project documentation
CLAUDE.md         This file
```

## Notes for Future Development

**No npm/Build Tools:**
- No package.json, no dependencies, no build step
- Keep it simple; external libraries should be imported via CDN if needed
- Avoid adding a bundler unless justified (e.g., TypeScript, large library ecosystem)

**Testing:**
- Currently no test framework
- If tests are needed, consider Jest (runs in Node) or Vitest (ES modules friendly)

**Performance:**
- App re-renders entire list on every change (`render()` method)
- For lists under 100 items, this is acceptable
- For larger lists, consider partial updates or virtual scrolling

**Browser Compatibility:**
- Targets modern browsers (Chrome, Firefox, Safari, Edge latest versions)
- LocalStorage is supported in all modern browsers
- No polyfills or IE11 support

**Dark Mode:**
- CSS includes dark mode rules via `@media (prefers-color-scheme: dark)`
- Currently non-interactive (uses system preference); README mentions future toggle feature

## Immediate Next Steps (if extending)

1. Add categories/tags → Update data structure in Storage, add filter toggle in App
2. Add edit dates → Extend item form, add date picker
3. Data export/import → Add methods to BucketStorage to serialize/deserialize JSON
4. Drag-and-drop reordering → Update BucketStorage to track order, add DOM listeners in App
