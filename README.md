# folder-mirror

Keep **any two folders on your Mac** mirrored in real time — using **zero third-party packages** and barely any battery or CPU.

Edit, create, rename, or delete a file in **either** folder (including inside deep subfolders), and the other folder updates automatically in about **0.35 seconds**. If both folders are linked to the same Git repository, commits and pulls stay in sync between them too.

Works for any two local folders — project directories, notes folders, config folders, or tool/agent folders.

---

## Set it up with your AI agent (copy & paste)

Copy the prompt below and paste it into your coding agent. It will clone this repo, ask which two folders you want mirrored, set everything up in the background on your Mac, and run a quick live test to prove it works.

```text
Please set up https://github.com/prshv1/folder-mirror on my Mac to keep two folders mirrored in real time.

Before changing anything, ask me:
1. The exact paths of the two folders I want to mirror (Folder A and Folder B).
2. Whether these two folders are Git repositories that should also sync commits/pulls with each other (optional).
3. Whether I want shortcut environment variables added to my shell profile (optional).

Then set it up step by step:
1. Clone https://github.com/prshv1/folder-mirror.git to ~/.local/share/folder-mirror-repo (or update it if it already exists).
2. Copy `folder-mirror` to `~/.local/bin/folder-mirror` and make it executable (`chmod +x ~/.local/bin/folder-mirror`).
3. Create the folder `~/.local/state/folder-mirror`.
4. Create `~/Library/LaunchAgents/com.user.folder-mirror.plist` from `templates/com.user.folder-mirror.plist.template`:
   - Replace `{{PYTHON3}}` with my actual Python 3 path (`which python3`).
   - Replace `{{HOME}}` with my home folder path (`$HOME`).
   - Replace `{{DIR_A}}` and `{{DIR_B}}` with my two folder paths.
   - Make sure the `PATH` in the plist includes where `python3` and `git` live.
5. Run one initial sync manually so both folders start cleanly matched:
   `MIRROR_DIR_A="<Folder A>" MIRROR_DIR_B="<Folder B>" ~/.local/bin/folder-mirror`
   (Do not wipe my files if one folder already has unique files — the first run will safely merge them.)
6. Start the background watcher with:
   `launchctl bootout gui/$(id -u) ~/Library/LaunchAgents/com.user.folder-mirror.plist 2>/dev/null || true`
   `launchctl bootstrap gui/$(id -u) ~/Library/LaunchAgents/com.user.folder-mirror.plist`
7. If both folders are Git repos sharing the same remote, install `templates/git-hook.sh.template` into `.git/hooks/` as `post-commit`, `post-merge`, `post-checkout`, and `post-rewrite` in both folders:
   - Replace `{{DIR_A}}` and `{{DIR_B}}` with the two folder paths.
   - Replace `{{SIDE}}` with `A` in Folder A and `B` in Folder B.
   - Run `chmod +x` on each hook.
8. Run a quick live test to verify real-time mirroring:
   - Create a temporary file in a subfolder inside Folder A and confirm it appears in Folder B within ~1 second.
   - Edit that file in Folder B and confirm Folder A updates.
   - Delete the temporary folder in Folder A and confirm it disappears from Folder B cleanly.
9. When finished, explain in plain English:
   - That the mirror is running and passed the live test
   - How to check the log (`~/.local/state/folder-mirror/sync.log`)
   - How to pause, restart, or uninstall it anytime.
```

---

## Why it's fast and safe

- **Instant, not on a timer:** One tiny Python process sleeps quietly in the background and wakes up the moment macOS notices a file changed.
- **Tiny footprint:** Uses **~0.01% CPU** while idle (about 0.4 seconds of CPU per hour) and **~12–20 MB of memory** — no constant Python restarts.
- **Protects your files:**
  - Writes files safely so a sudden crash or power cut won't corrupt anything.
  - If you edit the exact same file in both folders at the exact same moment, it saves a backup copy in `~/.local/state/folder-mirror/conflicts/` and pops up a macOS notification so nothing is ever lost.
  - Waits politely if Git is mid-commit, mid-merge, or mid-rebase.
  - Runs a quiet safety double-check every 5 minutes in case your Mac just woke from sleep.

---

## How this project evolved (4 commits)

All 4 commits are kept in this repo so you can see how multi-agent reviews ("the council") turned a simple script into a fast, dependable background tool:

| Commit | Version | Sync Speed | Idle CPU (1 core) | Memory | What changed |
| --- | --- | --- | --- | --- | --- |
| **1** | Basic script | Up to 5 sec | ~1.8% | 15 MB spikes | Simple 196-line Python script running every 5 seconds. Re-read every file on every check. |
| **2** | Council Run 1 | Up to 30 sec | ~0.3% | 28 MB spikes | Added safe file writes, conflict backups, optional Git syncing + hooks, and crash protection. |
| **3** | Council Run 2 | Up to 30 sec | ~0.2% (~9× lighter) | 28 MB spikes | Skips reading unchanged files and skips running Git when idle. Added case-name safety and auto log cleanup. |
| **4** | **Council Run 3 (Current)** | **~0.35 sec (real time)** | **~0.01% (~23× lighter)** | **~16–20 MB steady** | Replaced the 30-second timer with macOS's built-in real-time folder watcher. Instant subfolder mirroring + desktop alerts on conflicts. |

---

## Everyday commands

### Run a one-time sync manually
```bash
~/.local/bin/folder-mirror --dir-a "/path/to/folder-a" --dir-b "/path/to/folder-b"
```

### Test real-time watching in your terminal
```bash
FOLDER_MIRROR_STDIO=1 ~/.local/bin/folder-mirror   --dir-a "/path/to/folder-a"   --dir-b "/path/to/folder-b"   --watch
```

### Check the sync log
```bash
tail -f ~/.local/state/folder-mirror/sync.log
```

### Pause, restart, or uninstall
```bash
# Pause
launchctl bootout gui/$(id -u) ~/Library/LaunchAgents/com.user.folder-mirror.plist

# Restart
launchctl bootstrap gui/$(id -u) ~/Library/LaunchAgents/com.user.folder-mirror.plist

# Uninstall completely
launchctl bootout gui/$(id -u) ~/Library/LaunchAgents/com.user.folder-mirror.plist
rm -f ~/Library/LaunchAgents/com.user.folder-mirror.plist ~/.local/bin/folder-mirror
rm -rf ~/.local/state/folder-mirror
```

---

## Requirements

- **macOS** (uses built-in macOS file watching; falls back to 30-second checks on Linux)
- **Python 3.10+** (standard library only — nothing to `pip install`)
- **Git** (optional — only used if both folders are Git repos)

## License

MIT
