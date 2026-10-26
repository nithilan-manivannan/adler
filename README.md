# Adler

**A lightweight auto clicker for Windows and macOS with millisecond intervals, global hotkeys, and pause that keeps its place.**

Adler is a single file Python app that clicks the mouse for you. Type an interval, pick a mouse button, press a key, and Adler clicks at that rate until you tell it to stop. No config files, no background services, no installer required if you run it from source with Python.

## Why it exists

Most auto clickers are bloated or imprecise. Adler stays tiny and gets the details right: the interval is built from four fields (hours, minutes, seconds, milliseconds), pausing does not eat into your schedule, and the controls work even when the window is not focused.

## Features

- **Millisecond precision**: the click interval is composed of separate hrs, min, sec, and ms fields. 15 seconds 50 milliseconds is a valid interval. An invalid or empty interval falls back to 50 ms.
- **Global hotkeys**: press the `-` key to start, pause, or resume, and the `=` key to stop. Both work system wide through pynput, even while another window has focus.
- **Pause without penalty**: pausing freezes the loop and slides the countdown target forward, so paused time never counts against the schedule. The stop key immediately ends the loop.
- **Repeat limit**: leave the repeat field at 0 for unlimited clicks, or set a count and Adler stops itself with a "limit reached" notice.
- **Start delay**: an optional one to ten second countdown before the first click so you can move the cursor into place.
- **Mouse button choice**: left, right, or middle click, chosen from a dropdown.
- **Live status**: the status line shows the countdown, Running, Paused, or the stop reason, with a running click count underneath.
- **Bundled interface**: dark card layout with Courier typography, drawn entirely in Tkinter. No external assets or themes at runtime.

## At a glance

| Key | Action |
|---|---|
| `-` | Start, pause, or resume clicking |
| `=` | Stop (global, works anytime) |

| Field | Meaning |
|---|---|
| hrs, min, sec, ms | Build up the full interval between clicks |
| Mouse button | Left, Right, or Middle |
| Repeat | Click limit, 0 means unlimited |
| Start delay | Countdown in seconds before clicking begins |

## Requirements

- Python 3.10 or newer
- [pynput](https://pypi.org/project/pynput/)

```bash
pip install pynput
python adler.py
```

On macOS, pynput needs Accessibility permission the first time the app runs. On Windows, some games ignore synthetic clicks unless the app runs as administrator.

## Building executables

Tag a release such as `v1.0` and GitHub Actions takes over. The workflow in `.github/workflows/build.yml`:

- Builds `Adler.exe` with PyInstaller (onefile, windowed, using `adler.ico`) on Windows.
- Generates an `.icns` from `adler_logo.png` with `sips` and `iconutil`, then builds and zips `Adler.app` on macOS.
- Uploads both artifacts and attaches them to the GitHub release.

No code changes are needed to publish a new build beyond tagging the newest commit.

## Repository structure

```
adler/
├── adler.py                        # the whole app in one file
├── adler.ico                       # Windows icon
├── adler_logo.png                  # macOS icon source art
├── index.html                      # project landing page
└── .github/
    └── workflows/
        └── build.yml               # release builds on tag push
```

## Design notes

- The click loop runs in a daemon thread and talks to the UI only through `root.after`, which keeps Tkinter thread safe.
- `pausable_sleep` advances time in 50 ms steps and checks the shared state each step, which is what makes pause immediate and exact rather than approximate.
- State is tracked as three simple constants: idle, running, and paused. Toggling hotkeys transitions between them; the stop key always returns to idle and resets the count.
