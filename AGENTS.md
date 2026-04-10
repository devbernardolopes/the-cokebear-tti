# The Cokebear TTI - Agent Instructions

## Project Overview

This is a **Text-to-Image Generator UI** for [Perchance.org](https://perchance.org). It provides a web interface for generating AI images with features like:

- Variable-based prompt system via scratchpad
- Concurrency-controlled parallel generation
- IndexedDB persistence (via Dexie) for settings and prompt history
- Image modal with navigation
- Keyboard shortcuts
- Tab keep-alive mechanism (prevents throttling during generation)

**The ONLY file that matters is `index.html`** — this is a single-file application deployed to Perchance.

## Key Dependencies

- **Dexie.js 3.2.4** (CDN: `https://unpkg.com/dexie@3.2.4/dist/dexie.min.js`)
- **Local CSS** (`style.css` - styles are in a separate local file)

## Architecture

### Fetch Interception

The app intercepts `window.fetch` to:

1. Capture generate requests and store resolved prompts for tracking
2. Monitor queue position during generation

### Database Schema (IndexedDB via Dexie)

```javascript
db.version(1).stores({ prompts: "++id, prompt, timestamp" });
db.version(2).stores({
  prompts: "++id, prompt, timestamp",
  settings: "key, value",
});
```

### Prompt Variable System

Scratchpad variables are defined as `VARIABLE_NAME = {value1|value2|value3}` and inserted into prompts as `[VARIABLE_NAME]`.

### Style Constants

Prompt suffixes for different image styles (cinematic, professional, casual, etc.) are defined as constants and appended during generation.

## Commands

### Testing

No test framework is currently set up. Manual testing in browser is required.

### Linting/Typechecking

No linter or type-checker is configured.

### Development

This is a single HTML file — no build step required. Edit `index.html` directly and test in a browser.

## Keyboard Shortcuts

| Shortcut       | Action                                |
| -------------- | ------------------------------------- |
| `Ctrl+Shift+S` | Toggle Scratchpad                     |
| `Ctrl+Shift+H` | Toggle History                        |
| `Ctrl+Shift+E` | Auto-adjust prompt height             |
| `Ctrl+Shift+Q` | Cancel generation                     |
| `Ctrl+Enter`   | Generate (when not focused in prompt) |
| `Enter`        | Generate (when "ENTER to send" is on) |
| `Escape`       | Close modal                           |

## Important Notes

1. **Perchance Integration**: The `image()` function is provided by Perchance's plugin system. The app polls for it via `initPlugin()`.

2. **Variable Names**: Use uppercase names in scratchpad (e.g., `HAIR_COLOR = {black|brown}`) and insert as `[HAIR_COLOR]`.

3. **Generation Concurrency**: Controlled by the concurrency dropdown (1-4 parallel requests).

4. **Tab Keep-Alive**: Uses silent AudioContext + setInterval to prevent browser throttling during long generations.

## Code Conventions

- Vanilla JavaScript, no build tools
- DOM elements cached in `el` object
- Global functions exposed via `window.*`
- Async/await for database operations
- Dexie.js for IndexedDB abstraction

## Git Rules

- Never run `git commit`, `git push`, `git add`, or any other git commands that modify the repository.
- After completing a task where any file was changed, always suggest a single-line, concise commit message in plain-text in its own separate paragraph.
