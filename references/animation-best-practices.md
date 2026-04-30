# Animation Best Practices · Positive Animation Design Grammar

> Based on deep deconstruction of Anthropic's three official product animations
> (Claude Design / Claude Code Desktop / Claude for Word), this document distills
> the "Anthropic-grade" animation design rules.
>
> Used together with `animation-pitfalls.md` (the avoidance checklist) — this file is
> "**what you should do**", pitfalls is "**what you should not do**". The two are
> orthogonal, both must be read.
>
> **Constraint statement**: This file only collects **motion logic and expressive style**,
> **does not introduce any brand-specific color values**.
> Color decisions go through the §1.a core asset protocol (extracted from the brand spec)
> or the "design direction advisor" (the color schemes of each of the 20 philosophies).
> This reference discusses "**how to move**", not "**what color**".

---

## §0 · Who You Are · Identity and Taste

> Before reading any technical rule below, read this section first. The rules
> **emerge from identity** — not the other way around.

### §0.1 Identity Anchor

**You are a motion designer who has studied the motion archives of Anthropic / Apple / Pentagram / Field.io.**

When making animations, you are not tweaking CSS transitions — you are using digital
elements to **simulate a physical world**, making the audience's subconscious believe
"these are objects with weight, with inertia, that overshoot."

You don't make PowerPoint-style animations. You don't make "fade in fade out" animations.
You make animations **that make people believe the screen is a space they can reach into**.

### §0.2 Core Beliefs (3 items)

1. **Animation is physics, not animation curves**
   `linear` is numbers, `expoOut` is objects. You believe pixels on screen deserve
   to be treated as "objects". Every choice of easing is answering the physics question
   "How heavy is this element? How large is the friction coefficient?"

2. **Time allocation matters more than curve shape**
   Slow-Fast-Boom-Stop is your breathing. **Even-rhythm animations are technical demos;
   rhythmic animations are narrative.**
   Slowing down at the right moment — matters more than using the right easing at the wrong moment.

3. **Yielding to the audience is harder than showing off**
   Pausing 0.5 seconds before a key result is **technique**, not compromise. **Letting
   the human brain have reaction time is the highest virtue of an animator.**
   AI defaults to making an animation with no pauses and information density at full —
   that's the novice. What you must do is restraint.

### §0.3 Taste Standards · What Is Beautiful

Your standards for judging "good" and "great" are as follows. Each has a **recognition
method** — when you see a candidate animation, use these questions to judge whether it
meets the bar, instead of mechanically comparing against 14 rules.

| Dimension of Beauty | Recognition Method (Audience Reaction) |
|---|---|
| **Physical weight** | At animation end, the element "**lands**" stably — not "**stops**" there. The audience subconsciously feels "this has weight" |
| **Yielding to the audience** | Before key information appears, there is a perceptible pause (≥300ms) — the audience has time to "**see**" before continuing |
| **Whitespace** | The ending is a sudden stop + hold, not fade to black. The last frame is clear, definite, with a sense of decision |
| **Restraint** | Throughout the entire piece there is only one place of "120% refinement", the other 80% is just right — **showing off everywhere is a cheap signal** |
| **Hand feel** | Arcs (not straight lines), irregular (not the mechanical rhythm of setInterval), with breath |
| **Respect** | Showing the tweak process, showing the bug fix — **don't hide work, don't give "magic"**. AI is a collaborator, not a magician |

### §0.4 Self-Check · Audience First Reaction Method

After making an animation, **what is the audience's first reaction after watching?** —
this is the only metric you should optimize.

| Audience Reaction | Rating | Diagnosis |
|---|---|---|
| "Looks pretty smooth" | good | Passing but featureless, you're making PowerPoint |
| "This animation is really fluid" | good+ | Technique is right, but not amazing |
| "This thing really looks like it's **floating up from the desktop**" | great | You touched physical weight |
| "This doesn't look like AI made it" | great+ | You touched the Anthropic threshold |
| "I want to **screenshot** this and post it on social" | great++ | You made the audience want to actively share |

