---
layout: post
title: "If the Binaries Exist: Rescuing a Lost 2009 Mobile Game Nobody Played"
date: 2026-08-17
description: "An obscure delisted advergame, a fifteen-year community hunt, an AI-assisted teardown of an undocumented Unity format — and the question of how much digital culture will actually be preserved, and by whom."
published: false
---


**TL;DR:** I used AI to tear down a delisted, undocumented 2009 mobile
game — dead file format, dead hardware, no tooling support — and rebuild it
as a native port that will outlive every device that could run the
original. The game itself is a curiosity. The point is what the exercise
demonstrates: reverse engineering has stopped being expensive, which
changes who can do software preservation, how much of our digital culture
might actually survive, and how long yesterday's DRM keeps meaning
anything. If the binaries exist, AI can and will take them apart.

In 2009, Eminem's comeback album *Relapse* got the full promotional
apparatus: singles, a horror-styled video campaign, and — announced by
tweet, released on album day for $2.99 — an iPhone game. Six levels,
41.7 MB, built by a small Florida contract studio in the App Store's first
year, on a marketing deadline. You don't need my opinion of it, because
2009 left one on record: the only contemporary review I could find — and
that I could find only one is its own data point — declared it
["literally the worst game I have ever been subjected to in my entire
life"](https://web.archive.org/web/20221129152921/https://thekoalition.com/reviews/eminem-relapse-the-game-review),
while a launch-week blog post settled for "Relapse = FAIL". It was
quietly delisted — still on the store when that review ran six weeks in,
gone at some unrecorded point after — and almost nobody played it then or
has thought about it since.

The premise, when you reach it, takes some easing into. You play Eminem
fighting his way out of a psychiatric rehab facility, and the things
standing between you and the exit are the patients. You punch them — and
the orderlies, and the doctors — to earn "will power points". At 400
points a 9mm materialises. At 1200, an M16. If your tolerance meter fills,
the screen says **YOU RELAPSED** and you start again. It is, in short, a
deeply strange object: an official, label-adjacent product with the logic
of a fever dream, very much of a year when the App Store would approve a
taser-noise app and nobody blinked. File that strangeness away; it turns
out to matter for the preservation question, not just the curiosity one.

Meanwhile, its very unloveliness is what makes it the right specimen: no
community was ever going to do this work, so it's a clean test of what one
person and an AI can manage alone.
The game is now fully extracted, decompiled, audited method by method, and
reimplemented as a native C++ port that runs on Linux, Windows, Android
and — cross-compiled from Linux with no Mac in sight — iOS. It will outlive
every device that could ever run the original. The gap between what this
thing *is* and what it took to save it is the story, and I don't think the
implications stop at novelty promo games.

## Popsomp Hills, $2.99, and the sequel that never came

The game was credited to "Shady Games and DS Media Labs" — the former
presumably the artist-side label (the bundle ID is
`com.shadygames.Relapse`), the latter a real and traceable studio. DS Media
Labs, Inc. was a small iPhone contract shop working out of the West
Palm Beach area (four of them flew to WWDC '09), and their [archived blog](https://web.archive.org/web/20090828032318/http://www.dsmedialabs.com/blog.php) reads like a core sample of the
early App Store: a taser-noise app (*Stun-O-Matic*), a light-cycles clone
(*Light Riders*), *FLOverload*, a *Fall Out Boy All Access* app, and — a few
months after Relapse — *Ramp Champ*, a genuinely well-regarded skeeball game
built with The Iconfactory. In mid-2009 they posted a job listing for a 3D
developer: "Unity 3D, Mathematics, C# and the Mono/.NET frameworks",
OpenGL ES a bonus. That listing is effectively this game's build manifest —
it describes the exact stack the teardown later fell out of.

The launch beat was real, if brief. On May 3, 2009 Eminem tweeted "Relapse
iPhone game coming along nicely" with a screenshot: a buff mini-Em wielding
a **2-by-4** at a bloody-faced zombie in a decrepit lobby. [MTV News picked
it up](https://web.archive.org/web/20221220013255/https://www.mtv.com/news/ao3j50/eminem-readying-relapse-iphone-game) two days later; [Pocket Gamer covered it](https://web.archive.org/web/20260817174903/https://www.pocketgamer.com/relapse/eminem-resurfaces-with-iphone-game/) the day after that and
wondered whether Apple would even approve it, the App Store having recently
rejected a Nine Inch Nails app for "objectionable content". Apple approved
it. The game landed on May 19, 2009 — album day — at $2.99, with a [trailer
on Skee.TV](https://www.youtube.com/watch?v=ujwng1czda8). The studio's own blurb: "you assume the role of Eminem as he
fights his way out of Popsomp Hills, the rehab center he was remanded to.
Insane patients, Orderlies, and Nurses will stop at nothing to see that Em
never makes it out alive." (The blurb says Nurses; the code says Doctors.
Even the marketing copy and the enemy table disagree.)

A week after launch, the same blog post promised version 1.1: longer
levels, smarter AI, a more responsive Eminem, a skippable intro, and — the
good part — "a two by four for orderly smacking goodness". Here is where
the archaeology gets satisfying: that 2-by-4, the weapon in the
*announcement screenshot*, is sitting in the shipped binary's weapon table
right now — `TwoByFour`, range 0.5, damage 0.35, between the fist and the
handgun — and the shipped game has no way to give it to you. Only the 9mm
and the M16 ever drop. It was cut between the announcement and launch, its
stats left behind in the code, then promised back for 1.1. And when you
finish the game, the ending video closes on three words in scratched
grindhouse type: **TO BE CONTINUED…** — I pulled that frame out of the
shipped `end.m4v` myself.

None of it happened — as far as anyone can prove. The game vanished from
the App Store not long after launch and took the whole plan with it. No
continuation, no sequel; and whether 1.1 itself ever landed is now
unknowable in an instructive way. The one binary that survives is 1.0 —
the archive.org item is literally named `relapse-1.0.0`. If an update did
ship in the game's short time on the store, nobody kept a copy, and the
App Store keeps no public record of version history for software it has
delisted. So the promised patch is either vapour or lost media nested
inside lost media — a version of a game nobody preserved, of which we
cannot even establish the existence. That is how thin the record gets,
seventeen years out, for anything that lived only in a storefront. A "to be
continued" in a delisted binary is about as clean a monument to abandoned
digital culture as you could design on purpose.

## The fifteen-year hunt

None of this starts with me. It starts with the YouTube creator
[COBYSUCKS!](https://www.youtube.com/watch?v=sv6y-zlXtlg), whose documentary
*The 15 Year Hunt For The Lost Eminem Game* records how a chain of
volunteers dragged *Relapse: Resistance* back from the edge of lost media.
The short version: the IPA surfaced anonymously on
[archive.org](https://archive.org/details/relapse-1.0.0) in July 2024 and
sat unnoticed for months. When it was finally found, the game's framework
turned out to be gone from iOS 10 onward — it needs iOS 9 or earlier —
forcing everything onto period hardware — a six-hour guided jailbreak of an eBay iPhone 3G, a dead Wi-Fi
chip that nearly ended the whole project, a defunct screen-mirroring
utility brought back from the grave by spotting a broken domain prefix in
a URL, strangers donating decade-old hardware to the cause. If you have
any affection for a lost-media quest, watch the documentary — the people
who made each step happen are thanked properly there, and the full story
is better told at video length than I can manage here.

That is what software preservation actually looks like: unglamorous,
contingent, and absurdly disproportionate to the artifact — years of
volunteer effort to get an obscure album tie-in running long enough to point a
camera at it. And at no point does anyone in that story stop to ask whether
the game deserved it, which is the correct attitude. Preservation that only
saves the good stuff isn't preservation, it's curation, and the culture of
2009 includes its marketing sludge whether we like it or not.

## Life support has a clock on it

What that group achieved is best described as life support. The game runs —
on a shrinking pool of 32-bit ARM devices, behind jailbreaks, with capture
rigs held together by community generosity. The binary is armv6. Apple
dropped 32-bit app support in iOS 11 and armv6 fell out of the toolchain
years before that; there is no simulator slice in the IPA, and no emulator
runs it — that has been tried. Period hardware is the only path, every year
there is less of it, and when the last capacitor in the last iPhone 3G gives
out, "the game runs" stops being true forever.

I wanted the other kind of preservation: the kind where the artifact no
longer needs its host. Extract the assets, the constants and the behaviour
into an open specification, reimplement against commodity APIs, and the game
survives the hardware. So that's the project — a full teardown of the
archive.org IPA (md5 `d3ef1baa8a2e15264b3ca62f725adcab`, verified against the
upload) and a native port: C++17, SDL2, OpenGL 3.3, one binary, no
third-party asset libraries.

## The format nobody could read

Here is where it gets properly low-level, because the first wall is a good
one.

The IPA's Unity data files are serialized-file **format version 6** — Unity
iPhone 1.0.2f4, 2009. Later Unity formats can embed a *type tree*: a schema,
per class, describing every field and its width, so a tool can walk any
object without prior knowledge — and where a build strips it, modern tools
fall back on curated type databases. Version 6 has neither option. No type trees, no
version string, just an object table — offsets, sizes, class IDs — and then
opaque bytes. UnityPy and AssetRipper will happily enumerate the objects and
then shrug at every body: their type databases reach back a long way, but
not to 2009. Nothing here is encrypted or obfuscated. The format is merely
*forgotten*, which for preservation purposes is indistinguishable from
protected.

So the schema had to be reconstructed, and it came in two halves.

**Field order came from the shipped binary itself.** Unity's serializer
registers field names in order, and those names survive as plain strings
in the armv6 Mach-O — so the string table preserves the field sequence. One command gives you the skeleton of the Mesh
serializer:

    strings -a -t d Relapse | grep m_IndexBuffer
    # and its neighbours in the string table: m_SubMeshes, m_Vertices, ...

The 2009 binary carries its own field manifest, in order, for every
serializable class. It just doesn't tell you how wide anything is.

**Field widths came from constraint-solving against the data.** Pick
candidate widths, parse, and check invariants that cannot hold by accident:

- a parse must consume its object *exactly* — end one byte short or long and
  the candidate dies;
- texture dimensions must be powers of two;
- every mesh index must be less than the vertex count;
- normals must be unit length.

Guess wrong and something violates one of those within a handful of objects.
Guess right and **1,354 objects parse with zero failures** — which is what
happened. That's the whole trick: the file format documents itself if you
hold it to standards it can't fake.

Some of what fell out, for flavour:

- **Meshes** are 16-bit **triangle strips** (2009 iPhone GPU, of course they
  are), de-stripped to triangle lists at pack time. `TangentSpace` is seven
  floats per vertex — normal, tangent, handedness. Skin weights and bind
  poses parse too; the player is a 34-joint skeleton with 37 animation clips.
- **AudioClips** come in two flavours: trailing raw PCM16, or raw AAC access
  units with a CoreAudio ASBD, an `esds`, and a 16-byte-per-packet table —
  and **no ADTS framing**, so nothing will play them as-is. The extractor
  reframes the packets into ADTS; decoded durations match the stored
  `m_Length` to within one 23 ms packet, which is how you know the reframing
  is right and not merely tolerated.
- **Shaders**: 75 ship, 28 mention shadows, and precisely none of the
  shadow ones are used by any of the game's six materials — the iPhone
  player is OpenGL ES 1.1 fixed-function and every shadow you see is baked
  into five lightmap textures. Sweeping the binary to prove a *negative*
  (no real-time shadows, no fog, no projectors — nothing to port) is as much
  a part of fidelity as implementing things.

The other half of the ground truth was a gift: the game logic ships as
`Assembly - CSharp.dll`, .NET IL, and IL decompiles into something
substantially identical to the original C#. 33 classes, 5,232 lines — the
entire game, readable. And the Mach-O has `cryptid = 0`: this app was
**never FairPlay-encrypted**. No protection was circumvented because there
was no protection. The whole game sat in the open; it just took fifteen
years and a chain of volunteers before anyone could look.

## Decompiled code as specification — and getting burned by it

The port treats the decompiled C# as a spec, and the fidelity audit
(`FIDELITY.md`: all 195 methods across the 33 types, every ported constant
required to cite the `file:line` it came from) is honest about how that
went: **17 defects**. A few generalise well beyond this game.

**D1, the transposed weapon table.** `Weapon`'s constructor is

    Weapon(WeaponType type, bool automatic, float effectiveRange, float damage, int defaultAmmo)

Range third, damage fourth — semantics fixed by how the body *uses* them
(`_effectiveRange` feeds `Physics.Raycast`'s distance, `_damage` feeds
`ApplyDamage`), not by what the decompiler names anything. The port had read
the table sideways. The fist — real damage 0.25 against enemy health of
1.0–1.5 — was doing 1.25. The handgun was doing 10, which is its *range*.
The rifle was doing 20. Fist 5× too strong, handgun 25×, rifle 40×. Every
enemy in the original takes four to six punches; in the port they were dying
in one. Worse, the automated end-to-end test had been built on the wrong
numbers, so fixing the bug *broke the test* — the test suite had encoded the
defect as an expectation. The playthrough got 50 seconds slower when the
numbers were corrected, because the bot now has to stand and fight instead
of one-punching its way through the hospital. Fidelity, measured in
inconvenience.

**D5, the jump that was never a jump.** The port implemented a ballistic
arc — `jump_speed = 5.2`, `gravity = -14.0`, a landing test. Reasonable,
obvious, and entirely invented: neither constant exists anywhere in the
original. `ActorController.Update` ends with, unconditionally, every frame:

    transform.position = new Vector3(transform.position.x, 0f, clampedZPlane);

Y is pinned to zero. The original's jump is an animation, a sound, and an
invulnerability frame — `Weapon.Attack` refuses to damage a jumping actor,
so the "jump" is actually a dodge you can spam (that's D4, a real mechanic
hiding behind a fake one). The character never leaves the floor. Restoring
this makes the jump feel broken to anyone who only knows the port, which is
exactly the point.

**D6, the wrong easing curve.** Both the player and the camera use
`Mathf.SmoothDamp` — a critically-damped spring, an exponential approach
that never quite arrives. The port had a linear ramp that hits the target
and stops dead. Different response, different stopping distance, different
camera lag. The fix is cheap because `SmoothDamp` is fully specified — it's
a short polynomial approximation of the exponential, and the original's
tuning (`maxWalkVelocity 4.0`, `accelerationTime 0.15`) comes straight out
of the scene data.

**D14, the most useful negative result.** Two classes on the "missing"
list — `CameraFadeInOut`, `LoadingLevelController` — are unreachable in the
shipped build. One is attached to no GameObject in any scene and referenced
by no script. The other ships *inactive*, and both call sites that would
show it are gutted into empty bodies — literally
`if (loadingLevelScreen != null) { }`. Someone at the studio stubbed the
loading screen out and shipped. Implementing these classes would have made
the port *less* faithful. Sixteen methods of apparent work, correctly not
done: a class existing in the assembly says nothing about it running.

**D8, where faithful and correct disagree.** Level 5 has two exit triggers
and both load `MainMenu` — one of them 6.58 units *behind* the spawn. And
`GameController.LoadScene` ends with:

    if (Application.loadedLevelName == "Level5") PlayerPrefs.SetInt("PlayEndingSequence", 1);

— on **any** departure from Level5. Walk six units the wrong way at the
start of the final level and you get the ending video. That is a bug in the
2009 game, faithfully preserved and written down as a decision, because
otherwise someone "fixes" it back and forth forever. (The level data even
contains the clean fix if ever wanted: the forward exit is always the child
of the elevator carrying the `ElevatorDoorOpener` script. Noted, declined.)

**D16, the moonwalk.** The original tracks movement as four booleans —
`movingLeft`, `movingRight`, `wantsToMoveLeft`, `wantsToMoveRight` — and
they encode something a signed axis cannot: press right while holding left
and the new input parks in `wantsToMoveRight` while `lookDirection` stays
put, so you walk right while facing left, and the animation controller runs
the walk cycle at *negative speed*. The port had summed the two keys to zero
and stood still. Eminem is supposed to be able to moonwalk. He now can. The
animation sampler had to learn to wrap negative time to make the clip play
backwards, which is possibly the most effort ever spent on a feature no
player will find.

And the rule that would have prevented most of the seventeen: **every defect
was a hand-transcribed constant.** Values read out of the shipped scene data
at pack time — enemy health, walk speeds, camera bounds, spawn frequencies —
never drifted, not once. Values that lived in *code* and were retyped by
hand into a header did. A constant nobody typed cannot be transposed. The
fix is structural: parse `WeaponFactory`'s constructor calls out of the
decompiled source at pack time and emit them into the data files, and treat
any uncited constant in the port as suspect — the four uncited floats in
`game.h` are exactly what turned out to be wrong.

## The port, briefly

C++17, SDL2, OpenGL 3.3. Offline tools bake the extracted data into three
pack formats: per-level paks (textures as raw RGBA, de-stripped meshes,
baked world matrices, the scene camera, spawn parameters, exits), a player
pak (skeleton, skinned mesh, weapon attachments bound to wrist joints, 37
clips), and a game pak (sprite atlases with precomputed UVs, all audio
pre-decoded to 22.05 kHz mono PCM16 so the runtime needs no codec).

The end-to-end test plays the game with a bot that inputs no faster than a
human — about three attacks a second, a jump or two per level, no
fast-forward, deliberately no way to add one — and asserts what actually
happened: every level reached, the right music on each, footsteps, screams,
the 9mm dropping at 400 points with its holster sound, a screenshot per
level, none of them black or blown out. Four scenarios, including a pacifist
run (must die) and a backtrack run (walking out of a level's entrance exit
must return to the title without counting as a win). Full playthrough:
275.8 seconds.

The iOS build deserves a paragraph, because it is built **on Linux**. lld
speaks Mach-O, so stock clang plus an iPhoneOS SDK sysroot plus `ldid` for
signing gets you an `.ipa` with no Mac involved. Three things a Mac would
have silently hidden: SDL2 mixes ARC `.m` files with manual-retain `.c`
files, so `-fobjc-arc` has to be set per file — one global setting fails
whichever way you set it; Apple's `@available` runtime check
(`__isPlatformVersionAtLeast`) lives in Apple's compiler-rt, which Ubuntu's
clang doesn't ship, so a small compat shim reads `kern.osproductversion` the
way the real one does; and the difference between a fullscreen app and a
letterboxed compatibility-mode one is an empty `UILaunchScreen` dict in
Info.plist — the alternative is a compiled storyboard, which needs `actool`,
which needs a Mac. There is a pleasing symmetry in the port returning to
iPhone hardware via SideStore, on a signing certificate that expires every
seven days: the original platform is now the *least* hospitable one this
game runs on.

## The disproportion, stated plainly

Hold the two halves next to each other.

On one side: a six-level advergame where Eminem punches hospital
patients, shipped to sell an album, with a stubbed-out loading screen and a
bug that rolls the credits if you walk left. Nobody played it. On the other:
a frontier AI system reconstructing an undocumented seventeen-year-old
serialization format from the binary's own string table, constraint-solving
field widths against parse-exactness and unit-length normals, reframing
headerless AAC by packet table, auditing 195 decompiled methods against a
reimplementation, and catching the exact place where a decompiler's
parameter names disagreed with the argument semantics.

The disproportion is the point. This level of effort used to be reserved for
software a community could sustain obsession over — Mario 64, Ocarina of
Time, decompilations with decades of person-years behind them. The economics
gatekept preservation: only the beloved got saved. What this project
demonstrates, in a small way, is that the economics have flipped. The
cathedral treatment now costs one person a few weeks, so the janky ad gets
the cathedral treatment — and "is it worth it?" stops being a technical
question at all.

So ask the non-technical version honestly: should anyone bother preserving
what is essentially an advertisement? I think the question contains its own
answer, because *nobody knows in advance*. Nobody in 2009 flagged this game
as culturally significant — it took fifteen years for anyone to notice it
was gone. The App Store has delisted software by the hundreds of thousands;
Flash took a whole medium with it; every dead platform strands a generation
of work, and the overwhelming majority of it — almost certainly the vast
majority of all digital art ever made — will simply not survive, because
survival currently depends on someone caring at exactly the right moment.
And some of it will be lost on purpose: a product this strange, this far
from how its rights holder wants to be remembered, is exactly the kind of
asset that gets quietly left to rot. Embarrassment is a preservation risk
in its own right — nobody works to keep their oddities online — and
deliberate forgetting is still forgetting. There is no legal deposit library for mobile
software. No institution was ever coming for *Relapse: Resistance*. The people who led this charge were
an anonymous uploader, a fan who got curious, a jailbreaker with a free
evening, and a YouTuber — volunteers, every one, doing institutional work
with none of the mandate and none of the budget. If that's who leads, then
the thing that changes the percentage isn't more institutions. It's
dropping the cost of the work until the volunteers can afford everything.

## What happens when this scales

Which brings me to the thesis, and it's the uncomfortable one.

Everything above worked because the artifact couldn't fight back. A 2009
Unity game has no type trees, but it also has no anti-tamper, no server-side
logic, no attestation — and, thanks to `cryptid = 0`, not even the
encryption its own platform offered. But look at what the method actually
was: recover structure from strings the binary couldn't help shipping; pin
the unknowns with invariants; treat decompiled output as a specification;
audit the reimplementation against it adversarially. None of that is
game-specific. It is a general procedure for digesting binaries, and every
step of it rides the same capability curve the models do.

Scale it up and the targets stop being novelty games. Bigger games.
Middleware. Operating systems. The 90s and 2000s are full of software
protected by schemes that were *economically* secure rather than
mathematically strong — SafeDisc, SecuROM, dongle checks, bespoke archive
formats, licence blobs. These were never walls. They were toll booths,
priced against scarce human attention: not worth a skilled reverser's month
for an abandoned CAD package or a dead MMO client. AI reprices human
attention. The month becomes an afternoon, the afternoon becomes a batch
job, and old DRM doesn't get broken so much as **outlived** — it was always
executing on the attacker's own hardware, holding its keys somewhere in the
process, and obscurity was the only thing it ever really had.

So I'll state it flatly: **if the binaries exist, AI can and will take them
apart.** Decrypt the weak stuff, strip the protection, recover the formats,
port the lot. Not tomorrow and not all at once, but monotonically — every
artifact that survives in binary form is on a conveyor belt toward
legibility, and nothing on the protection side of the ledger compounds the
way the analysis side does. Real cryptography holds; your TLS session is
fine. But DRM was never cryptography in that sense. It was economics plus
obscurity, and both inputs are collapsing.

For preservation this is straightforwardly liberating, and this project is
the demonstration. The alternative was watching the last armv6 iPhone die.
Every abandoned format, every orphaned save file, every dead middleware
runtime is now recoverable in principle and increasingly in practice — and
the unloved artifacts benefit most, because they were never going to earn a
fan decompilation. The DMCA's preservation exemptions were drafted imagining
museums and libraries with staff reverse-engineers; the reality is going to
be individuals with a model and a weekend, and the legal framework has not
begun to metabolise that.

For rights holders it is the same fact with the sign flipped, and I won't
pretend otherwise. The tool that reconstructs a delisted 2009 advergame from
serialized bytes reconstructs a live product just as readily. The
distinction between this project and piracy is legal and ethical, not
technical: the game is delisted and unmonetizable; nothing extracted is
redistributed — the repo ships tools and audit documents and readers supply
their own IPA; and the binary was never encrypted, so no protection measure
was touched. Those lines matter and I've stayed on the right side of every
one. But they are lines *I* drew. The capability doesn't draw them, and the
next person holding it might not either.

## Coda

Somewhere in the 275.8-second verified playthrough, Eminem punches a doctor
six times — 1.5 health, 0.25 a punch, the arithmetic is now correct — while
the walk cycle plays backwards, because moonwalking was in the original's
movement flags and now it's in the port's. Nobody will ever need this. Almost
nobody wanted the game the first time. That was never the question. The
question was whether a piece of 2009, silly as it is, outlives the hardware
it was chained to — and whether one person with an AI can now do for the
unloved what whole communities used to do for the beloved.

It does, and they can. Thanks to COBYSUCKS! and every volunteer in his
documentary's chain — they spent fifteen years proving this thing was worth
saving, so that the saving could take a few weeks. Go watch it.

---

*The port's source, extraction tools and audit documents are the publishable
output of this project. The game's code, art and music remain the property
of Shady Games / DS Media Labs; no game content is redistributed, and the
tools require you to supply your own IPA.*
