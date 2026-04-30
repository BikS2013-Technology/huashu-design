# Cinematic Patterns · Best Practice for Workflow Demos

> 5 key patterns for upgrading from "PowerPoint animations" to "keynote-event-grade cinematic" pieces.
> Distilled from two cinematic demos in the 2026-04 "Let's Talk Skills" deck (Nuwa workflow + Darwin workflow), tested and reproducible.

---

## 0 · What problem does this document solve

When you need to make a "demo animation showing a workflow" (typical scenarios: skill workflows, product onboarding, API call flows, agent task execution), there are two common approaches:

| Paradigm | What it looks like | Consequence |
|---|---|---|
| **PowerPoint animation** (bad) | step 1 fade in → step 2 fade in → step 3 fade in, 4 boxes laid out on the same screen | Viewers feel "this is just a PPT with fade effects added", no wow moment |
| **Cinematic** (good) | scene-based, focus on one thing at a time, scenes transition via dissolve / focus pull / morph | Viewers feel "this is a product launch event clip", they want to screenshot and share it |

The root of the difference is **not animation technique**, it's the **narrative paradigm**. This document explains how to upgrade from the former to the latter.

---

## 1 · Five core patterns

### Pattern A · Dashboard + Cinematic Overlay two-layer structure

**Problem**: Pure cinematic defaults to a black screen + a ▶ button. If a user lands on this page and doesn't click, they see nothing.

**Solution**:
```
DEFAULT state (always visible): full static workflow dashboard
  └── Viewer instantly sees how this skill / workflow runs

POINT ▶ trigger (overlay floats up): 22-second cinematic
  └── After it finishes, automatically fades back to DEFAULT

```

**Implementation points**:
- `.dash` is visible by default, `.cinema` defaults to `opacity: 0; pointer-events: none`
- `.play-cta` is a small gold button in the bottom-right corner (not a giant central overlay)
- Click → `cinema.classList.add('show')` + `dash.classList.add('hide')`
- Use `requestAnimationFrame` to run once (not loop), and call `endCinematic()` to reverse state when done

**Anti-pattern**: Default = giant central ▶ overlay covering everything, page is blank before clicking.

---

### Pattern B · Scene-based, NOT Step-based

**Problem**: Splitting the animation into "step 1 shows → step 2 shows → ..." is PowerPoint thinking.

**Solution**: Split into 5 scenes, each scene is an **independent shot**, full-screen, focused on only one thing:

| Scene type | Responsibility | Duration |
|---|---|---|
| 1 · Invoke | User input triggers it (terminal typewriter) | 3-4s |
| 2 · Process | Visualization of the core workflow (unique visual language) | 5-6s |
| 3 · Result/Insight | The key extracted artifact (visualized) | 4-5s |
| 4 · Output | The actual produced artifact (file / diff / number) | 3-4s |
| 5 · Hero Reveal | Closing hero moment (large text + value proposition) | 4-5s |

**Total duration ≈ 22 seconds** — this is the tested golden length:
- Shorter than 18 seconds: PMs haven't gotten into the state before it ends
- Longer than 25 seconds: lose patience
- 22 seconds is just enough to "hook → unfold → conclude → leave an impression"

**Implementation points**:
- `T = { DURATION: 22.0, s1_in: [0, 0.7], s2_in: [3.8, 4.6], ... }` global timeline
- A single `requestAnimationFrame(render)` runs all opacity / transform calculations for all scenes
- Don't use setTimeout chains (easy to break, hard to debug)
- Easing must use `expoOut` / `easeOut` / cubic-bezier, **never linear**

---

### Pattern C · Each demo's visual language must be independent

**Problem**: After making the first cinematic, you get lazy and reuse the same template for the second one (same orbit + pentagon + typewriter + hero large text), only changing the copy.

**Consequence**: Viewers notice that the two skills "look identical", which is equivalent to saying "these two skills are no different".

**Solution**: The core metaphor of each workflow is different, so the visual language must be different.

**Comparison case**:

| Dimension | Nuwa (distilling people) | Darwin (optimizing skills) |
|---|---|---|
| Core metaphor | Collect → refine → write | Loop → evaluate → ratchet |
| Visual motion | Float / radiate / pentagon | Loop / ascend / contrast |
| Scene 2 | 3D Orbit · 8 archives floating in perspective ellipse | Spin Loop · token runs 5 laps along 6-node ring |
| Scene 3 | Pentagon · 5 tokens radiating from center | v1 vs v5 · side-by-side diff (red version vs gold version) |
| Scene 4 | SKILL.md typewriter | Hill-Climb · full-screen curve drawing |
| Scene 5 hero | "21 minutes" serif italic large text | Spinning gear ⚙ + "KEPT +1.1" gold tag |

**Test criterion**: Cover the copy and look only at the visuals — can you tell which demo this is? If not, you got lazy.

---

### Pattern D · Use AI-generated real assets, not emojis or hand-drawn SVGs

**Problem**: 3D orbit / gallery needs asset fragments to float around. Emojis (📚🎤) are ugly and unbranded; hand-drawn SVG book spines never look like real books.

**Solution**: Use `huashu-gpt-image` to render a 4×2 grid mega-image (8 themed objects · white background · 60px breathing space · unified style), then use `extract_grid.py --mode bbox` to extract 8 separate transparent PNGs.

**Prompt key points** (detailed prompt patterns in the `huashu-gpt-image` skill):
- IP anchoring ("1960s Caltech archive aesthetic" / "Hearthstone-style consistent treatment")
- White background (easy to extract; gray backgrounds have nice ambience but transparent extraction is hard)
- 4×2 not 5×5 (avoid last-row compression bug)
- Persona finishing ("You are a Wired magazine curator preparing an exhibition photo")

**Anti-pattern**: Using emojis as icons, using CSS silhouettes instead of product images.

---

### Pattern E · BGM + SFX dual-track system

**Problem**: Animation without sound — viewers subconsciously feel "this thing looks like a cheap demo".

**Solution**: BGM long-form audio + 11 SFX cues.

**Universal SFX cue recipe** (applicable to workflow demos):

