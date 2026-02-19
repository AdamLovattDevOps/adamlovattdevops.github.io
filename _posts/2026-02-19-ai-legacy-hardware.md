---
layout: post
title: "Should We Use AI to Revive Legacy Hardware?"
date: 2026-02-19
description: "I used a coding LLM to build a game for a twenty-year-old console. The same idea applies to factories, retail, data centres, and billions of devices we throw away."
---

**TL;DR:** AI dramatically lowers the cost of writing software for old hardware. That matters far beyond retro gaming — it has implications for any organisation running legacy kit it can't easily replace.

## The Proof of Concept

![Demon Blaster running on a Sony PlayStation Portable, with the project source code on screen behind it](/assets/images/demon-blaster-psp.png)

I used a coding LLM to build [Demon Blaster](https://github.com/adamlovattdevops/demon-blaster-psp) — a game running at 60 frames per second on a Sony PlayStation Portable from 2004. Native C, compiled against the original SDK, targeting a 333MHz MIPS processor with 32MB of RAM.

Historically, writing software for a platform like this required deep specialist knowledge: MIPS assembly, DMA channels, GPU command lists, memory budgets measured in kilobytes. That expertise is expensive and scarce. When the commercial incentive disappears, the developers move on and the platform dies — not because the hardware stopped working, but because nobody's writing software for it anymore.

Coding LLMs changed the equation. They can hold the entire PlayStation Portable SDK in context alongside real-world open-licensed homebrew code. They understand the platform constraints, reason about memory layouts, and generate working optimised C. An afternoon's work instead of a month's research. Total cost: a subscription.

That's interesting for retro gaming. But the principle is what matters.

## The Principle

Many organisations have legacy hardware. Not because they want it, but because replacing it is expensive, disruptive, or both.

**Retail.** Point-of-sale terminals running embedded Windows or Linux on decade-old processors. They work. The card readers work. The barcode scanners work. But the software is end-of-life, the vendor's gone, and the security patches stopped three years ago. Replacing the lot costs six figures per site. So they stay, unpatched, because nobody can justify the capital expenditure.

**Manufacturing.** Factory floor controllers, PLCs, and HMI panels running on hardware from the 2000s and 2010s. The machines they control still function perfectly. The problem is the software layer — written by a contractor who's long gone, in a language nobody on staff knows, targeting a platform the original vendor has abandoned. Replacing the hardware means recertifying the entire production line.

**Healthcare.** Medical devices with embedded systems that passed regulatory approval a decade ago. The hardware is fine. The software needs updating. But any change triggers a re-certification process that costs more than the device. So it stays frozen, running an OS with known vulnerabilities, because the economics of updating it don't work.

**Data centres.** Hyperscalers like AWS and Google decommission server hardware on three-to-five-year cycles. Not because it fails — because the software stack moves on. Newer frameworks demand newer instruction sets, newer kernel features, newer everything. Perfectly functional compute ends up recycled or scrapped because optimising software for older silicon isn't worth the engineering time.

In every case, the bottleneck is the same: software, not hardware.

## What AI Changes

Writing software for a constrained or legacy platform used to be a specialist job. You needed someone who understood the target architecture, its quirks, its toolchain, its limitations. That person was expensive, hard to find, and had better-paying options elsewhere.

AI compresses that expertise into something accessible on demand. Not perfectly — you still need someone who understands what they're asking for — but the gap between "I know what this system needs to do" and "I can write the code that makes it do that" has narrowed enormously.

For my PlayStation Portable project, that meant going from zero platform knowledge to a shipping product in weeks rather than months. For an enterprise, it could mean:

- Patching legacy point-of-sale software without the original vendor
- Extending the life of factory floor systems by writing new integration layers
- Updating embedded device firmware for platforms where the original toolchain is barely documented
- Optimising workloads for older server hardware to delay capital refresh cycles

The knowledge to do all of this already exists — in old documentation, forum archives, SDK references, open-source projects. AI concentrates it and makes it usable at a fraction of the historical cost.

## The Numbers Favour Extension

New hardware keeps getting more expensive. Semiconductor supply chains remain under pressure — COVID-era chip shortages demonstrated how fragile the pipeline is, and surging demand for AI infrastructure continues to constrain capacity and push up memory prices. Building new fabrication plants costs tens of billions and takes years.

Meanwhile, existing hardware depreciates on a schedule that assumes the software ecosystem will abandon it. If AI changes that assumption — if maintaining and extending software for older platforms becomes cheap — then the depreciation curve flattens. Hardware that was scheduled for replacement becomes viable for another cycle. Multiply that across thousands of devices in a retail chain, hundreds of machines on a factory floor, or racks of servers in a data centre, and the capital savings are significant.

There's a green angle here too, The UN estimates 62 million tonnes of electronic waste per year, with only around 22% formally recycled. Every device that stays in service rather than being replaced is manufacturing, shipping, and disposal that doesn't happen. Extending hardware lifecycles through better software is straightforwardly less wasteful than the alternative.

## The Uncomfortable Question

The consumer electronics industry runs on a replacement cycle: new hardware demands new software, new software demands new hardware, repeat. It works commercially. Whether it works sustainably is a different question.

AI introduces a variable that cycle hasn't accounted for. If one developer with an AI assistant can build performant native software for a twenty-year-old handheld console in weeks, what does it cost to keep a fleet of ten-year-old retail terminals secure and functional? What does it cost to extend a factory controller's software by five years instead of replacing the entire line?

Those numbers are now worth running. For a lot of organisations, the answer will be: less than you'd think.

## Where This Goes

The PlayStation Portable in my drawer was the test case. The idea is broader: AI has quietly made it economically viable to write quality software for platforms that were considered dead. Not just games — any software, for any hardware, at a fraction of what it used to cost.

If software was always the real bottleneck — and it was — then the calculus on replace versus extend just changed.

---

*The [Demon Blaster](https://github.com/adamlovattdevops/demon-blaster-psp) source is on GitHub if you want to see what AI-assisted legacy platform development actually looks like in practice.*
