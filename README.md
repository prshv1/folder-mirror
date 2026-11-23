# folder-mirror

Fast, safe two-way folder mirror for macOS.

### What changed in Council Run 2
- **~9x less CPU when idle:** skips reading unchanged files and skips running Git when nothing changed.
- Handles uppercase/lowercase filename conflicts cleanly on macOS.
- Protects against half-finished Git operations and automatically trims log files.
