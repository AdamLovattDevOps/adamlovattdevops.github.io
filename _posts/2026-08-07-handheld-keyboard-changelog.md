---
layout: post
title: "Better Handheld Keyboard — release history"
date: 2026-08-07
description: "Every release of the SteamOS on-screen keyboard, what broke, and what the fix actually was. Updated with each tag."
---

**A living page.** [Better Handheld Keyboard](https://github.com/AdamLovattDevOps/better-handheld-keyboard) replaces the on-screen keyboard on SteamOS handhelds in Desktop Mode — the [build write-up is here](/steam-deck-keyboard/). This is the release history, updated with each tag.

More of it is bug fixes than features, and the bugs are the interesting part. Nearly every one arrived the same way: something worked on the two devices I own and failed on hardware I'd never touched, or worked on a fresh install and broke on an upgrade.

**Latest: [v1.0.11](https://github.com/AdamLovattDevOps/better-handheld-keyboard/releases/tag/v1.0.11)** · install with `git clone -b v1.0.11` or grab the [latest release](https://github.com/AdamLovattDevOps/better-handheld-keyboard/releases/latest).

---

## v1.0.11 — Twenty languages
*7 August 2026*

Key labels for twenty XKB layouts, an **AltGr** key to reach the third level, a tray-menu language picker, and a way to know any of it is correct.

The labels are the whole feature, and the awkward part. The keyboard injects real keycodes, so what a key *types* was always the OS layout's decision — selecting Russian typed Cyrillic correctly while the keyboard drew `q` on the key that produces `й`. The gap was never in typing; it was the keyboard refusing to admit what the OS was doing.

I can proofread Latin and squint at Cyrillic and Greek. I cannot proofread Arabic, Hebrew, Devanagari or Thai — and "looks plausible" in a script you can't read is worth nothing. So nothing is hand-written: labels are generated from [xkeyboard-config](https://gitlab.freedesktop.org/xkeyboard-config/xkeyboard-config) and `keysymdef.h`, then checked against **libxkbcommon** (940 key/level pairs, zero mismatches, re-run in CI), then each language is **typed into Kate** on real hardware and the saved file compared character by character.

Writing that parser produced three bugs, each of which shipped a layout that looked fine and was wrong:

- **Braces inside comments** — `key <AB03> {[...]};  // ؤ }` ended the Arabic block after three keys, leaving `v b n m , . /` in Latin on an Arabic keyboard.
- **Deprecated keysym aliases** carry no `U+` comment, so `masculine` and `guillemotleft` resolved to nothing — and a key whose *first* level fails is dropped entirely, so Spanish lost its `º` key rather than a label.
- **Type declarations** — `type[group1] = "…"` was read as a symbol list, losing Turkish's dotted and dotless `i`.

Two things only the typing test could find:

- **`Ctrl+S` dies under a non-Latin-only keymap.** Application shortcuts bind to Latin keysyms, so the S key produces `Cyrillic_yeru` and Save never fires. Keep one Latin layout among your four; the picker warns.
- **The rouble was on a level nothing could reach.** Russian defines `₽` on level 3 and binds no AltGr switch. The label matched the keymap perfectly — the keymap was making the unkeepable promise. Level-3 labels now only appear for layouts with a switch.

Also: **four layouts live at once**, which is the keymap format rather than a setting — libxkbcommon discards a fifth outright. Twenty sets of labels install; the tray picker chooses which four.

Fixed along the way: a long label could resize the whole keyboard (the grid is column-homogeneous, so `🌐LATAM` pushed the window to 1728px on a 1280px desktop, and GTK won't shrink below natural width — so reset appeared broken); the 🌐 badge named the country rather than the language (`il` is Israel, not Hebrew); the logout prompt compared layout *counts*, so swapping Russian for Vietnamese looked like nothing had happened; upgrading asked for a password after the keyboard had gone; and `uninstall.sh` left eleven of eighteen binaries behind.

## v1.0.10 — The Steam Deck keeps its trackpads
*6 August 2026*

On a Deck the keyboard is summoned by Steam's own OSK appearing, and we hid Steam's by setting its opacity to zero. The window stayed mapped, so Steam went on believing its keyboard was open — and while it believes that, it forces the controller into its `KB ActionSet`, where the sticks and trackpads navigate *that* keyboard instead of moving the pointer.

```
Set OSK active 1 and appid 413080
OnFocusWindowChanged On Screen Keyboard Forcing to window type: KB ActionSet
```

A keyboard you could only use by touch, and no pointer at all until Steam restarted. Steam's keyboard is now closed rather than hidden. Never affected a Legion Go, where the hardware button goes through InputPlumber and Steam's keyboard is never involved — which is why it looked device-specific for so long.

Added a system tray icon and `handheld-kbd-ctl` for the recovery actions, because typing a command is the one thing you can't do when the keyboard is the problem.

## v1.0.9 — The tick actually holds the position
*6 August 2026*

Finishing a move asked KWin where the window was and waited a fixed 250 ms. A real drag answers slower than that, and giving up handed placement back to the docking script — so the keyboard snapped home. It now waits for the answer, and a move that can't be read back falls into a `free` mode where nothing places the window at all, rather than defaulting to the dock.

## v1.0.8 — Event-driven, free movement, prediction out of the box
*6 August 2026*

The daemon polled the X window tree ten times a second to notice Steam's keyboard appearing. That was both the latency in summoning and a constant drip of processes competing with Steam Input. The KWin script now calls us over DBus the instant the window maps: 27.2s of CPU per session became 1.4s, and show/hide went to 3–10 ms.

Free movement replaced the lock key, one thing places the keyboard instead of two fighting, and `handheld-kbd-toggle` summons it without Steam — so a dead Steam client can't leave you with no keyboard.

## v1.0.7 — Correct at any resolution and any scale
*6 August 2026*

v1.0.6 docked using the display's *physical* pixels; KWin positions windows in *logical* ones, and they only agree at scale 1. A Legion Go 2 is 1920×1200 at scale 1.5 — a 1280×800 desktop — so a 1920-wide rect went mostly off the side. Geometry now comes from GDK, and docking is computed by the KWin script from `clientArea(MaximizeArea)`, the only source that knows the usable area with panels excluded.

Keys are also sized to the dock height *before* placement: they used to keep their natural height, pushing the window's minimum past the dock and coming back 100px taller with its bottom off-screen.

## v1.0.6 — Same place on every device
*6 August 2026*

The default position stopped being a pixel rect that happened to suit a 1280×800 Deck. It's computed from the panel — full width, bottom-flush, height a fraction of the display — so a Deck LCD, a Deck OLED and a Legion Go 2 get the same keyboard in the same place, which is how Steam's own OSK behaves.

## v1.0.5 — Upgrades that actually upgrade
*5 August 2026*

Two bugs of the same shape: files only ever written on a *fresh* install, leaving upgrades in a state the rest of the code didn't expect.

Opacity cycling silently stopped working, because the KWin script had two writers — the installer's static copy hardcoded `0.72` with no `var OP` line for the opacity key to patch. And new keys never reached existing installs: layouts are only written when absent, so a release could ship the code for a key while your layout had no button for it. The installer now merges in missing action keys.

## v1.0.4 — Unlock, drag, lock
*5 August 2026*

v1.0.3's move key cycled preset docking slots. It didn't work well, so: unlock, put it where you want, lock. Wayland clients can't place their own windows, so both gestures hand off to the compositor — the same mechanism a titlebar uses.

## v1.0.3 — A move key
*5 August 2026*

Superseded by v1.0.4 the same evening. Its lasting contribution: every rect is clamped to the display it lands on, so a saved position can't put the keyboard off-screen — the shipped geometry assumed 1280×800 and hung 78px off the bottom of a 720p panel.

## v1.0.2 — Predictive text and a bigger keyboard
*5 August 2026*

The features that had been living on my own Legion Go 2: predictive text that learns locally and never leaves the device, big mode, opacity cycling, swipe typing, gesture and focus triggers, and resume handling for a stale trigger after sleep.

## v1.0.1 — Legion Go 1 keyboard recovery
*5 August 2026*

**v1.0.0 could leave a Legion Go 1 with no on-screen keyboard at all** — not mine, not Steam's. Same for a Deck or ROG Ally under Bazzite or ChimeraOS.

The installer picked its trigger by grepping InputPlumber's default profile for `button: Keyboard`. That profile is generic — it carries the mapping on every device it supports — so the grep answered "is InputPlumber installed", not "does this handheld have a keyboard button". Seamless mode got selected on hardware that can't drive it. Then the second mistake turned a dead trigger into a dead desktop: the KWin script still forced Steam's keyboard to `opacity = 0.0`, so the fallback was suppressed by the thing that had already failed.

Detection is by DMI product name now, and Steam's keyboard is only hidden once ours is *known* to appear. A broken trigger degrades to the stock keyboard, which is where the user started. Thanks to the Legion Go 1 user who reported it and confirmed the fix.

The lesson I'd keep: a capability advertised in a config file, or even over DBus, is a claim about software, not evidence about hardware. And when you suppress someone's fallback, make the suppression conditional on your replacement actually working.

## v1.0.0 — First release
*5 August 2026*

Keys injected through `/dev/uinput` as a virtual input device, so `Ctrl`, `Alt`, `Super`, `F1`–`F12`, `Tab`, `Esc` and the arrows reach any focused application — including `Ctrl+C` in a terminal. Triggered by the hardware keyboard button, adjustable transparency via a KWin script, full and compact layouts, everything configurable in JSON.

---

*Full technical detail for every release is in [CHANGELOG.md](https://github.com/AdamLovattDevOps/better-handheld-keyboard/blob/master/CHANGELOG.md). Bugs and layout corrections: [issues](https://github.com/AdamLovattDevOps/better-handheld-keyboard/issues).*