**The difference between great and good is not technical correctness, it's taste judgment**.
Technically correct + right taste = great.
Technically correct + empty taste = good. Technically wrong = haven't entered the door.

### §0.5 The Relationship Between Identity and Rules

The technical rules in §1-§8 below are the **execution means** of this identity in
specific scenarios — not an independent rule list.

- Encountering a scenario the rules don't cover → return to §0, use **identity** to judge, don't blindly guess
- Encountering conflicts between rules → return to §0, use **taste standards** to judge which is more important
- Want to break a rule → first answer: "Does doing this fit which item in §0.3 of beauty?" If you can answer, break it; if not, don't.

Good. Continue reading.

---

## Overview · Animation Is Physics Unfolded in Three Layers

The root cause of cheapness in most AI-generated animations is — **they behave like "numbers"
not "objects"**.
Real-world objects have mass, inertia, elasticity, and overshoot. The root cause of the
"high-end feel" of Anthropic's three pieces is giving digital elements a set of
**physical world motion rules**.

This set of rules has 3 layers:

1. **Narrative rhythm layer**: Slow-Fast-Boom-Stop time allocation
2. **Motion curve layer**: Expo Out / Overshoot / Spring, reject linear
3. **Expression language layer**: Show the process, mouse arcs, Logo morph closing

---

## 1. Narrative Rhythm · Slow-Fast-Boom-Stop 5-Segment Structure

All three Anthropic pieces follow this structure without exception:

| Segment | Proportion | Rhythm | Function |
|---|---|---|---|
| **S1 Trigger** | ~15% | Slow | Give humans reaction time, establish realism |
| **S2 Generate** | ~15% | Medium | Visual wow point appears |
| **S3 Process** | ~40% | Fast | Show controllability/density/detail |
| **S4 Burst** | ~20% | Boom | Camera pulls back / 3D pop-out / multi-panel surge |
| **S5 Landing** | ~10% | Still | Brand Logo + sudden stop |

**Specific time mapping** (15-second animation as example):
S1 Trigger 2s · S2 Generate 2s · S3 Process 6s · S4 Burst 3s · S5 Landing 2s

**What is forbidden**:
- ❌ Even rhythm (same information density per second) — audience fatigue
- ❌ Sustained high density — no peak, no memory point
- ❌ Fade-out ending (fade out to transparent) — should be **sudden stop**

**Self-check**: Draw 5 thumbnails on paper, each representing the climax frame of a
segment. If the 5 images don't differ much, the rhythm hasn't been made.

---

## 2. Easing Philosophy · Reject linear, Embrace Physics

All animations in Anthropic's three pieces use bezier curves with a "damping feel".
The default cubic easeOut (`1-(1-t)³`) **isn't sharp enough** — startup isn't fast enough,
the pause isn't stable enough.

### Three Core Easings (built into animations.jsx)

```js
// 1. Expo Out · Quick start, slow brake (most commonly used, default main easing)
// CSS equivalent: cubic-bezier(0.16, 1, 0.3, 1)
Easing.expoOut(t) // = t === 1 ? 1 : 1 - Math.pow(2, -10 * t)

// 2. Overshoot · Bouncy toggle/button pop-out
// CSS equivalent: cubic-bezier(0.34, 1.56, 0.64, 1)
Easing.overshoot(t)

// 3. Spring physics · Geometric body return-to-place, natural settling
Easing.spring(t)
```

### Usage Mapping

| Scenario | Which Easing |
|---|---|
| Card rise-in / panel entrance / Terminal fade / focus overlay | **`expoOut`** (main easing, most commonly used) |
| Toggle switch / button pop-out / emphasized interaction | `overshoot` |
| Preview geometric body return / physical settling / UI element bounce | `spring` |
| Continuous motion (e.g., mouse trajectory interpolation) | `easeInOut` (preserve symmetry) |

### Counter-Intuitive Insight

Most product promo animations are **too fast and too hard**. `linear` makes digital
elements feel like machines, `easeOut` is the basic score, `expoOut` is the technical
root of the "high-end feel" — it gives digital elements a kind of **physical-world weight**.

