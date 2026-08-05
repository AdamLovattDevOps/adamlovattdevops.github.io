---
layout: post
title: "Hijacking the Steam Deck's On-Screen Keyboard"
date: 2026-08-05
description: "Steam's OSK fakes keystrokes at the X11 layer, so Ctrl never works. I built one that types through /dev/uinput instead."
---

**TL;DR:** Steam's on-screen keyboard has no `Ctrl`, no `Alt`, no `Esc` and no F-keys, and what it does send is synthesised at the X11 layer — so it can't drive a terminal or a native Wayland app. I wrote a replacement that injects real keycodes through `/dev/uinput` and remapped the hardware keyboard button to summon it instead. Code: [better-handheld-keyboard](https://github.com/AdamLovattDevOps/better-handheld-keyboard).

## The stock keyboard

Desktop Mode on a SteamOS handheld is KDE Plasma 6 on Wayland. A real desktop. The only text input you get without plugging in USB is Steam's on-screen keyboard, summoned with `Steam + X`.

![Steam's on-screen keyboard in Desktop Mode, annotated](/assets/images/stock-steam-osk.png)

*Screenshot: [Pi My Life Up](https://pimylifeup.com/steam-deck-desktop-mode-keyboard/), annotations mine.*

The gaps aren't subtle:

- **No `Ctrl`.** No `Ctrl+C`, no `Ctrl+D`, no `Ctrl+W`. A terminal is unusable.
- **No `Alt`, no `Super`.** No menu traversal, no window management, no KDE shortcuts.
- **No `Esc`, no `F1`–`F12`, no `Home`/`End`, no `PgUp`/`PgDn`, no `Del`.** Refresh in Firefox, escape a dialog, jump to the start of a line — all unreachable.
- **Arrow keys exist, barely.** Four half-height keys in the bottom-right corner, next to `Paste` and `Move`.
- **Keys that aren't keys.** `X`, `R2` and `L2` glyphs sit on `Backspace`, `Enter` and `Shift` — it's a controller UI wearing a keyboard's clothes.
- **Opaque, and half the screen.** You can't see the thing you're typing into.

These are documented complaints going back to launch, not my discovery. It's a keyboard designed for entering a password into a login box, sat in front of a full KDE desktop.

## Why a replacement instead of a patch

The keys aren't the interesting part — the injection path is.

![Two ways to fake a keystroke](/assets/images/keystroke-paths.png)

Steam's OSK is an XWayland client called `Steam Input On-screen Keyboard`, and Steam Input fakes its events with the X11 **XTEST** extension. That has two consequences:

1. The event is born inside the X server, so it reaches XWayland clients and nothing else. Native Wayland clients are not in that world.
2. It's a synthetic key press, not a device. Modifier state is whatever the X server can be talked into, not a physical latch — which is why held modifiers are unreliable even where the keys exist.

Mine goes the other way. `python-evdev` opens `/dev/uinput` and registers a virtual input device, then writes real keycodes into it. The kernel emits them via evdev, libinput picks them up, KWin routes them to whatever holds focus. At that point nothing in userspace can tell it apart from a USB keyboard, so `Ctrl`, `Alt`, `Super`, `F1`–`F12`, `Tab`, `Esc` and the arrows all work everywhere — including `Ctrl+C` in Konsole.

Cost of that approach: the process needs write access to `/dev/uinput`, i.e. membership of the `input` group and a udev rule. That's the whole reason the installer asks for a password once and then makes you log out.

## The architecture, and where we cut in

![SteamOS Desktop Mode stack and the three hijack points](/assets/images/kde-hijack-architecture.png)

Three interception points:

**1. The trigger.** On handhelds running InputPlumber (the SteamOS gamepad daemon), the hardware keyboard button is a gamepad button mapped to `Guide` + `North` — the chord Steam listens for. I take InputPlumber's current default profile, rewrite that one binding to emit a DBus event (`ui_osk`) instead, and load the modified profile over DBus per composite device:

```bash
busctl --system call org.shadowblip.InputPlumber "$dev" \
    org.shadowblip.Input.CompositeDevice LoadProfilePath s "$PROF"
```

No keystroke is emitted, so nothing else in the system reacts to the button. The keyboard subscribes to that DBus signal and toggles itself. The remap is built by string-replacing the exact block in `/usr/share/inputplumber/profiles/default.yaml`; if the block doesn't match, the script bails rather than guessing.

**2. The compositor.** A KWin script matches on window properties and sets opacity: `handheld-kbd` (our GTK `app_id`, set via `GLib.set_prgname`) to the configured value, default `0.72`; Steam's OSK to `0.0`.

```javascript
if (c.indexOf("handheld-kbd") !== -1) w.opacity = 0.72;
else if (cap.indexOf("Steam Input On-screen Keyboard") !== -1) w.opacity = 0.0;
```

The script is regenerated from `config.json` at daemon start, and the daemon re-checks every ~3s that it's still loaded — the boot-time load loses the race against KWin startup often enough to matter.

**3. The keys.** `/dev/uinput`, as above, with a 20 ms settle between press and release (`key_settle_ms`) so modifier combinations land in order.

## The ghost window

The default trigger mode doesn't fight Steam, it rides it: the keyboard button still opens Steam's OSK, KWin makes that invisible, and a daemon mirrors its visibility onto ours via `SIGUSR1`/`SIGUSR2` on a 100 ms poll.

Which surfaces a Valve bug ([steam-for-linux#9099](https://github.com/ValveSoftware/steam-for-linux/issues/9099)): the OSK leaves windows mapped after dismissal. Opacity `0.0` is invisible, not gone — it still swallows every tap in its rectangle.

![Opacity 0 is not gone](/assets/images/osk-ghost-window.png)

So the hide key sets a latch, and the daemon `xdotool windowunmap`s Steam's OSK rather than just dimming it. Taps reach the app underneath again. The latch clears once the window is genuinely gone.

Two more failure modes worth naming, because both cost me an evening:

- **Ghost typing.** Two keyboard instances, both listening on the same DBus signal, both injecting — every keypress doubled. Fixed with a single-instance lock.
- **A dead DBus subscription.** The bus connection was being garbage-collected after setup, silently killing the subscription. The connection object is now held in a module-level list.

The daemon also watchdogs the keyboard process: its Wayland connection drops when Steam restarts or the compositor churns, so it's respawned (throttled to once per 3s) instead of needing a reboot.

## The result

![Better Handheld Keyboard over Firefox](/assets/images/handheld-kbd-screenshot.png)

*Screenshot predates dropping `Home`/`End`, which are still visible in the bottom row.*

Translucent, full key set, US/UK layout switching via a 🌐 key that flips KDE's XKB layout over `org.kde.KeyboardLayouts` DBus **and** re-skins the labels, so what's printed and what's typed stay in sync.

Layout, theme, key sizes, opacity and geometry are plain JSON in `~/.config/handheld-kbd/`. Adding a key is a JSON object with a `label` and an evdev `key` name; unknown key names are skipped with a warning rather than taking the keyboard down.

Two things deliberately absent:

- **Predictive text.** It never earned a row. Typing `git rebase --onto` doesn't want help.
- **`Home` and `End`.** They widened the bottom row for navigation the arrow cluster and `PgUp`/`PgDn` already cover. Dropped from the current layout.

## Install

Desktop Mode, KDE Plasma 6 (Wayland). Needs `python3`, `python-gobject` (GTK 3), `python-evdev`.

```bash
git clone https://github.com/AdamLovattDevOps/better-handheld-keyboard
cd better-handheld-keyboard
./install.sh   # then log out and back in
```

Or double-click `Install Better Handheld Keyboard.desktop`. The log out is not optional — group membership for `input` is read at session start.

There's an experimental gamescope-overlay path for Game Mode in the code, gated behind an env var. Desktop Mode is what I actually use it in.

---

*Built against SteamOS Desktop Mode on a KDE Plasma 6 handheld. The InputPlumber remap is the only device-specific part — it targets the Legion Go's keyboard button — and the default mirror mode works without it.*

Sources: [PCGamesN on the Desktop Mode keyboard](https://www.pcgamesn.com/steam-deck/keyboard-desktop-mode) · [steam-for-linux#9099](https://github.com/ValveSoftware/steam-for-linux/issues/9099) · [gamescope#33 on XTEST under XWayland](https://github.com/ValveSoftware/gamescope/issues/33)