| Time point | SFX | Trigger scene |
|---|---|---|
| 0.10s | whoosh | Terminal rises from below |
| 3.0s | enter | Typewriter completes, press enter |
| 4.0s | slide-in | Scene 2 elements enter |
| 5-9s × 5 times | sparkle | Key process nodes (each generation / each token / each data point) |
| 14s | click | Switch to output scene |
| 17.8s | logo-reveal | Hero reveal moment |
| typewriter | type | Triggered every 2 characters (don't make density too high) |

**Frequency band isolation**: BGM volume 0.32 (low-frequency floor), SFX volume 0.55 (mid-high frequency punch), sparkle 0.7 (needs to stand out), logo-reveal 0.85 (strongest hero moment).

**User control**:
- Must have a ▶ start overlay (browser autoplay restrictions)
- Small mute button in upper-right (user can toggle silence anytime)
- Don't make it "force play when you flip to this page"

---

## 2 · Static Dashboard design points

The Dashboard is Layer 1 of the two-layer structure. Even if the PM doesn't click ▶, they can understand this skill.

**Layout**: 3-column grid (or 1 large + 2 small), each panel solves one problem:

| Panel type | What problem it solves | Case |
|---|---|---|
| **Pipeline / Flow Diagram** | "What is the workflow of this skill?" | Nuwa 4-stage pipeline · Darwin autoresearch loop |
| **Snapshot / State** | "What does the actual generated data look like?" | Darwin 8-dimension rubric snapshot |
| **Trajectory / Evolution** | "How does it change after multiple runs?" | Darwin 5-generation hill-climb curve |
| **Examples / Gallery** | "What has already been produced?" | Nuwa 21 personas gallery |
| **Strip · Example I/O** | "Input what → output what" | Nuwa example strip: `› nuwa distill feynman → feynman.skill (21 min)` |

**Key constraints**:
- Information density should be sufficient (every panel must carry differentiated information)
- But don't stuff with data slop (every number must be meaningful)
- Color scheme consistent with the cinematic (same color family, easy to switch without jarring)

---

## 3 · Debugging and development tools

Any long animation must be paired with three dev tools, otherwise debugging will explode.

### Tool 1 · `?seek=N` freeze to second N

```js
const seek = parseFloat(params.get('seek'));
if (!isNaN(seek)) {
  started = true; muted = true;
  frozenT = seek;  // render() uses this t instead of elapsed
  cinema.classList.add('show'); dash.classList.add('hide');
}

// Inside render():
let t = frozenT !== null ? frozenT : (elapsed % T.DURATION);
```

Usage: `http://.../slide.html?seek=12` directly view the frame at second 12, no need to wait for playback.

### Tool 2 · `?autoplay=1` skip the ▶ overlay

Convenient for playwright automatic screenshot testing, also for force-starting when embedded in iframes.

### Tool 3 · Manual REPLAY button

Small button in the upper-right, allows users/debuggers to replay any number of times. CSS:

```css
.replay{position:absolute;top:18px;right:18px;background:rgba(212,165,116,0.1);
  border:1px solid rgba(212,165,116,0.3);color:#D4A574;
  font-family:monospace;font-size:10px;letter-spacing:.28em;text-transform:uppercase;
  padding:6px 12px;border-radius:1px;cursor:pointer;backdrop-filter:blur(6px);z-index:6}
```

---

## 4 · iframe embedding pitfalls (if cinematic is embedded in a deck)

### Pitfall 1 · Parent window's click zone intercepts iframe-internal buttons

If the deck index.html adds "left/right 22vw transparent click zones for paging", these will **cover the ▶ play button inside the iframe** — user clicks the button but it gets swallowed as "next page".

**Fix**: Add `top: 12vh; bottom: 25vh` to the click zone, leaving the top and bottom 25% non-intercepting, so both the central ▶ and the bottom-right ▶ inside the iframe are clickable.

### Pitfall 2 · After iframe steals focus, keyboard events are lost

After the user clicks the iframe, focus is inside the iframe, and the parent window's ←/→ keyboard events aren't received.

**Fix**:
```js
iframe.addEventListener('load', () => {
  // Inject keyboard forwarder
  const doc = iframe.contentDocument;
  doc.addEventListener('keydown', (e) => {
    window.dispatchEvent(new KeyboardEvent('keydown', { key: e.key, ... }));
  });
  // After clicking, pull focus back to parent window
  doc.addEventListener('click', () => setTimeout(() => window.focus(), 0));
});
```

### Pitfall 3 · file:// vs https:// behavior differences

A cinematic that tested fine locally on file:// may break after deployment, because:
- Under file://, iframe contentDocument is same-origin
- Under https:// it's also same-origin (if same host), but audio autoplay restrictions are stricter

**Fix**:
- Before deployment, use `python3 -m http.server` to start a local HTTP server and test it
- BGM must wait for user click ▶ before `bgm.play()` is called, don't play immediately on page-load

---

## 5 · Anti-pattern quick reference

| ❌ Anti-pattern | ✅ Right pattern |
|---|---|
| Default = black screen ▶ overlay | Default = static dashboard, ▶ is auxiliary |
| 4 steps fading in side-by-side on same screen | 5 scenes full-screen switching, each scene focuses on one thing |
| Reuse template, swap copy for different demos | Each demo has independent visual language (cover the copy and they're distinguishable) |
| Emoji / hand-drawn SVG as assets | gpt-image-2 mega-image + extract_grid extraction |
| No BGM no SFX | BGM + 11 SFX cues dual-track |
| Use setTimeout chain to schedule | requestAnimationFrame + global timeline T object |
| linear animation | Expo / cubic-bezier easing |
| No dev tools | `?seek=N` + `?autoplay=1` + REPLAY button |
| Buttons inside iframe swallowed by parent click zone | Click zone adds top/bottom margin to make room for buttons |

---

## 6 · Time budget

Following this set of patterns, a complete cinematic demo (with dashboard):

| Task | Time |
|---|---|
| Design 5-scene narrative + visual language | 30 minutes (be careful, decides independence) |
| Dashboard static layout + content | 1 hour |
| Cinematic 5 scenes implementation | 1.5 hours |
| Audio cues timing tuning + replay button | 30 minutes |
| Playwright screenshot validation of 5 key moments | 15 minutes |
| **Single demo total** | **3-4 hours** |

The second demo reuses the framework but **the visual language must be independent**, taking about 2-3 hours.