---

## 3. Motion Language · 8 Common Principles

### 3.1 Background Color Is Not Pure Black or Pure White

None of Anthropic's three pieces use `#FFFFFF` or `#000000` as the main background.
**Neutral colors with a color temperature** (warm or cool) have a "paper / canvas / desktop"
material feel, weakening the machine feel.

**Specific color value decisions** go through the §1.a core asset protocol (extracted
from the brand spec) or the "design direction advisor"
(the background color schemes of each of the 20 philosophies). This reference does not
provide specific color values — that is a **brand decision**, not a motion rule.

### 3.2 Easing Is Never linear

See §2.

### 3.3 Slow-Fast-Boom-Stop Narrative

See §1.

### 3.4 Show the "Process" Not "Magic Results"

- Claude Design shows tweaking parameters, dragging sliders (not one-click perfect generation)
- Claude Code shows code errors + AI fixes (not one-shot success)
- Claude for Word shows the Redline red-strikethrough green-add editing process (not directly giving the final draft)

**Common subtext**: The product is a **collaborator, pair engineer, senior editor** —
not a one-click magician.
This precisely strikes professional users' pain points around "controllability" and "authenticity".

**Anti-AI slop**: AI defaults to making "magic one-click success" animations
(one-click generation → perfect result), which is the common-denominator convention.
**Do the opposite** — show the process, show tweaks, show bugs and fixes —
this is the source of brand recognition.

### 3.5 Mouse Trajectory Manually Drawn (Arc + Perlin Noise)

Real human mouse motion is not a straight line, it's "starting acceleration → arc →
deceleration correction → click".
A mouse trajectory that AI directly straight-line interpolates **has subconscious rejection**.

```js
// Quadratic bezier curve interpolation (start → control point → end)
function bezierQuadratic(p0, p1, p2, t) {
  const x = (1-t)*(1-t)*p0[0] + 2*(1-t)*t*p1[0] + t*t*p2[0];
  const y = (1-t)*(1-t)*p0[1] + 2*(1-t)*t*p1[1] + t*t*p2[1];
  return [x, y];
}

// Path: start → off-center midpoint → end (creates an arc)
const path = [[100, 100], [targetX - 200, targetY + 80], [targetX, targetY]];

// Then overlay tiny Perlin Noise (±2px) to create "hand tremor"
const jitterX = (simpleNoise(t * 10) - 0.5) * 4;
const jitterY = (simpleNoise(t * 10 + 100) - 0.5) * 4;
```

### 3.6 Logo "Morph Closing" (Morph)

The Logo entrance in Anthropic's three pieces is **not a simple fade-in**, it's
**morphed from the previous visual element**.

**Common pattern**: In the last 1-2 seconds, do Morph / Rotate / Converge, making the
entire narrative "collapse" onto the brand point.

**Low-cost implementation** (without true morphing):
Let the previous visual element "collapse" into a color block (scale → 0.1, translate
toward center), the color block then "expands" into the wordmark. The transition uses
150ms quick cut + motion blur (`filter: blur(6px)` → `0`).

```js
<Sprite start={13} end={14}>
  {/* Collapse: previous element scale 0.1, opacity stays, filter blur increases */}
  const scale = interpolate(t, [0, 0.5], [1, 0.1], Easing.expoOut);
  const blur = interpolate(t, [0, 0.5], [0, 6]);
</Sprite>
<Sprite start={13.5} end={15}>
  {/* Expand: Logo from color block center scale 0.1 → 1, blur 6 → 0 */}
  const scale = interpolate(t, [0, 0.6], [0.1, 1], Easing.overshoot);
  const blur = interpolate(t, [0, 0.6], [6, 0]);
</Sprite>
```

### 3.7 Serif + Sans-Serif Dual Fonts

- **Brand / narration**: Serif (has "academic feel / publication feel / taste")
- **UI / code / data**: Sans-serif + monospace

**Single font is wrong**. Serif gives "taste", sans-serif gives "function".

