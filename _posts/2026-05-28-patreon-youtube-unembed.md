---
layout: post
title: "Patching Patreon From the Client Side"
date: 2026-05-28
description: "Patreon's YouTube embeds are broken. I wrote a 200-line Chrome extension to fix them instead of waiting for support."
---

**TL;DR:** Patreon embeds YouTube videos that won't play — the "sign in to confirm you're not a bot" wall sends you to the YouTube homepage instead of the actual video. The real URL is sitting in plain text on the page the whole time. I built a tiny Chrome extension that grabs it and opens the video properly. Source on [GitHub](https://github.com/adamlovattdevops/patreon-youtube-unembed). Store listing pending.

## The Problem

I'm a paying Patreon subscriber to a couple of creators who post video content. Half the time, the embedded YouTube player on the post page shows:

> Sign in to confirm you're not a bot

Click through, and instead of landing on the video you paid to watch, YouTube dumps you on its homepage. The page knows you're trying to watch a specific video — there's a referrer, there's an embed URL — but the redirect loses it.

Whose fault is this? Honestly, both. YouTube's anti-bot system is overzealous, Patreon's embed wrapper doesn't pass the video ID through to the click-out, and the user — the one who paid — is the one stuck.

I could open a support ticket. Patreon would tell me to clear my cookies. YouTube would tell me to check my account. Both would be wrong. The bug is in the integration, which nobody owns.

## The Hack

Right-click → View Page Source. Ctrl-F for "youtube". Two seconds later:

```html
<script type="application/ld+json">
{
  "@context": "https://schema.org",
  "@type": "VideoObject",
  "@id": "https://www.patreon.com/posts/andrew-charlie-158811832#article",
  "embedUrl": "https://www.youtube.com/watch?v=qU03HBbVWL4",
  ...
}
</script>
```

Patreon ships the actual video URL in its JSON-LD metadata — the structured data Google uses to index the page. It's right there, in plain text, on every post that has a video.

So: copy the URL, paste it into a new tab, video plays. Problem solved.

The first time I did it, it felt like a clever workaround. The fifth time, it felt like a chore. By the tenth, I'd written the extension.

## The Extension

[Patreon → YouTube Unembed](https://github.com/adamlovattdevops/patreon-youtube-unembed) is about 200 lines of JavaScript. It does one thing: when you visit a Patreon page, it scrapes the JSON-LD for any YouTube video IDs, and drops a small red button in the corner that opens them in a new tab.

The whole thing is a Manifest V3 content script. The interesting bits:

1. **Prefer JSON-LD over everything else.** My first version scanned the whole page for YouTube links — iframes, anchor tags, raw text. That broke on post pages, because Patreon's sidebar shows "more from this creator" with thumbnails that link to *other* videos. So I now use the JSON-LD as the authoritative source on single-post pages, and only fall back to the broader scan on creator listing pages (which have no `VideoObject` JSON-LD).

2. **Normalise embed URLs to watch URLs.** `youtube.com/embed/<id>` plays fine in an iframe but feels weird as a standalone tab. A regex turns it into `youtube.com/watch?v=<id>`.

3. **Multi-video posts get a tiny menu.** Some creators link to several videos from a single post. The button shows the count, and clicking it opens a dark dropdown so you can pick one or hit "Open all". Shift-click skips the menu entirely.

4. **Stay out of the way.** First version of the button was a big red pill bottom-right of the page. Felt like a malware popup. Current version is a 24px-tall dim chip that fades up to full opacity on hover — present when you need it, ignorable when you don't.

That's it. No backend. No tracking. No analytics. No remote code. The whole zip is 6KB.

## The Lens: Client-Side Patching

This is the part that's been on my mind.

Modern web apps are increasingly multi-vendor stitch-ups. Patreon embeds YouTube. YouTube has its own anti-bot system. The integration between them is owned by neither team. When it breaks, the support process at either company will be slower than the actual fix.

The trick I keep coming back to: **the data you need is almost always in the page already**. View source, find it, write 50 lines of JavaScript. You can't deploy your fix into their codebase, but you can deploy it into your own browser, and that's enough.

The underlying app has the data and the capability. What's broken is the last-mile UX. And the last mile is exactly where a 200-line extension can win.

It's not glamorous. You're not building an alternative client, you're not reverse-engineering anything, you're not exploiting anything. You're just reading the same HTML the page already gave you and presenting it in a way that doesn't suck. The leverage-to-effort ratio is wild.

## Why It's Not on the Store Yet

Quick caveat: as I write this, the extension isn't on the Chrome Web Store. To publish, Google wants a one-time $5 developer fee tied to a personal account. I don't want to use my work email for a side project I might want to keep if I ever change jobs, and I haven't decided which personal account to anchor it to.

For now, install it unpacked from the [GitHub repo](https://github.com/adamlovattdevops/patreon-youtube-unembed):

1. `chrome://extensions`
2. Toggle Developer mode
3. Load unpacked → pick the folder
4. Open any Patreon post with a YouTube video — small red ▶ button bottom-right

I'll update this post with the store link once it's live.

## Conclusion

Two vendors point at each other. The user pays for the privilege of being stuck in the middle. The data needed to bypass the broken bit is sitting in plain text on the page.

When the support response time is longer than the time it takes to write the fix, you write the fix. The browser is your last-mile platform. Use it.

---

*Source: [github.com/adamlovattdevops/patreon-youtube-unembed](https://github.com/adamlovattdevops/patreon-youtube-unembed). Issues and PRs welcome.*
