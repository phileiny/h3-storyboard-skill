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

**H3 drops facial instructions silently when a shot holds too many of them.** You write
"her upper eyelids lift fully open, her eyebrows rise as one line" and nothing happens —
no error, no warning, just a neutral face for seven seconds.

Three versions of the same shock beat. Same seed, same references, same facial
instructions. One variable changed at a time:

| | shot structure | dialogue | PSNR at the emotional peak | result |
|---|---|---|---|---|
| **A** | one 7 s close-up holding 9 beats | none | **37–42 dB** | face never moved |
| **B** | three 2–3 s shots, one beat each | none | **22–23 dB** | **every beat landed** |
| **C** | same as B | `<d>` line added | **19 dB** | slightly larger range |

Lower PSNR means more change between frames. 42 dB is effectively a frozen frame.

**A → B is the mechanism.** Splitting the shots recovers almost all of it.
**B → C is a bonus.** The dialogue tag helps, but it is not what was blocking the
performance.

So: **count your beats before you write anything else.** More than two or three
expression beats in one shot and it needs splitting.

> An earlier version of this repo credited the dialogue tag as the cause. That was wrong —
> the failing and working versions had changed two things at once. Running the missing
> control is what corrected it. See [`SOURCES.md`](skills/h3-storyboard/SOURCES.md).

## What else is in it

| | |
|---|---|
| **Never write "nothing changes"** | Likely leaks across the whole shot. Put the pause in the edit instead — controllable either way. |
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

Five have controlled comparisons, including the headline one above. One is plausible but
not isolated. Four are reasoned and untested — among them where the beat-density ceiling
actually sits: 9 beats fails, 1 works, the middle is unmeasured.

All of it comes from producing a serialised short-form drama: 9:16, Ref2VA, local ComfyUI,
roughly 60 s and 1,100–1,400 frames per episode.

If you test something here and get a different result, please open an issue — that is
more useful to everyone than the rule as written.
