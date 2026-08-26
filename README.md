# h3-storyboard

**A Claude Code skill for turning a script into MiniMax H3 shot lists — with emotional performance that actually renders.**

Every other H3 skill stops at translation: you decide what you want, they format it into
H3's prompt schema. None of them cover the part before that — how to break a script into
shots, how many beats a shot can hold, or how to direct a face.

This one does. Every rule comes from real output rather than from reading docs, and
[`SOURCES.md`](skills/h3-storyboard/SOURCES.md) says which ones have a controlled
comparison behind them and which are still inference.

繁體中文說明在 [`skills/h3-storyboard/SKILL.md`](skills/h3-storyboard/SKILL.md)。

---

## The finding that started it

**H3 drops facial instructions silently.** You write "her upper eyelids lift fully open,
her eyebrows rise as one line" and nothing happens — no error, no warning, just a neutral
face for seven seconds.

Same shock close-up, same nine facial beats, two versions:

| | version A | version B |
|---|---|---|
| PSNR between frames 0.2 s apart, at the emotional peak | **42 dB** | **18 dB** |
| what actually happened | nothing — downcast and neutral throughout | eyes widen, head pulls back, eyes flick and return |

42 dB is effectively a frozen frame. Every beat was dropped.

**⚠️ Two variables changed between A and B, and they are not yet isolated:**

| | shot structure | dialogue |
|---|---|---|
| A (failed) | one 7 s close-up holding 9 beats | none |
| B (worked) | split into three 2–3 s shots, one beat each | `<d>` line added |

So the honest claim is: **splitting the shots *and* adding a `<d>` line, together, works.**
The control that would separate them — short shots with no dialogue — has not been run yet.
In practice you want both anyway; each has an independent rationale.

The fix is one line, and the audio gets thrown away anyway:

```
the young woman, in a small unsteady voice that catches once partway through (S1),
says in an off-screen voiceover: <d>[Chinese]...</d>
while her lips remain completely closed.
```

**Emotion goes in the delivery clause, outside `<d>`.** Inside `<d>` is only the language
tag and the verbatim line.

> ⚠️ This is not in any official documentation. MiniMax's H3 prompt-writing guides
> (both base and ref) contain no section on expression, emotion or performance at all.
>
> ⚠️ **Do not confuse H3 with Hailuo 02 / 2.3.** Most "H3 nails micro-expressions" claims
> online describe the *hosted API* models, which are different weights and run behind
> MiniMax's own prompt rewriter. In ComfyUI your text is tokenized verbatim — there is
> no rewriter.

---

## What else is in it

| | |
|---|---|
| **One beat per shot** | 9 micro-beats in a 7 s close-up get averaged away. Split into 2–3 s shots. |
| **Never write "nothing changes"** | The instruction leaks across the whole shot and freezes everything. Put the pause in the edit instead. |
| **Size: crop relationships, not fractions** | `two thirds as tall as the frame` renders at 45–52%. Describing the crop hit 69% first try. |
| **Shape: use a reference image** | Three text attempts at a 1.75:1 panel all produced a square. A blank reference image fixed it in one. |
| **Silent characters use the body** | No dialogue means no mount point. Swap jaw clench → exhale with shoulders dropping. |
| **Verify with PSNR, not eyes** | A dB table for reading whether a beat happened. |
| **Tail collapse** | H3 degrades 1.2–1.7 s before the end. Budget the margin, check every clip. |

Plus the physiological ordering of expressions (brow → eye → mouth), an
emotion-to-observable-action table, a six-step breakdown workflow, and post-production
division of labour.

---

## Install

```bash
npx skills add https://github.com/phileiny/h3-storyboard-skill --skill h3-storyboard
```

Or copy `skills/h3-storyboard/` into your `.claude/skills/`.

**Pairs with [`minimax-h3`](https://github.com/teskor-hub/minimax-h3-skill)** — that skill
covers prompt syntax, ComfyUI setup, quantisation and troubleshooting. This one covers
what to put in the prompt. They do not overlap.

---

## Honesty about scope

[`SOURCES.md`](skills/h3-storyboard/SOURCES.md) sorts every rule into **verified**,
**partly verified** (the effect is real, the cause is not isolated) and **inferred**.

Five have controlled comparisons. One — the big one above — has a confirmed effect with
an unisolated cause. Three are reasoned but untested, including where the beat-density
ceiling actually sits: 9 beats fails, 1 works, the middle is unmeasured.

All of it comes from producing a serialised short-form drama: 9:16, Ref2VA, local ComfyUI,
roughly 60 s and 1,100–1,400 frames per episode.

If you test something here and get a different result, please open an issue — that is
more useful to everyone than the rule as written.