Specific font choices go through the brand spec (Display / Body / Mono three stacks
in brand-spec.md) or the design direction advisor's 20 philosophies. This reference
does not give specific fonts — that is a **brand decision**.

### 3.8 Focus Switch = Background Weakening + Foreground Sharpening + Flash Guidance

A focus switch is **not just** lowering opacity. The complete recipe is:

```js
// Filter combination for non-focus elements
tile.style.filter = `
  brightness(${1 - 0.5 * focusIntensity})
  saturate(${1 - 0.3 * focusIntensity})
  blur(${focusIntensity * 4}px)        // ← key: only with blur does it really "recede"
`;
tile.style.opacity = 0.4 + 0.6 * (1 - focusIntensity);

// After focus completes, do 150ms Flash highlight at focus position to guide gaze return
focusOverlay.animate([
  { background: 'rgba(255,255,255,0.3)' },
  { background: 'rgba(255,255,255,0)' }
], { duration: 150, easing: 'ease-out' });
```

**Why blur is required**: With only opacity + brightness, elements outside focus are still
"sharp", and visually there's no "receding to the back" effect. blur(4-8px) makes
the non-focus truly recede a layer of depth of field.

---

## 4. Specific Motion Techniques (Code Snippets You Can Copy Directly)

### 4.1 FLIP / Shared Element Transition

A button "expands" into an input field — **not** the button disappearing + a new panel
appearing. The core is **the same DOM element** transitioning between two states, not
two elements cross-fading.

```jsx
// Use Framer Motion layoutId
<motion.div layoutId="design-button">Design</motion.div>
// ↓ After click, same layoutId
<motion.div layoutId="design-button">
  <input placeholder="Describe your design..." />
</motion.div>
```

For native implementation reference https://aerotwist.com/blog/flip-your-animations/

### 4.2 "Breathing" Expansion (width→height)

Panel expansion is **not pulling width and height simultaneously**, but:
- First 40% of time: only pull width (keep height small)
- Last 60% of time: width holds, push out height

This simulates the physical world feeling of "expand first, then fill with water".

```js
const widthT = interpolate(t, [0, 0.4], [0, 1], Easing.expoOut);
const heightT = interpolate(t, [0.3, 1], [0, 1], Easing.expoOut);
style.width = `${widthT * targetW}px`;
style.height = `${heightT * targetH}px`;
```

### 4.3 Staggered Fade-up (30ms stagger)

When table rows, card columns, or list items enter, **each element delays 30ms**,
`translateY` returns from 10px to 0.

```js
rows.forEach((row, i) => {
  const localT = Math.max(0, t - i * 0.03);  // 30ms stagger
  row.style.opacity = interpolate(localT, [0, 0.3], [0, 1], Easing.expoOut);
  row.style.transform = `translateY(${
    interpolate(localT, [0, 0.3], [10, 0], Easing.expoOut)
  }px)`;
});
```

### 4.4 Non-Linear Breathing · Hover 0.5s Before Key Result

The machine executes fast and continuously, but **hover 0.5 seconds before the key result
appears**, letting the audience's brain have reaction time.

```jsx
// Typical scenario: AI generation done → hover 0.5s → result appears
<Sprite start={8} end={8.5}>
  {/* 0.5s pause — nothing moves, let the audience stare at the loading state */}
  <LoadingState />
</Sprite>
<Sprite start={8.5} end={10}>
  <ResultAppear />
</Sprite>
```

**Counter-example**: After AI generation completes, immediately seamlessly cut to result —
the audience has no reaction time, information is lost.

### 4.5 Chunk Reveal · Simulating Token Streaming

For AI-generated text **don't use `setInterval` to pop out single characters** (like old
movie subtitles); use **chunk reveal** — 2-5 characters appearing at once, with irregular
intervals, simulating real token streaming output.

```js
// Split by chunks, not by characters
const chunks = text.split(/(\s+|,\s*|\.\s*|;\s*)/);  // split by word + punctuation
let i = 0;
function reveal() {
  if (i >= chunks.length) return;
  element.textContent += chunks[i++];
  const delay = 40 + Math.random() * 80;  // irregular 40-120ms
  setTimeout(reveal, delay);
}
reveal();
```

