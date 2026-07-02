# Better xCloud (webOS) — LG TV Mod

**Base:** [Better xCloud](https://github.com/redphx/better-xcloud) v6.2.1 "Lite" build for webOS, by [redphx](https://github.com/redphx) (MIT License)
**What this is:** an unofficial patch on top of the official webOS `.ipk` release, built to fix/extend a few things specifically for older/lower-end LG TVs (tested on an LG 32LR600BPSA, HD-Ready webOS TV).

This is **not** a fork of the source repo — it's a binary patch applied directly to the compiled `userScript.js` inside the `.ipk`, since no webOS-specific source build was available. All additions are plain injected JavaScript/CSS, MIT-compatible, no obfuscation.

---

## Why this exists

Better xCloud's webOS build works great for streaming, but a few things don't work out of the box on LG TVs that aren't officially supported by the Xbox app:

- No on-screen keyboard — text fields (search, sign-in, in-game chat/usernames) can't be typed into with a TV remote or a game controller.
- No controller-based navigation for any of the mod's own UI.
- No overscan/safe-zone handling — older LG panels can crop the edges of the interface.
- No easy way to find the fastest Xbox Cloud Gaming region for your location.
- Changing `stream.video.resolution` to 1080p/1080p-HQ can produce a black screen on lower-end hardware, with no quick way back if you get stuck.

## What was added

### 1. On-screen keyboard
- Automatically opens when any real `<input>`/`<textarea>`/`contenteditable` field gets focus (login, search, in-game text boxes rendered as real DOM elements).
- Works two ways:
  - **Pointer (LG Magic Remote):** point and click keys normally.
  - **D-pad remote:** arrow keys move the selection, OK/Enter presses the key, Back closes the keyboard.
- Shows a live text preview with cursor position, has Shift, Backspace, Space, and a floating ⌨ button to force-open it if a field doesn't trigger focus automatically.
- Writes into React-controlled inputs correctly (uses the native input-value setter + dispatches a proper `input` event, not a plain `.value =` assignment).

### 2. Xbox controller support for the on-screen keyboard
- D-pad **or** left stick navigates the keyboard grid (with initial-delay + repeat while held).
- **A** selects a key, **B** closes the keyboard.
- While the keyboard is open, `navigator.getGamepads()` is temporarily overridden to report no controllers connected — so the game underneath doesn't receive any input while you're typing (same technique Better xCloud itself uses for its mouse/keyboard-emulation mode). The original function reference is restored exactly as it was when the keyboard closes, so it doesn't break other controller-related features.

### 3. TV safe-zone / overscan fix
- A small panel (📺 button) with `−` / `+` controls from 0–10%.
- Applies a `transform: scale()` to the page, shrinking it slightly with a black margin so a TV that crops the outer edge of the picture doesn't cut off UI elements.
- Value is saved and re-applied automatically on every launch.

### 4. Automatic "fastest region" selector
- 🌐 button opens a panel with a "Test regions" action.
- Pings every Xbox Cloud Gaming region returned for your account (reads `STATES.serverRegions`, populated after opening the game library once) and measures response time for each with `fetch(..., {mode: "no-cors"})`, timing the round trip.
- Sorts results by latency, auto-applies the fastest one as your preferred region (`server.region` pref), and lets you manually pick a different one from the list if you want.

### 5. Quick resolution switcher with one-tap/blind revert
- 🖥️ button opens a panel to switch between `auto` / `720p` / `1080p` / `1080p-HQ`.
- Every change is remembered, with a "↩ Revert to X" button always available.
- **Blind revert:** hold the TV remote's Back button *or* the controller's B button for 1.5s at any time (even during gameplay, even with a black screen) to instantly revert to the previous resolution — useful because on some lower-end webOS TVs, forcing 1080p changes the spoofed device profile sent to the server (Windows/Tizen instead of Android) to a decode profile the hardware can't handle, resulting in a black screen with audio only.

---

## Known limitations

- Resolution/keyboard/controller changes only take effect on the **next** stream session — you need to back out to the dashboard and relaunch the game after changing resolution.
- The on-screen keyboard only works on real HTML input elements. If a game renders its own text-entry UI inside the video stream itself (rare, but some titles do), there's no way to hook into it — it's just pixels.
- The region ping test needs `STATES.serverRegions` to be populated first, which only happens after the app has loaded your game library at least once in the current session.
- Tested only on an LG 32LR600BPSA (webOS, HD-Ready). Should work on any webOS TV running this same Better xCloud Lite build, but layout/remote key codes were tuned for LG's remote (`Back` = keyCode 461).

## Installation

1. Install the [Homebrew Channel](https://www.webosbrew.org/) on your LG TV if you haven't already.
2. Download the `.ipk` and make sure it actually has the `.ipk` extension (some browsers/apps rename downloads to `.txt` — just rename it back).
3. Uninstall any previous version of Better xCloud on the TV first if you're updating from a differently-modified build.
4. In Homebrew Channel → Install → pick the file.

## Credits

- [redphx/better-xcloud](https://github.com/redphx/better-xcloud) — all core streaming/UI functionality, MIT License.
- This patch only adds the features above on top of the existing MIT-licensed code; no original functionality was removed or altered beyond what's described here.
