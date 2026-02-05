---
layout: post
title: "AWDL: The Silent Latency Killer on macOS"
date: 2026-02-05
description: "How I fixed 90ms latency spikes on my Mac by disabling a single feature I never use"
---

**TL;DR:** If you're experiencing random lag spikes on your Mac during RDP, gaming, or video calls, disable AWDL (AirDrop's wireless protocol). It fixed my 90ms spikes instantly.

## The Problem

I was getting frustrating mouse "floatiness" during Remote Desktop sessions. The cursor would feel responsive for a few seconds, then suddenly lag, then recover. Classic symptoms of network jitter.

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

If you don't use AirDrop (I never do), disable AWDL:

```bash
# Disable immediately
sudo ifconfig awdl0 down

# Verify it worked
ifconfig awdl0 | grep status
# Should show: status: inactive
```

To make it permanent across reboots, create a launch daemon:

```bash
# Create the plist
sudo tee /Library/LaunchDaemons/com.user.disable-awdl.plist << 'EOF'
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>Label</key>
    <string>com.user.disable-awdl</string>
    <key>ProgramArguments</key>
    <array>
        <string>/sbin/ifconfig</string>
        <string>awdl0</string>
        <string>down</string>
    </array>
    <key>RunAtLoad</key>
    <true/>
</dict>
</plist>
EOF

# Load it
sudo launchctl load /Library/LaunchDaemons/com.user.disable-awdl.plist
```

## Human Perception Context

Why does this matter? Here's where those latencies sit relative to human perception:

![Latency in human context](/assets/images/human-context.png)

- **4ms** (AWDL off): Imperceptible, feels like local
- **25ms** (AWDL on average): Noticeable for fast interactions
- **93ms** (AWDL spike): Clearly laggy, breaks flow state
- **100ms**: Threshold where delay becomes consciously perceivable
- **250ms**: Average human visual reaction time

For RDP, the rule of thumb is <20ms for "smooth" and <50ms for "usable". With AWDL on, 29% of my packets exceeded the smooth threshold. With it off, zero did.

## The Scripts

I've open-sourced the diagnostic tools on [GitHub](https://github.com/adamlovattdevops/slow-wifi):

- **jitter-check.py** - Continuous latency monitor with spike detection
- **network_optimizer_test.py** - A/B test various macOS network settings

## Conclusion

Apple's seamless device ecosystem comes at a cost: AWDL constantly scanning for nearby devices, interrupting your Wi-Fi multiple times per second. If you're not using AirDrop, AirPlay, or Handoff, turn it off. Your RDP sessions, video calls, and games will thank you.

---

*Tested on macOS Sequoia. Your mileage may vary on other versions.*