### 4.6 Anticipation → Action → Follow-through

3 of the Disney 12 principles. Anthropic uses them very explicitly:

- **Anticipation**: Before the action begins, there's a small reverse motion (button slightly shrinks then pops out)
- **Action**: The main action itself
- **Follow-through**: After the action ends, there's an aftertaste (card slightly bounces after settling)

```js
// Complete three segments of card entrance
const anticip = interpolate(t, [0, 0.2], [1, 0.95], Easing.easeIn);     // anticipation
const action  = interpolate(t, [0.2, 0.7], [0.95, 1.05], Easing.expoOut); // main action
const settle  = interpolate(t, [0.7, 1], [1.05, 1], Easing.spring);       // bounce back
// Final scale = product of three segments or piecewise application
```

**Counter-example**: An animation with only Action and no Anticipation + Follow-through
looks like "PowerPoint animation".

### 4.7 3D Perspective + translateZ Layering

To get the temperament of "tilted 3D + floating cards", give the container perspective,
and give individual elements different translateZ values:

```css
.stage-wrap {
  perspective: 2400px;
  perspective-origin: 50% 30%;  /* Line of sight slightly looking down */
}
.card-grid {
  transform-style: preserve-3d;
  transform: rotateX(8deg) rotateY(-4deg);  /* Golden ratio */
}
.card:nth-child(3n) { transform: translateZ(30px); }
.card:nth-child(5n) { transform: translateZ(-20px); }
.card:nth-child(7n) { transform: translateZ(60px); }
```

**Why rotateX 8° / rotateY -4° is the golden ratio**:
- Greater than 10° → element distortion is too strong, looks like "falling over"
- Less than 5° → looks like "skew" rather than "perspective"
- The asymmetric ratio of 8° × -4° simulates the natural angle of "camera looking down from the upper left of the desk"

### 4.8 Diagonal Pan · Move XY Simultaneously

Camera motion is not pure up-down or pure left-right, but **moves XY simultaneously**
to simulate diagonal movement:

```js
const panX = Math.sin(flowT * 0.22) * 40;
const panY = Math.sin(flowT * 0.35) * 30;
stage.style.transform = `
  translate(-50%, -50%)
  rotateX(8deg) rotateY(-4deg)
  translate3d(${panX}px, ${panY}px, 0)
`;
```

**Key**: X and Y have different frequencies (0.22 vs 0.35), avoiding regularization of the Lissajous loop.

---

## 5. Scenario Recipes (Three Narrative Templates)

The three reference videos correspond to three product personalities. **Pick the one
most fitting your product**, don't mix and match.

### Recipe A · Apple Keynote Drama (Claude Design type)

**Suitable for**: Major version releases, hero animations, visual wow priority
**Rhythm**: Slow-Fast-Boom-Stop strong arc
**Easing**: `expoOut` throughout + a small amount of `overshoot`
**SFX density**: High (~0.4/s), SFX pitch tuned to BGM scale
**BGM**: IDM / minimal tech electronic, calm + precise
**Closing**: Camera pulls back rapidly → drop → Logo morph → ethereal single tone → sudden stop

### Recipe B · One-Take Tool Style (Claude Code type)

**Suitable for**: Developer tools, productivity Apps, flow scenarios
**Rhythm**: Sustained stable flow, no obvious peaks
**Easing**: `spring` physics + `expoOut`
**SFX density**: **0** (purely BGM-driven editing rhythm)
**BGM**: Lo-fi Hip-hop / Boom-bap, 85-90 BPM
**Core technique**: Key UI actions step on the BGM kick/snare transients — "**music groove is the interaction sound effect**"

### Recipe C · Office Efficiency Narrative (Claude for Word type)

