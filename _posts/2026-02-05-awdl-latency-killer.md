---
layout: post
title: "AWDL: The Silent Latency Killer on macOS"
date: 2026-02-05
description: "How I fixed 90ms latency spikes on my Mac by disabling a single feature I never use"
---

**TL;DR:** If you're experiencing random lag spikes on your Mac during game streaming, video calls, or using tools like Synergy, disable AWDL (AirDrop's wireless protocol). It fixed my 90ms spikes instantly.

## The Problem

I was getting frustrating mouse stickiness when using [Synergy by Symless](https://symless.com/synergy) to share my keyboard and mouse across machines. The cursor would feel responsive for a few seconds, then stutter like skipped frames, then recover. Classic symptoms of network jitter.

I also noticed the same issue with [Sunshine](https://github.com/LizardByte/Sunshine)/[Apollo](https://github.com/ClassicOldSong/Apollo) game streaming - input lag that came and went in a regular pattern.

I wrote a quick Python script to measure LAN latency to my router every 200ms. The results were damning:

![Baseline latency showing regular spikes](/assets/images/baseline-latency.png)

Those red spikes hitting 80-90ms every ~1.5 seconds? That's not network congestion. That pattern is too regular, too predictable. Something on my Mac was interrupting the network stack like clockwork.

## The Culprit: AWDL

**AWDL (Apple Wireless Direct Link)** is the protocol behind AirDrop, AirPlay, and Handoff. It works by periodically switching your Wi-Fi radio to a different channel to scan for nearby Apple devices.

During those scans, your normal Wi-Fi traffic waits. For ~80-90ms. Every 1-2 seconds.

## The Evidence

I wrote a test script that toggled various macOS network settings and measured the impact:

![Test results table](/assets/images/results-table.png)

| Setting | Avg Latency | Max Latency | Jitter | Spikes (>15ms) |
|---------|-------------|-------------|--------|----------------|
| Baseline (AWDL on) | 25.4ms | 93ms | 41.6ms | **29%** |
| TCP Delayed ACK off | 34.4ms | 139ms | 58.9ms | 38% |
| **AWDL off** | **4.3ms** | **12ms** | **1.8ms** | **0%** |
| Bluetooth off | 36.5ms | 349ms | 59.8ms | 39.5% |

AWDL was the only setting that mattered. Disabling it:
- **6x lower** average latency
- **8x lower** max latency
- **23x lower** jitter
- **Zero spikes**

![Before and after comparison](/assets/images/before-after.png)

## The Fix

### Step 1: Disable AWDL immediately

```bash
sudo ifconfig awdl0 down

# Verify it worked
ifconfig awdl0 | grep status
# Should show: status: inactive
```

### Step 2: Disable AirDrop and Handoff properly

The `ifconfig` command only lasts until reboot (or until macOS decides to re-enable AWDL). To make it stick, disable the features that use AWDL:

**Via System Settings:**
1. Go to **System Settings → General → AirDrop & Handoff**
2. Set AirDrop to **"No One"**
3. Turn **Handoff** off

**Via Terminal (more thorough):**
```bash
# Disable AirDrop
defaults write com.apple.NetworkBrowser DisableAirDrop -bool YES

# Disable Handoff
defaults write com.apple.coreservices.useractivityd ActivityAdvertisingAllowed -bool NO
defaults write com.apple.coreservices.useractivityd ActivityReceivingAllowed -bool NO
```

### Step 3: After reboots

With AirDrop and Handoff disabled, AWDL usually stays off. But if you notice lag returning, just run:

```bash
sudo ifconfig awdl0 down
```

I initially tried using a launch daemon to automatically disable AWDL at boot, but ran into issues with `launchctl` errors on newer macOS versions. The `defaults write` approach above is cleaner and Apple-supported.

### Reverting (if needed)

If you ever want AirDrop back:

```bash
defaults write com.apple.NetworkBrowser DisableAirDrop -bool NO
defaults delete com.apple.coreservices.useractivityd ActivityAdvertisingAllowed
defaults delete com.apple.coreservices.useractivityd ActivityReceivingAllowed
sudo ifconfig awdl0 up
```

## Human Perception Context

Why does this matter? Here's where those latencies sit relative to human perception:

![Latency in human context](/assets/images/human-context.png)

- **4ms** (AWDL off): Imperceptible, feels like local
- **25ms** (AWDL on average): Noticeable for fast interactions
- **93ms** (AWDL spike): Clearly laggy, breaks flow state
- **100ms**: Threshold where delay becomes consciously perceivable
- **250ms**: Average human visual reaction time

For game streaming and tools like Synergy, the rule of thumb is <20ms for "smooth" and <50ms for "usable". With AWDL on, 29% of my packets exceeded the smooth threshold. With it off, zero did.

## The Scripts

The scripts I used are on [GitHub](https://github.com/adamlovattdevops/slow-wifi) if you want to test your own network:

```bash
# Monitor latency in real-time
python3 jitter-check.py 192.168.1.1

# A/B test different macOS settings
python3 network_optimizer_test.py 192.168.1.1
```

## Conclusion

Apple's seamless device ecosystem comes at a cost: AWDL constantly scanning for nearby devices, interrupting your Wi-Fi multiple times per second. If you're not using AirDrop, AirPlay, or Handoff, turn it off. Your Sunshine/Apollo streams, Synergy sessions, and video calls will thank you.

---

*Tested on macOS Sequoia. Your mileage may vary on other versions.*
