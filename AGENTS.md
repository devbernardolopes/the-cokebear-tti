# The Cokebear TTI - Agent Instructions

## Project Overview

This is a **Text-to-Image Generator UI** for [Perchance.org](https://perchance.org). It provides a web interface for generating AI images with features like:

- Variable-based prompt system via scratchpad
- Concurrency-controlled parallel generation (1-6 parallel requests)
- IndexedDB persistence (via Dexie) for settings and prompt history
- Image modal with navigation and keyboard shortcuts
- Tab keep-alive mechanism (prevents browser throttling during long generations)

**The ONLY file that matters is `index.html`** — this is a single-file application deployed directly to Perchance with no build step.

## Key Dependencies

- **Dexie.js 3.2.4** (CDN: `https://unpkg.com/dexie@3.2.4/dist/dexie.min.js`) — IndexedDB abstraction
- **Perchance Plugin System** — provides the `image()` function for AI image generation; polled via `initPlugin()`
- **Local CSS** — all styles are contained in a `<style>` block within `index.html`

## Architecture

### Fetch Interception

The app intercepts `window.fetch` to:

1. Capture generate requests and store resolved prompts for history tracking
2. Monitor queue position during parallel generation
3. Handle generation cancellation via `Ctrl+Shift+Q`

### Database Schema (IndexedDB via Dexie)

```javascript
db.version(1).stores({ prompts: "++id, prompt, timestamp" });
db.version(2).stores({
  prompts: "++id, prompt, timestamp",
  settings: "key, value",
});
```

- `prompts` table stores generation history with auto-incrementing IDs
- `settings` table stores user preferences (e.g., concurrency, style suffix)

### Prompt Variable System

Scratchpad variables are defined as `VARIABLE_NAME = {value1|value2|value3}` (uppercase names) and inserted into prompts as `[VARIABLE_NAME]`. The app resolves variables by replacing placeholders with random values from the defined set before generation.

### Style Constants

Prompt suffixes for different image styles (cinematic, professional, casual, etc.) are defined as uppercase constants and appended to prompts during generation.

### Tab Keep-Alive

Uses a silent `AudioContext` + `setInterval` to prevent browser tab throttling during long-running generations.

## Commands

### Build

No build step required. This is a single-file application (`index.html`) deployed directly to Perchance. No compilation, bundling, or minification is needed.

### Testing

No automated test framework is set up. All testing is manual via browser by user.

### Development

Edit `index.html` directly. No hot-reload or dev server is required; refresh the browser to see changes. For quick iteration, use the browser's DevTools to debug JS/CSS directly.

## Important Notes

1. **Perchance Integration**: The `image()` function is provided by Perchance's plugin system. The app polls for it every 500ms via `initPlugin()` until available.
2. **Variable Names**: Use uppercase names in scratchpad (e.g., `HAIR_COLOR = {black|brown}`) and insert as `[HAIR_COLOR]`. Variable names are case-sensitive.
3. **Generation Concurrency**: Controlled by the concurrency dropdown (1-4 parallel requests). Exceeding 4 parallel requests may trigger Perchance rate limits.
4. **Tab Keep-Alive**: Uses silent AudioContext + setInterval to prevent browser throttling during long generations. Do not modify this logic unless fixing a throttling issue.

## Code Style Guidelines

### Imports & Dependencies
- Only external dependency is Dexie.js 3.2.4, loaded via CDN script tag in `index.html`
- No local imports or ES modules; all code is contained within `index.html`
- Do not add additional external dependencies unless critical; Perchance's plugin system provides core generation functionality

### Formatting
- Indent all code (HTML, CSS, JS) with 2 spaces; no tabs
- Use single quotes for string literals (consistent with existing Dexie CDN URL)
- Omit semicolons unless required by syntax (e.g., before immediately invoked function expressions)
- Keep line lengths under 100 characters where possible for readability

### Naming Conventions
- DOM elements cached in the `el` object with camelCase names (e.g., `el.promptInput`, `el.historyList`)
- Global functions exposed via `window` use camelCase (e.g., `window.initPlugin`, `window.generateImage`)
- Scratchpad variables use UPPERCASE names as per project conventions
- Constants (e.g., style suffixes) use UPPER_SNAKE_CASE
- Avoid abbreviations in variable/function names unless widely recognized (e.g., `el` for DOM element cache is acceptable)

### Error Handling
- Use try/catch blocks for all async operations (Dexie database calls, fetch interception)
- Log errors to the browser console with descriptive messages; avoid suppressing errors
- For Perchance plugin initialization, use polling with 500ms intervals until `image()` is available
- Don't expose user-facing error messages unless critical (e.g., generation failure)

### Types
- No TypeScript or type annotations; this is a Vanilla JS project
- Use JSDoc comments for complex functions if adding new functionality (optional, but recommended for clarity)

## External Tool Rules
No Cursor rules (`.cursorrules`, `.cursor/rules/`) or GitHub Copilot instructions (`.github/copilot-instructions.md`) are present in this repository.

## Git Rules
- Always commit changes after completing a task that modifies `index.html`
- Create concise, imperative commit messages (e.g., "Add scratchpad variable validation" not "Added scratchpad variable validation")
- Never commit secrets, API keys, or sensitive data (verify `index.html` for sensitive content before committing)
- Do not force-push to the remote repository unless explicitly requested by the user
- If amending a commit, ensure it has not been pushed to the remote
- Follow the existing commit style: short, descriptive messages focused on the "why" of the change
