# folder-mirror

Two-way folder mirror for macOS with safe file writes, conflict backups, and optional Git syncing.

### What changed in Council Run 1
- Switched to safe temporary-file swaps so a crash or power cut can't leave half-written files.
- Saves a backup copy if the exact same file is edited in both folders at once.
- Added optional Git syncing + Git hooks so committing or pulling in either folder updates the other one too.
- Changed background check interval from 5s to 30s at low background priority.
