# Animations: Timeline-Driven Animation Engine

Read this when building animation / motion-design HTML. Principles, usage, and typical patterns.

## Core Pattern: Stage + Sprite

Our animation system (`assets/animations.jsx`) provides a timeline-driven engine:

- **`<Stage>`**: container for the entire animation. Automatically provides auto-scale (fit-viewport) + scrubber + play/pause/loop controls
- **`<Sprite start end>`**: a time slice. A Sprite only renders during the `start`–`end` window. Inside it, you can read its local progress `t` (0→1) via the `useSprite()` hook
- **`useTime()`**: reads the current global time (in seconds)
- **`Easing.easeInOut` / `Easing.easeOut` / ...**: easing functions
- **`interpolate(t, from, to, easing?)`**: interpolate based on `t`

This pattern borrows from Remotion / After Effects but is lightweight and zero-dependency.

## Getting Started

```html
<script type="text/babel" src="animations.jsx"></script>
<script type="text/babel">
  const { Stage, Sprite, useTime, useSprite, Easing, interpolate } = window.Animations;

  function Title() {
    const { t } = useSprite();  // local progress 0→1
    const opacity = interpolate(t, [0, 1], [0, 1], Easing.easeOut);
    const y = interpolate(t, [0, 1], [40, 0], Easing.easeOut);
    return (
      <h1 style={{ 
        opacity, 
        transform: `translateY(${y}px)`,
        fontSize: 120,
        fontWeight: 900,
      }}>
        Hello.
      </h1>
    );
  }

  function Scene() {
    return (
      <Stage duration={10}>  {/* 10-second animation */}
        <Sprite start={0} end={3}>
          <Title />
        </Sprite>
        <Sprite start={2} end={5}>
          <SubTitle />
        </Sprite>
        {/* ... */}
      </Stage>
    );
  }

  const root = ReactDOM.createRoot(document.getElementById('root'));
  root.render(<Scene />);
</script>
```

## Common Animation Patterns

### 1. Fade In / Fade Out

```jsx
function FadeIn({ children }) {
  const { t } = useSprite();
  const opacity = interpolate(t, [0, 0.3], [0, 1], Easing.easeOut);
  return <div style={{ opacity }}>{children}</div>;
}
```

**Note on the range**: `[0, 0.3]` means the fade-in completes during the first 30% of the sprite's time, then opacity stays at 1 afterward.

### 2. Slide In

```jsx
function SlideIn({ children, from = 'left' }) {
  const { t } = useSprite();
  const progress = interpolate(t, [0, 0.4], [0, 1], Easing.easeOut);
  const offset = (1 - progress) * 100;
  const directions = {
    left: `translateX(-${offset}px)`,
    right: `translateX(${offset}px)`,
    top: `translateY(-${offset}px)`,
    bottom: `translateY(${offset}px)`,
  };
  return (
    <div style={{
      transform: directions[from],
      opacity: progress,
    }}>
      {children}
    </div>
  );
}
```

### 3. Character-by-Character Typewriter

```jsx
function Typewriter({ text }) {
  const { t } = useSprite();
  const charCount = Math.floor(text.length * Math.min(t * 2, 1));
  return <span>{text.slice(0, charCount)}</span>;
}
```

### 4. Number Count-Up

```jsx
function CountUp({ from = 0, to = 100, duration = 0.6 }) {
  const { t } = useSprite();
  const progress = interpolate(t, [0, duration], [0, 1], Easing.easeOut);
  const value = Math.floor(from + (to - from) * progress);
  return <span>{value.toLocaleString()}</span>;
}
```

### 5. Phased Explanation (typical instructional animation)

```jsx
function Scene() {
  return (
    <Stage duration={20}>
      {/* Phase 1: present the problem */}
      <Sprite start={0} end={4}>
        <Problem />
      </Sprite>

      {/* Phase 2: show the approach */}
      <Sprite start={4} end={10}>
        <Approach />
      </Sprite>

      {/* Phase 3: show the result */}
      <Sprite start={10} end={16}>
        <Result />
      </Sprite>

      {/* Caption visible throughout */}
      <Sprite start={0} end={20}>
        <Caption />
      </Sprite>
    </Stage>
  );
}
```

## Easing Functions

