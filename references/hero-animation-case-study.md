# Gallery Ripple + Multi-Focus · Scene Choreography Philosophy

> **A reusable visual choreography structure** distilled from the huashu-design hero animation v9 (25 seconds, 8 scenes).
> Not an animation production pipeline — it's about **what scenarios make this choreography "right"**.
> Real-world reference: [demos/hero-animation-v9.mp4](../demos/hero-animation-v9.mp4) · [https://www.huasheng.ai/huashu-design-hero/](https://www.huasheng.ai/huashu-design-hero/)

## One-line Summary

> **When you have 20+ visually homogeneous assets and the scene needs to "express scale and depth", prioritize Gallery Ripple + Multi-Focus over stacking layouts.**

General SaaS feature animations, product launches, skill promotions, series portfolio showcases — as long as the asset count is sufficient and the style is consistent, this structure almost always works.

---

## What Exactly Does This Technique Express

It's not "showing off assets" — it's telling a narrative through **two rhythmic shifts**:

**Beat 1 · Ripple Unfold (~1.5s)**: 48 cards spread out from the center to the periphery. The viewer is shocked by "the quantity" — "oh, this thing has produced this much".

**Beat 2 · Multi-Focus (~8s, 4 cycles)**: While the camera slowly pans, the background dims + desaturates 4 times, isolating one card and enlarging it to the screen center — the viewer switches from "shock at quantity" to "gaze at quality", with each cycle holding a stable 1.7s rhythm.

**Core narrative structure**: **Scale (Ripple) → Gaze (Focus × 4) → Fade out (Walloff)**. These three beats combined express "Breadth × Depth" — not just that you can do many things, but each is worth pausing to look at.

Compare with anti-examples:

| Approach | Viewer perception |
|------|---------|
| 48 cards arranged statically (no Ripple) | Looks nice but no narrative, like a grid screenshot |
| One-by-one quick cuts (no Gallery context) | Like a slideshow, loses the "sense of scale" |
| Only Ripple, no Focus | Shocking but no specific card is memorable |
| **Ripple + Focus × 4 (this recipe)** | **First shocked by quantity, then gaze at quality, finally calm fade-out — complete emotional arc** |

---

## Prerequisites (all must be met)

This choreography is **not universal**. The following 4 conditions are all required:

1. **Asset count ≥ 20, preferably 30+**
   Fewer than 20 makes Ripple appear "empty" — only when every cell of the 48 grid is moving does the density feel right. v9 used 48 cells × 32 images (cyclically filled).

2. **Assets have consistent visual style**
   All 16:9 slide previews / all app screenshots / all cover designs — aspect ratios, color tones, layouts must look like "a set". Mixing styles makes the Gallery look like a clipboard.

3. **Assets remain readable when individually enlarged**
   Focus enlarges a card to 960px wide. If the original blurs or has thin information when enlarged, the Focus beat is wasted. Reverse verification: can you pick 4 of the 48 as "most representative"? If not, the asset quality is uneven.

4. **The scene is landscape or square, not portrait**
   The Gallery's 3D tilt (`rotateX(14deg) rotateY(-10deg)`) needs horizontal extension; portrait makes the tilt look narrow and awkward.

**Fallback paths when conditions are missing**:

| Missing what | Degrade to what |
|-------|-----------|
| Assets < 20 | Switch to "3-5 side-by-side static display + focus each one in turn" |
| Inconsistent style | Switch to "cover + 3 chapter big images" keynote-style |
| Thin information | Switch to "data-driven dashboard" or "punchline + large text" |
| Portrait scene | Switch to "vertical scroll + sticky cards" |

---

## Technical Recipe (v9 production parameters)

### 4-Layer Structure

```
viewport (1920×1080, perspective: 2400px)
  └─ canvas (4320×2520, oversized overflow) → 3D tilt + pan
      └─ 8×6 grid = 48 cards (gap 40px, padding 60px)
          └─ img (16:9, border-radius 9px)
      └─ focus-overlay (absolute center, z-index 40)
          └─ img (matches selected slide)
```

**Key**: canvas is 2.25× larger than viewport, so panning gives a "peeking into a larger world" feeling.

### Ripple unfold (distance-delay algorithm)

```js
// Each card's entry time = distance from center × 0.8s delay
const col = i % 8, row = Math.floor(i / 8);
const dc = col - 3.5, dr = row - 2.5;       // offset to center
const dist = Math.hypot(dc, dr);
const maxDist = Math.hypot(3.5, 2.5);
const delay = (dist / maxDist) * 0.8;       // 0 → 0.8s
const localT = Math.max(0, (t - rippleStart - delay) / 0.7);
const opacity = expoOut(Math.min(1, localT));
```

**Core parameters**:
- Total duration 1.7s (`T.s3_ripple: [8.3, 10.0]`)
- Max delay 0.8s (center appears earliest, corners latest)
- Each card entry duration 0.7s
- Easing: `expoOut` (burst feeling, not smooth)

**Done simultaneously**: canvas scale from 1.25 → 0.94 (zoom out to reveal) — pairs with the synchronized push-back feeling as elements appear.

### Multi-Focus (4-cycle rhythm)

```js
T.focuses = [
  { start: 11.0, end: 12.7, idx: 2  },  // 1.7s
  { start: 13.3, end: 15.0, idx: 3  },  // 1.7s
  { start: 15.6, end: 17.3, idx: 10 },  // 1.7s
  { start: 17.9, end: 19.6, idx: 16 },  // 1.7s
];
```

**Rhythm pattern**: Each focus 1.7s, 0.6s breathing interval. Total 8s (11.0–19.6s).

**Inside each focus**:
- In ramp: 0.4s (`expoOut`)
- Hold: middle 0.9s (`focusIntensity = 1`)
- Out ramp: 0.4s (`easeOut`)

**Background changes (this is the key)**:

```js
if (focusIntensity > 0) {
  const dimOp = entryOp * (1 - 0.6 * focusIntensity);  // dim to 40%
  const brt = 1 - 0.32 * focusIntensity;                // brightness 68%
  const sat = 1 - 0.35 * focusIntensity;                // saturate 65%
  card.style.filter = `brightness(${brt}) saturate(${sat})`;
}
```

**Not just opacity — simultaneously desaturate + darken**. This makes the foreground overlay's colors "pop", rather than just "becoming a bit brighter".

**Focus overlay size animation**:
- From 400×225 (entry) → 960×540 (hold state)
- Surrounded by 3 layers of shadow + 3px accent color outline ring, presenting "the feeling of being framed"

### Pan (continuous motion keeps stillness from being boring)

```js
const panT = Math.max(0, t - 8.6);
const panX = Math.sin(panT * 0.12) * 220 - panT * 8;
const panY = Math.cos(panT * 0.09) * 120 - panT * 5;
```

- Sine wave + linear drift dual-layer motion — not pure looping, every moment's position is different
- X/Y frequencies differ (0.12 vs 0.09) to avoid visually perceptible "regular looping"
- Clamped at ±900/500px to prevent drifting out

**Why not pure linear pan**: With pure linear, viewers can "predict" where the next second will be. Sine + drift makes every second new; under 3D tilt it produces "slight motion sickness" (the good kind), holding attention.

---

## 5 Reusable Patterns (distilled from v6→v9 iteration)

### 1. **expoOut as the main easing, not cubicOut**

`easeOut = 1 - (1-t)³` (smooth) vs `expoOut = 1 - 2^(-10t)` (burst then rapidly converge).

**Reason for choice**: expoOut's first 30% quickly reaches 90%, more like physical damping, matching the intuition of "heavy things landing". Especially suited for:
- Card entry (sense of weight)
- Ripple spread (shockwave)
- Brand floating up (sense of settling)

**When to still use cubicOut**: focus out ramp, symmetric micro-animations.

### 2. **Paper-feel base color + terracotta orange accent (Anthropic lineage)**

```css
--bg: #F7F4EE;        /* warm paper */
--ink: #1D1D1F;       /* near black */
--accent: #D97757;    /* terracotta orange */
--hairline: #E4DED2;  /* warm hairline */
```

**Why**: Warm base color still feels "breathing" after GIF compression, unlike pure white which feels "screen-y". The terracotta orange as the sole accent runs through terminal prompt, dir-card selected, cursor, brand hyphen, focus ring — all visual anchors are strung together by this one color.

**v5 lesson**: Added a noise overlay to simulate "paper texture", but the result was ruined by GIF frame compression (every frame differs). v6 changed to "just base color + warm shadow", retained 90% of paper feel, and GIF size shrank by 60%.

### 3. **Two-tier shadow simulating depth, no real 3D**

```css
.gallery-card.depth-near { box-shadow: 0 32px 80px -22px rgba(60,40,20,0.22), ... }
.gallery-card.depth-far  { box-shadow: 0 14px 40px -16px rgba(60,40,20,0.10), ... }
```

Use a deterministic algorithm `sin(i × 1.7) + cos(i × 0.73)` to assign each card a near/mid/far shadow tier — **visually has the feeling of "three-dimensional stacking", but the transform is completely unchanged each frame, GPU consumption 0**.

**The cost of real 3D**: Each card individually uses `translateZ`, GPU calculates 48 transforms + shadow blurs every frame. v4 tried it, even Playwright recording 25fps was strained. v6's two-tier shadow has <5% visual difference to the eye, but 10× cost difference.

### 4. **Font-weight changes (font-variation-settings) more cinematic than font-size changes**

```js
const wght = 100 + (700 - 100) * morphP;  // 100 → 700 over 0.9s
wordmark.style.fontVariationSettings = `"wght" ${wght.toFixed(0)}`;
```

The brand wordmark transitions from Thin → Bold over 0.9s, paired with letter-spacing fine-tuning (-0.045 → -0.048em).

**Why this is better than scale up/down**:
- Scale up/down has been seen too many times by viewers, expectations are fixed
- Weight change is "internal fullness", like a balloon being inflated, not "being pushed closer"
- Variable fonts only became widespread post-2020, viewers subconsciously feel "modern"

**Limitation**: Must use a font that supports variable fonts (Inter/Roboto Flex/Recursive, etc.). Plain static fonts can only mimic it (switching between several fixed weights causes jumps).

### 5. **Corner Brand low-intensity continuous signature**

During the Gallery stage, there's a small `HUASHU · DESIGN` mark in the upper-left, 16% opacity color value, 12px font size, wide letter-spacing.

**Why add this**:
- After the Ripple burst, viewers easily "lose focus" and forget what they're looking at; the upper-left subtle mark helps anchor
- More sophisticated than a full-screen big logo — people who do branding know that brand signatures don't need to shout
- Still leaves attribution signal when GIFs are screenshotted and shared

**Rule**: Only appears in the middle (when the picture is busy), turned off at opening (don't obscure terminal), turned off at ending (brand reveal is the protagonist).

---

## Anti-examples: when NOT to use this choreography

**❌ Product demos (showing features)**: Gallery makes each card flash by, viewers can't remember any single feature. Switch to "single-screen focus + tooltip annotation".

**❌ Data-driven content**: Viewers need to read numbers; Gallery's fast pace doesn't give time. Switch to "data charts + item-by-item reveal".

**❌ Story narratives**: Gallery is a "parallel" structure; stories need "causation". Switch to keynote chapter transitions.

**❌ Only 3-5 assets**: Ripple density isn't enough, looks like a "patch". Switch to "static layout + highlight one by one".

**❌ Portrait (9:16)**: 3D tilt needs horizontal extension; portrait makes the tilt feel "crooked" rather than "unfolding".

---

## How to Determine If Your Task Suits This Choreography

Three quick checks:

**Step 1 · Asset count**: Count how many similar visual assets you have. < 15 → stop; 15-25 → make do; 25+ → use directly.

**Step 2 · Consistency test**: Place 4 random assets side-by-side. Do they look like "a set"? If not → unify the style first or change approach.

**Step 3 · Narrative match**: Are you expressing "Breadth × Depth" (quantity × quality)? Or "process" / "feature" / "story"? If it's not the former, don't force it.

If all three are yes, fork the v6 HTML directly, modify the `SLIDE_FILES` array and timeline to reuse. Modify the palette via `--bg / --accent / --ink`, swap the skin without changing the bones.

---

## Related References

- Complete technical workflow: [references/animations.md](animations.md) · [references/animation-best-practices.md](animation-best-practices.md)
- Animation export pipeline: [references/video-export.md](video-export.md)
- Audio configuration (BGM + SFX dual-track): [references/audio-design-rules.md](audio-design-rules.md)
- Apple gallery style horizontal reference: [references/apple-gallery-showcase.md](apple-gallery-showcase.md)
- Source HTML (v6 + audio integration): `www.huasheng.ai/huashu-design-hero/index.html`