**Suitable for**: Enterprise software, document/spreadsheet/calendar types, professional feel priority
**Rhythm**: Multi-scene hard cuts + Dolly In/Out
**Easing**: `overshoot` (toggle) + `expoOut` (panel)
**SFX density**: Medium (~0.3/s), UI click as main
**BGM**: Jazzy Instrumental, minor key, BPM 90-95
**Core highlight**: A certain scene must have a "full-piece highlight" — 3D pop-out / floating off the plane

---

## 6. Counter-Examples · Doing It This Way Is AI Slop

| Anti-pattern | Why wrong | Correct approach |
|---|---|---|
| `transition: all 0.3s ease` | `ease` is a relative of linear, all elements at the same speed | Use `expoOut` + per-element stagger |
| All entrances `opacity 0→1` | No motion direction sense | Pair with `translateY 10→0` + Anticipation |
| Logo fade-in | No narrative closure feel | Morph / Converge / collapse-expand |
| Mouse moves in straight line | Subconscious machine feel | Bezier arc + Perlin Noise |
| Typing single char popping (setInterval) | Like old movie subtitles | Chunk Reveal, random intervals |
| Key result with no hover | Audience has no reaction time | 0.5s hover before the result |
| Focus switch only changes opacity | Non-focus elements still sharp | opacity + brightness + **blur** |
| Pure black background / pure white background | Cyber feel / reflection fatigue | Neutral color with color temperature (go through brand spec) |
| All animations equally fast | No rhythm | Slow-Fast-Boom-Stop |
| Fade out ending | No decision feel | Sudden stop (hold last frame) |

---

## 7. Self-Check Checklist (60 seconds before animation delivery)

- [ ] Is the narrative structure Slow-Fast-Boom-Stop, not even rhythm?
- [ ] Is the default easing `expoOut`, not `easeOut` or `linear`?
- [ ] Did toggle / button pop-out use `overshoot`?
- [ ] Do card / list entrances have 30ms stagger?
- [ ] Is there a 0.5s hover before key results?
- [ ] Does typing use Chunk Reveal, not setInterval single chars?
- [ ] Did focus switch add blur (not just opacity)?
- [ ] Is the Logo morph closing (Morph), not fade-in?
- [ ] Is the background not pure black / pure white (with color temperature)?
- [ ] Does text have serif + sans-serif hierarchy?
- [ ] Is the ending a sudden stop, not gradual fade?
- [ ] (If there's a mouse) Is the mouse trajectory an arc, not a straight line?
- [ ] Does SFX density match the product personality (see recipes A/B/C)?
- [ ] Is there a 6-8dB loudness difference between BGM and SFX? (see `audio-design-rules.md`)

---

## 8. Relationship with Other References

| reference | Positioning | Relationship |
|---|---|---|
| `animation-pitfalls.md` | Technical avoidance (16 items) | "**Don't do this**" · the opposite of this file |
| `animations.md` | Stage/Sprite engine usage | The basis of **how** animations are written |
| `audio-design-rules.md` | Dual-track audio rules | Rules for **adding audio** to animations |
| `sfx-library.md` | 37 SFX list | Sound effect **asset library** |
| `apple-gallery-showcase.md` | Apple Gallery showcase style | A specialized topic on a specific motion style |
| **This file** | Positive motion design grammar | "**You should do this**" |

**Call sequence**:
1. First look at the four position questions in Step 3 of the SKILL.md workflow (decide narrative role and visual temperature)
2. After choosing a direction, read this file to determine the **motion language** (recipe A/B/C)
3. When writing code, refer to `animations.md` and `animation-pitfalls.md`
4. When exporting video, go through `audio-design-rules.md` + `sfx-library.md`

---

## Appendix · Source of Materials for This File

- Anthropic official animation breakdown: `reference-animations/BEST-PRACTICES.md` in Hua Shu's project directory
- Anthropic audio breakdown: same directory `AUDIO-BEST-PRACTICES.md`
- 3 reference videos: `ref-{1,2,3}.mp4` + corresponding `gemini-ref-*.md` / `audio-ref-*.md`
- **Strict filtering**: This reference does not include any specific brand color values, font names, or product names.
  Color/font decisions go through the §1.a core asset protocol or the 20 design philosophies.
