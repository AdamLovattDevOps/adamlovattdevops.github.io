---
layout: post
title: "Hijacking the Steam Deck's On-Screen Keyboard"
date: 2026-08-05
description: "Steam's OSK has no Ctrl, Alt or Esc, and it dies with Steam. I built one that types real keycodes through /dev/uinput instead."
---

**TL;DR:** Steam's on-screen keyboard types letters fine, but it exposes no `Ctrl`, `Alt`, `Super`, `Esc` or F-keys at all, so no shortcut is reachable — and it only exists while Steam is running. I wrote a replacement that registers a virtual input device on `/dev/uinput` and remapped the hardware keyboard button to summon it. Code: [better-handheld-keyboard](https://github.com/AdamLovattDevOps/better-handheld-keyboard).

![Steam's on-screen keyboard next to Better Handheld Keyboard](/assets/images/before-after-keyboard.png)

*Left: [Pi My Life Up](https://pimylifeup.com/steam-deck-desktop-mode-keyboard/). Right: mine, typing into Firefox.*

## The stock keyboard

Desktop Mode on a SteamOS handheld is a full KDE Plasma desktop — Plasma 6 since SteamOS 3.7, on Wayland by default since 3.8.10 (before that, X11 unless you switched it yourself). The only text input you get without plugging in USB is Steam's on-screen keyboard, summoned with `Steam + X`.

![Steam's on-screen keyboard in Desktop Mode, annotated](/assets/images/stock-steam-osk.png)

The gaps aren't subtle:

- **No `Ctrl`.** No `Ctrl+C`, no `Ctrl+D`, no `Ctrl+W`. Letters reach the terminal; the moment you need to interrupt something, you're stuck.
- **No `Alt`, no `Super`.** No menu traversal, no window management, no KDE shortcuts.
- **No `Esc`, no `F1`–`F12`, no `Home`/`End`, no `PgUp`/`PgDn`, no `Del`.** Refresh in Firefox, escape a dialog, jump to the start of a line — all unreachable.
- **Arrow keys exist, barely.** Four half-height keys in the bottom-right corner, next to `Paste` and `Move`.
- **Keys that aren't keys.** `X`, `R2` and `L2` glyphs sit on `Backspace`, `Enter` and `Shift` — it's a controller UI wearing a keyboard's clothes.
- **Opaque, and half the screen.** You can't see the thing you're typing into.
- **It needs Steam.** It's a Steam client window (`steamwebhelper`). No Steam process, no keyboard.

These are documented complaints going back to launch, not my discovery. It's a keyboard designed for entering a password into a login box, sat in front of a full KDE desktop.

## Why a replacement instead of a patch

The keys aren't the interesting part — where the event is born is.

![Two ways to fake a keystroke](/assets/images/keystroke-paths.png)

Steam's OSK is a Steam client window whose input goes out through Steam Input's synthetic-input path, which on Linux is built on X11 (`XTEST`). Two practical consequences, neither of which is fixable from the outside:

1. **Nothing to press.** The layout simply has no modifier keys on it. Whatever the injection path can do, you can't ask it for `Ctrl+C`.
2. **It's Steam's, not the system's.** It lives and dies with the Steam client, and Steam Input's behaviour on Wayland sessions is patchy enough that a shim exists purely to feed Steam the `XTEST` interface it expects — [extest](https://github.com/Supreeeme/extest), which reimplements XTEST by creating uinput devices and is `LD_PRELOAD`ed into Steam. Worth noting where that lands: someone else independently concluded the fix was to stop synthesising X11 events and become a kernel device instead.

Mine goes in a layer lower. [`python-evdev`](https://python-evdev.readthedocs.io/en/latest/) opens [`/dev/uinput`](https://www.kernel.org/doc/html/latest/input/uinput.html) and registers a virtual input device, then writes real keycodes into it. The kernel emits them via evdev, [libinput](https://wayland.freedesktop.org/libinput/doc/latest/) picks them up, the compositor routes them to whatever holds focus. Nothing in userspace can tell it apart from a USB keyboard, so `Ctrl`, `Alt`, `Super`, `F1`–`F12`, `Tab`, `Esc` and the arrows work everywhere — X11 session or Wayland, XWayland client or native — including `Ctrl+C` in Konsole.

Cost of that approach: the process needs write access to `/dev/uinput`, i.e. membership of the `input` group and a udev rule. That's the whole reason the installer asks for a password once and then makes you log out.

## The architecture, and where we cut in

![SteamOS Desktop Mode stack and the three hijack points](/assets/images/kde-hijack-architecture.png)

Three interception points:

**1. The trigger.** On handhelds running [InputPlumber](https://github.com/ShadowBlip/InputPlumber) (the SteamOS gamepad daemon), the hardware keyboard button is a gamepad button mapped to `Guide` + `North` — the chord Steam listens for. I take InputPlumber's current default profile, rewrite that one binding to emit a DBus event (`ui_osk`) instead, and load the modified profile over DBus per [composite device](https://github.com/ShadowBlip/InputPlumber/blob/main/bindings/dbus-xml/org.shadowblip.Input.CompositeDevice.xml):

```bash
busctl --system call org.shadowblip.InputPlumber "$dev" \
    org.shadowblip.Input.CompositeDevice LoadProfilePath s "$PROF"
```

No keystroke is emitted, so nothing else in the system reacts to the button. The keyboard subscribes to that DBus signal and toggles itself. The remap is built by string-replacing the exact block in `/usr/share/inputplumber/profiles/default.yaml`; if the block doesn't match, the script bails rather than guessing.

**2. The compositor.** A [KWin script](https://develop.kde.org/docs/plasma/kwin/) ([API reference](https://develop.kde.org/docs/plasma/kwin/api/)) matches on window properties and sets opacity: `handheld-kbd` (our GTK `app_id`, set via `GLib.set_prgname`) to the configured value, default `0.72`; Steam's OSK to `0.0`.

```javascript
if (c.indexOf("handheld-kbd") !== -1) w.opacity = 0.72;
else if (cap.indexOf("Steam Input On-screen Keyboard") !== -1) w.opacity = 0.0;
```

The script is regenerated from `config.json` at daemon start, and the daemon re-checks every ~3s that it's still loaded — the boot-time load loses the race against KWin startup often enough to matter.

**3. The keys.** [`/dev/uinput`](https://www.kernel.org/doc/html/latest/input/uinput.html), as above, with a 20 ms settle between press and release (`key_settle_ms`) so modifier combinations land in order.

## The ghost window

The default trigger mode doesn't fight Steam, it rides it: the keyboard button still opens Steam's OSK, KWin makes that invisible, and a daemon mirrors its visibility onto ours via `SIGUSR1`/`SIGUSR2` on a 100 ms poll.

Opacity `0.0` is invisible, not gone — a fully transparent window still sits there swallowing every tap in its rectangle. Valve's own bug compounds it: dismissing the OSK can leave an invisible window mapped and on top, blocking input to whatever is underneath ([steam-for-linux#9099](https://github.com/ValveSoftware/steam-for-linux/issues/9099)).

![Opacity 0 is not gone](/assets/images/osk-ghost-window.png)

So the hide key sets a latch, and the daemon has [`xdotool`](https://github.com/jordansissel/xdotool) `windowunmap` Steam's OSK rather than just dimming it. Taps reach the app underneath again. The latch clears once the window is genuinely gone.

Two more failure modes worth naming, because both cost me an evening:

- **Ghost typing.** Two keyboard instances, both listening on the same DBus signal, both injecting — every keypress doubled. Fixed with a single-instance lock.
- **A dead DBus subscription.** The bus connection was being garbage-collected after setup, silently killing the subscription. The connection object is now held in a module-level list.

The daemon also watchdogs the keyboard process: its Wayland connection drops when Steam restarts or the compositor churns, so it's respawned (throttled to once per 3s) instead of needing a reboot.

## The result

![Better Handheld Keyboard over Firefox](/assets/images/handheld-kbd-screenshot.png)

*Bottom row still shows `Home`/`End` — see below.*

Translucent, full key set, US/UK layout switching via a 🌐 key that flips KDE's XKB layout over [`org.kde.KeyboardLayouts`](https://invent.kde.org/plasma/kwin/-/blob/master/src/keyboard_layout.cpp) DBus **and** re-skins the labels, so what's printed and what's typed stay in sync.

The transparency isn't decoration. A Steam Deck is 1280×800; a Legion Go 2 is 1920×1200. A full-width keyboard eats the bottom third either way, and on those panels there are no pixels to spare — so you need to read the terminal output or the form field you're typing into *through* the keys. Opacity defaults to `0.72` and is a number in `config.json`.

Layout, theme, key sizes, opacity and geometry are plain JSON in `~/.config/handheld-kbd/`. Adding a key is a JSON object with a `label` and an evdev `key` name; unknown key names are skipped with a warning rather than taking the keyboard down.

## Next revision

On `master`, not yet in a tagged release:

- **`Home` and `End` dropped from the full layout.** They widened the bottom row for navigation the arrow cluster and `PgUp`/`PgDn` already reach. Bottom row is now `Super`, `Alt`, `Space`, `PgUp`, `PgDn`, `Del`, arrows, locale.
- **Still no predictive-text row, and no plans for one.** Word suggestion is guessing at input that's usually a path, a flag or a hostname.

## Install

Desktop Mode, KDE Plasma 6, Wayland session (default from SteamOS 3.8.10; on 3.7 it's `steamos-session-select plasma-wayland-persistent`). Needs `python3`, [`python-gobject`](https://pygobject.gnome.org/) (GTK 3) and [`python-evdev`](https://python-evdev.readthedocs.io/en/latest/).

```bash
git clone https://github.com/AdamLovattDevOps/better-handheld-keyboard
cd better-handheld-keyboard
./install.sh   # then log out and back in
```

Or double-click `Install Better Handheld Keyboard.desktop`. The log out is not optional — group membership for `input` is read at session start.

There's an experimental [gamescope](https://github.com/ValveSoftware/gamescope)-overlay path for Game Mode in the code, gated behind an env var. Desktop Mode is what I actually use it in.

---

*Tested on a Steam Deck (1280×800) and a Legion Go 2 (1920×1200). The InputPlumber remap is the only device-specific part — it targets the Legion Go's keyboard button — and the default mirror mode works without it.*

Sources: [PCGamesN on the Desktop Mode keyboard](https://www.pcgamesn.com/steam-deck/keyboard-desktop-mode) · [steam-for-linux#9099 (invisible window blocks input)](https://github.com/ValveSoftware/steam-for-linux/issues/9099) · [steam-for-linux#10632 (Steam Input on Wayland)](https://github.com/ValveSoftware/steam-for-linux/issues/10632) · [SteamOS 3.8.10 release notes (Plasma 6.4.3, Wayland default)](https://www.opensourcefeed.org/steamos-3-8-10-release/)