Preset easing curves:

| Easing | Characteristic | Used for |
|--------|----------------|----------|
| `linear` | Constant speed | Scrolling captions, continuous animations |
| `easeIn` | Slow → fast | Exit / disappearance |
| `easeOut` | Fast → slow | Entrance / appearance |
| `easeInOut` | Slow → fast → slow | Position changes |
| **`expoOut`** ⭐ | **Exponential ease-out** | **Anthropic-grade primary easing** (sense of physical weight) |
| **`overshoot`** ⭐ | **Elastic bounce** | **Toggle / button pop / emphasis interactions** |
| `spring` | Spring physics | Interaction feedback, geometry settling |
| `anticipation` | Reverse first, then forward | Emphasized actions |

**Default primary easing is `expoOut`** (not `easeOut`) — see `animation-best-practices.md` §2.
Entrance uses `expoOut`, exit uses `easeIn`, toggle uses `overshoot` — the foundational rules of Anthropic-grade animation.

## Pacing and Duration Guide

### Micro-interactions (0.1–0.3 seconds)
- Button hover
- Card expand
- Tooltip appearance

### UI transitions (0.3–0.8 seconds)
- Page switch
- Modal appearance
- List item insertion

### Narrative animations (2–10 seconds per segment)
- One phase of a concept explanation
- Reveal of a data chart
- Scene transition

### A single narrative animation segment should not exceed 10 seconds
Human attention is finite. Take 10 seconds to tell one thing; once told, move on.

## The Order to Think About When Designing Animation

### 1. Content/story first, animation second

**Wrong**: deciding you want a fancy animation first, then stuffing content in
**Right**: figure out what information you need to convey first, then use animation to serve that information

Animation is **signal**, not **decoration**. A fade-in says "this is important, look here" — if everything fades in, the signal is destroyed.

### 2. Write the timeline scene by scene

```
0:00 - 0:03   Problem appears (fade in)
0:03 - 0:06   Problem zooms / expands (zoom+pan)
0:06 - 0:09   Solution appears (slide in from right)
0:09 - 0:12   Solution explained (typewriter)
0:12 - 0:15   Result demo (counter up + chart reveal)
0:15 - 0:18   One-line summary (static, read for 3 sec)
0:18 - 0:20   CTA or fade out
```

Write the timeline first, then the components.

### 3. Assets first

Prepare the images/icons/fonts the animation will use **before** you start. Don't pause halfway through to hunt for assets — it breaks your rhythm.

## Common Issues

**Animation stutters**
→ Mostly layout thrashing. Use `transform` and `opacity`; don't animate `top`/`left`/`width`/`height`/`margin`. Browsers GPU-accelerate `transform`.

**Animation is too fast, can't see it clearly**
→ A reader needs 100–150 ms per Chinese character, 300–500 ms per word. If you're telling a story with text, leave at least 3 seconds per sentence.

**Animation is too slow, audience gets bored**
→ Interesting visual changes should be densely packed. A static frame past 5 seconds becomes dull.

**Multiple animations interfere with each other**
→ Use CSS `will-change: transform` to tell the browser the element will animate, reducing reflow.

**Recording it as video**
→ Use the skill's built-in toolchain (one command produces three formats): see `video-export.md`
- `scripts/render-video.js` — HTML → 25fps MP4 (Playwright + ffmpeg)
- `scripts/convert-formats.sh` — 25fps MP4 → 60fps MP4 + optimized GIF
- Want more precise frame rendering? Make `render(t)` a pure function — see `animation-pitfalls.md` rule 5

## Pairing with Video Tools

This skill produces **HTML animation** (running in the browser). If the final deliverable is video material:

- **Short animations / concept demos**: build HTML animation here → screen capture
- **Long-form video / narrative**: this skill focuses on HTML animation; for long video, use an AI video-generation skill or professional video software
- **Motion graphics**: professional tools like After Effects / Motion Canvas are a better fit

## On Popmotion and similar libraries

If you really need physics-based animation (spring, decay, keyframes with precise timing), our engine can't handle it — fall back to Popmotion:

```html
<script src="https://unpkg.com/popmotion@11.0.5/dist/popmotion.min.js"></script>
```

But **try our engine first**. It's enough for 90% of cases.
