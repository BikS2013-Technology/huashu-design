# Animation Pitfalls: Bugs Encountered Doing HTML Animation and How to Avoid Them

The most common bugs hit when doing animation, and how to avoid them. Each rule comes from a real failure case.

Reading this before writing animation can save a round of iteration.

## 1. Stacking Layout — `position: relative` Is a Default Obligation

**Pitfall**: A sentence-wrap element wrapped 3 bracket-layers (`position: absolute`). Without setting `position: relative` on sentence-wrap, the absolute brackets used `.canvas` as their coordinate system and drifted 200px below the screen.

**Rules**:
- Any container with `position: absolute` children **must** explicitly have `position: relative`
- Even if visually no "offset" is needed, write `position: relative` as the coordinate system anchor
- If you're writing `.parent { ... }` and its children include `.child { position: absolute }`, instinctively add relative to parent

**Quick check**: For each occurrence of `position: absolute`, count up the ancestors and confirm the nearest positioned ancestor is the coordinate system you *want*.

## 2. Character Trap — Don't Rely on Rare Unicode

**Pitfall**: Wanted to use `␣` (U+2423 OPEN BOX) to visualize "space token". Neither Noto Serif SC nor Cormorant Garamond has this glyph; rendered as blank/tofu, viewers couldn't see it at all.

**Rules**:
- **Every character that appears in the animation must exist in your chosen font**
- Common rare character blacklist: `␣ ␀ ␐ ␋ ␨ ↩ ⏎ ⌘ ⌥ ⌃ ⇧ ␦ ␖ ␛`
- To express "space / return / tab" meta-characters, use **CSS-constructed semantic boxes**:
  ```html
  <span class="space-key">Space</span>
  ```
  ```css
  .space-key {
    display: inline-flex;
    padding: 4px 14px;
    border: 1.5px solid var(--accent);
    border-radius: 4px;
    font-family: monospace;
    font-size: 0.3em;
    letter-spacing: 0.2em;
    text-transform: uppercase;
  }
  ```
- Emojis must also be verified: some emojis fall back to gray squares in fonts other than Noto Emoji; best to use `emoji` font-family or SVG

## 3. Data-Driven Grid/Flex Templates

**Pitfall**: Code has `const N = 6` tokens, but CSS hardcodes `grid-template-columns: 80px repeat(5, 1fr)`. Result: the 6th token has no column, the entire matrix misaligned.

**Rules**:
- When count comes from a JS array (`TOKENS.length`), the CSS template should also be data-driven
- Option A: inject from JS via CSS variables
  ```js
  el.style.setProperty('--cols', N);
  ```
  ```css
  .grid { grid-template-columns: 80px repeat(var(--cols), 1fr); }
  ```
- Option B: use `grid-auto-flow: column` to let the browser auto-extend
- **Prohibit "fixed number + JS constant" combinations** — when N changes, CSS won't sync

## 4. Transition Discontinuity — Scene Switches Must Be Continuous

**Pitfall**: Between zoom1 (13-19s) → zoom2 (19.2-23s), main sentence already hidden, zoom1 fade out (0.6s) + zoom2 fade in (0.6s) + stagger delay (0.2s+) = about 1 second of pure blank screen. Viewers thought the animation got stuck.

**Rules**:
- When switching scenes consecutively, fade out and fade in should **cross-overlap**, not have the previous one completely disappear before starting the next
  ```js
  // Bad:
  if (t >= 19) hideZoom('zoom1');      // 19.0s out
  if (t >= 19.4) showZoom('zoom2');    // 19.4s in → 0.4s blank in between

  // Good:
  if (t >= 18.6) hideZoom('zoom1');    // Start fade out 0.4s earlier
  if (t >= 18.6) showZoom('zoom2');    // Fade in simultaneously (cross-fade)
  ```
- Or use an "anchor element" (like the main sentence) as visual connection between scenes; it briefly reappears during zoom switching
- Calculate CSS transition duration carefully to avoid triggering the next one before the transition finishes

## 5. Pure Render Principle — Animation State Should Be Seekable

**Pitfall**: Used `setTimeout` + `fireOnce(key, fn)` to chain-trigger animation states. Normal playback is fine, but for frame-by-frame recording / seeking to any time point, previously executed setTimeouts can't "go back to the past".

**Rules**:
- The `render(t)` function ideally is a **pure function**: given t, output a unique DOM state
- If side effects must be used (like class toggles), use a `fired` set with explicit reset:
  ```js
  const fired = new Set();
  function fireOnce(key, fn) { if (!fired.has(key)) { fired.add(key); fn(); } }
  function reset() { fired.clear(); /* clear all .show classes */ }
  ```
- Expose `window.__seek(t)` for Playwright / debugging:
  ```js
  window.__seek = (t) => { reset(); render(t); };
  ```
- Animation-related setTimeouts shouldn't span >1 second, otherwise seek-back gets messy

## 6. Measuring Before Font Load = Measuring Wrong

**Pitfall**: Page calls `charRect(idx)` to measure bracket position on DOMContentLoaded; fonts haven't loaded yet, every character's width is the fallback font's width, all positions wrong. After fonts load (about 500ms later), bracket's `left: Xpx` is still the old value, permanently offset.

**Rules**:
- Any layout code depending on DOM measurement (`getBoundingClientRect`, `offsetWidth`) **must** be wrapped in `document.fonts.ready.then()`
  ```js
  document.fonts.ready.then(() => {
    requestAnimationFrame(() => {
      buildBrackets(...);  // Fonts now ready, measurement accurate
      tick();              // Animation starts
    });
  });
  ```
- Extra `requestAnimationFrame` gives the browser a frame to commit layout
- If using Google Fonts CDN, `<link rel="preconnect">` accelerates first-time loading

## 7. Recording Preparation — Reserve Hooks for Video Export

**Pitfall**: Playwright `recordVideo` defaults to 25fps, recording from context creation. The first 2 seconds of page load and font load are recorded. Delivered video has 2 seconds of blank/flash at the start.

**Rules**:
- Provide `render-video.js` tool to handle: warmup navigate → reload to restart animation → wait duration → ffmpeg trim head + convert to H.264 MP4
- The animation's **frame 0** should be the complete initial state with final layout in place (not blank or loading)
- Want 60fps? Use ffmpeg `minterpolate` post-processing, don't expect browser source frame rate
- Want GIF? Two-stage palette (`palettegen` + `paletteuse`); for a 30s 1080p animation can compress to 3MB

See `video-export.md` for complete script invocation methods.

## 8. Batch Export — tmp Directory Must Have PID to Prevent Concurrency Conflicts

**Pitfall**: Used `render-video.js` with 3 processes in parallel to record 3 HTMLs. Because TMP_DIR was named only with `Date.now()`, when 3 processes started in the same millisecond, they shared the same tmp directory. The first process to finish cleaned up tmp, the other two got `ENOENT` reading the directory, all crashed.

**Rules**:
- Any temp directory potentially shared across processes must be named with a **PID or random suffix**:
  ```js
  const TMP_DIR = path.join(DIR, '.video-tmp-' + Date.now() + '-' + process.pid);
  ```
- If you really want parallel multi-file, use shell `&` + `wait` rather than forking within one node script
- For batch recording multiple HTMLs, conservative approach: **serial** execution (up to 2 in parallel, 3+ honestly queue up)

## 9. Recordings Have Progress Bars / Replay Buttons — Chrome Elements Pollute Video

**Pitfall**: Animation HTML added a `.progress` progress bar, `.replay` replay button, `.counter` timestamp for human debugging convenience. When recorded as MP4 for delivery, these elements appeared at the bottom of the video, like screenshots of dev tools were captured.

**Rules**:
- "Chrome elements" in HTML for human use (progress bar / replay button / footer / masthead / counter / phase labels) should be managed separately from the main video content
- **Convention class name** `.no-record`: any element with this class is automatically hidden by the recording script
- Script side (`render-video.js`) injects CSS by default to hide common chrome class names:
  ```
  .progress .counter .phases .replay .masthead .footer .no-record [data-role="chrome"]
  ```
- Inject using Playwright's `addInitScript` (takes effect before each navigate, also stable on reload)
- Add `--keep-chrome` flag when wanting to see original HTML (with chrome)

## 10. Animation Repeats in First Few Seconds of Recording — Warmup Frame Leakage

**Pitfall**: `render-video.js` old flow `goto → wait fonts 1.5s → reload → wait duration`. Recording started from context creation; warmup phase animation already played a segment, restarted from 0 after reload. Result: first few seconds of video are "mid-animation + switch + animation starting from 0", strong sense of repetition.

**Rules**:
- **Warmup and Record must use independent contexts**:
  - Warmup context (no `recordVideo` option): only responsible for load url, wait fonts, then close
  - Record context (with `recordVideo`): starts from fresh state, animation records from t=0
- ffmpeg `-ss trim` can only cut a tiny bit of Playwright's startup latency (~0.3s), **cannot** be used to mask warmup frames; the source must be clean
- Closing the recording context = webm file written to disk, this is Playwright's constraint
- Related code pattern:
  ```js
  // Phase 1: warmup (throwaway)
  const warmupCtx = await browser.newContext({ viewport });
  const warmupPage = await warmupCtx.newPage();
  await warmupPage.goto(url, { waitUntil: 'networkidle' });
  await warmupPage.waitForTimeout(1200);
  await warmupCtx.close();

  // Phase 2: record (fresh)
  const recordCtx = await browser.newContext({ viewport, recordVideo });
  const page = await recordCtx.newPage();
  await page.goto(url, { waitUntil: 'networkidle' });
  await page.waitForTimeout(DURATION * 1000);
  await page.close();
  await recordCtx.close();
  ```

## 11. Don't Draw "Fake Chrome" Inside the Frame — Decorative Player UI Collides with Real Chrome

**Pitfall**: Animation used `Stage` component, which already has scrubber + timecode + pause button (categorized as `.no-record` chrome, auto-hidden on export). I also drew a "`00:60 ──── CLAUDE-DESIGN / ANATOMY`" "magazine page-number-feel decorative progress bar" at the bottom of the frame, feeling good about myself. **Result**: user saw two progress bars — one from the Stage controller, one from my decoration. Visually a complete collision, identified as a bug. "What's with another progress bar inside the video?"

**Rules**:

- Stage already provides: scrubber + timecode + pause/replay buttons. **Don't draw inside the frame** progress indicators, current timecode, copyright signature bars, chapter counters — they either collide with chrome, or are pure filler slop (violating the "earn its place" principle).
- "Page-number feel", "magazine feel", "bottom signature bar" — these **decorative requests** are high-frequency filler AI auto-adds. Be wary of every occurrence — does it really convey irreplaceable information? Or is it merely filling blank space?
- If you firmly believe a bottom band must exist (e.g., the animation theme is about player UI), it must be **narratively necessary**, and **visually distinguishable from the Stage scrubber** (different position, different form, different tone).

**Element ownership test** (every element drawn into the canvas must answer):

| What does it belong to | Treatment |
|------------|------|
| Narrative content of a scene | OK, keep it |
| Global chrome (control/debug use) | Add `.no-record` class, hide on export |
| **Belongs to no scene, isn't chrome either** | **Delete**. This is ownerless, must be filler slop |

**Self-check (3 seconds before delivery)**: take a static screenshot, ask yourself —

- Is there anything in the frame that "looks like video player UI" (horizontal progress bar, timecode, control button shapes)?
- If yes, would deleting it harm the narrative? If not, delete.
- Does the same type of information (progress/time/signature) appear twice? Consolidate to one chrome location.

**Counter-examples**: drawing `00:42 ──── PROJECT NAME` at the bottom, drawing "CH 03 / 06" chapter counter at the bottom-right, drawing version number "v0.3.1" at the frame edge — all are fake chrome filler.

## 12. Recording Leading Blank + Recording Start Offset — `__ready` × tick × lastTick Triple Trap

**Pitfall (A · leading blank)**: 60-second animation exported MP4, first 2-3 seconds are blank pages. `ffmpeg --trim=0.3` can't cut it.

**Pitfall (B · start offset, real incident on 2026-04-20)**: Exported 24-second video; user perception: "video starts playing the first frame at 19 seconds". Actually the animation was recorded starting from t=5, recorded until t=24 then loop back to t=0, then recorded another 5 seconds to end — so the last 5 seconds of video is the actual animation start.

**Root cause** (both pitfalls share one root cause):

Playwright `recordVideo` starts writing WebM at the moment of `newContext()`; at this time Babel/React/font loading takes L seconds (2-6s). The recording script waits for `window.__ready = true` as the anchor "animation starts here" — it must strictly pair with the animation `time = 0`. Two common mistakes:

| Mistake | Symptom |
|------|------|
| `__ready` set in `useEffect` or sync setup phase (before tick's first frame) | Recording script thinks animation started, but WebM is still recording blank page → **leading blank** |
| tick's `lastTick = performance.now()` initialized at **script top level** | Font loading L seconds counted into first frame's `dt`, `time` jumps instantly to L → recording lags L seconds throughout → **start offset** |

**✅ Correct complete starter tick template** (handwritten animation must use this skeleton):

```js
// ━━━━━━ state ━━━━━━
let time = 0;
let playing = false;   // ❗ Default not playing, wait until fonts ready to start
let lastTick = null;   // ❗ Sentinel — tick first frame forces dt to 0 (don't use performance.now())
const fired = new Set();

// ━━━━━━ tick ━━━━━━
function tick(now) {
  if (lastTick === null) {
    lastTick = now;
    window.__ready = true;   // ✅ Pair: "recording start" with "animation t=0" same frame
    render(0);               // Render again to ensure DOM ready (fonts are now ready)
    requestAnimationFrame(tick);
    return;
  }
  const dt = (now - lastTick) / 1000;   // dt only starts advancing after first frame
  lastTick = now;

  if (playing) {
    let t = time + dt;
    if (t >= DURATION) {
      t = window.__recording ? DURATION - 0.001 : 0;  // Don't loop when recording, leave 0.001s to keep last frame
      if (!window.__recording) fired.clear();
    }
    time = t;
    render(time);
  }
  requestAnimationFrame(tick);
}

// ━━━━━━ boot ━━━━━━
// Don't immediately rAF at top level — wait until fonts loaded to start
document.fonts.ready.then(() => {
  render(0);                 // Draw initial frame first (fonts ready)
  playing = true;
  requestAnimationFrame(tick);  // First tick will pair __ready + t=0
});

// ━━━━━━ seek interface (for render-video defensive correction) ━━━━━━
window.__seek = (t) => { fired.clear(); time = t; lastTick = null; render(t); };
```

**Why this template is correct**:

| Step | Why this is necessary |
|------|-------------|
| `lastTick = null` + first frame `return` | Avoid the L seconds from "script load to tick first execution" being counted into animation time |
| `playing = false` default | Even if `tick` runs during font loading, time doesn't advance, avoiding render misalignment |
| `__ready` set on tick first frame | Recording script starts timing here, the corresponding frame is animation's true t=0 |
| Start tick only inside `document.fonts.ready.then(...)` | Avoid font fallback width measurement, avoid first-frame font swap |
| `window.__seek` exists | Lets `render-video.js` actively correct — second line of defense |

**Recording script-side corresponding defense**:
1. `addInitScript` injects `window.__recording = true` (before page goto)
2. `waitForFunction(() => window.__ready === true)`, record this moment's offset for ffmpeg trim
3. **Additionally**: after `__ready`, actively `page.evaluate(() => window.__seek && window.__seek(0))`, force HTML's potential time deviation to zero — second line of defense, against HTML that doesn't strictly follow the starter template

**Verification method**: after exporting MP4
```bash
ffmpeg -i video.mp4 -ss 0 -vframes 1 frame-0.png
ffmpeg -i video.mp4 -ss $DURATION-0.1 -vframes 1 frame-end.png
```
First frame must be the animation t=0 initial state (not mid-animation, not black), last frame must be animation final state (not some moment of the second loop).

**Reference implementation**: Stage component in `assets/animations.jsx`, `scripts/render-video.js` are both implemented per this protocol. Handwritten HTML must follow the starter tick template — every line defends against a specific bug.

## 13. Recording Must Disable Loop — `window.__recording` Signal

**Pitfall**: Animation Stage default `loop=true` (convenient for viewing in browser). `render-video.js` waits an extra 300ms after recording duration seconds before stopping; this 300ms makes Stage enter the next loop. ffmpeg `-t DURATION` truncates and the last 0.5-1s falls into the next loop — video end suddenly returns to first frame (Scene 1), viewers think there's a video bug.

**Root cause**: No "I'm recording" handshake protocol between recording script and HTML. HTML doesn't know it's being recorded, still loops per browser interaction scenario.

**Rules**:

1. **Recording script**: in `addInitScript`, inject `window.__recording = true` (before page goto):
   ```js
   await recordCtx.addInitScript(() => { window.__recording = true; });
   ```

2. **Stage component**: recognize this signal, force loop=false:
   ```js
   const effectiveLoop = (typeof window !== 'undefined' && window.__recording) ? false : loop;
   // ...
   if (next >= duration) return effectiveLoop ? 0 : duration - 0.001;
   //                                                       ↑ Leave 0.001 to prevent Sprite end=duration from being closed off
   ```

3. **Ending Sprite's fadeOut**: in recording scenarios should set `fadeOut={0}`, otherwise video end fades to transparent/dark — user expects to stop on a clear last frame, not fade out. For handwritten HTML, recommend setting `fadeOut={0}` on ending Sprites.

**Reference implementation**: Stage in `assets/animations.jsx` / `scripts/render-video.js` both have built-in handshake. Handwritten Stage must implement `__recording` detection — otherwise recording will definitely hit this pitfall.

**Verification**: after exporting MP4 `ffmpeg -ss 19.8 -i video.mp4 -frames:v 1 end.png`, check whether the last 0.2 seconds is still the expected last frame, without sudden switch to another scene.

## 14. 60fps Video Defaults to Frame Duplication — minterpolate Has Poor Compatibility

**Pitfall**: `convert-formats.sh` used `minterpolate=fps=60:mi_mode=mci...` to generate 60fps MP4, which can't be opened in some macOS QuickTime / Safari versions (all black or directly refused). VLC / Chrome can open it.

**Root cause**: minterpolate's output H.264 elementary stream contains certain SEI / SPS fields that some players have trouble parsing.

**Rules**:

- Default 60fps uses simple `fps=60` filter (frame duplication), with broad compatibility (QuickTime/Safari/Chrome/VLC can all open)
- High-quality interpolation uses `--minterpolate` flag explicit enable — but **must be locally tested** on target player before delivery
- The value of the 60fps tag is **upload platform algorithm recognition** (Bilibili / YouTube prioritize 60fps-tagged content); actual perceived smoothness improvement for CSS animation is minimal
- Add `-profile:v high -level 4.0` to improve H.264 universal compatibility

**`convert-formats.sh` already defaults to compatibility mode**. If you need high-quality interpolation, add `--minterpolate` flag:
```bash
bash convert-formats.sh input.mp4 --minterpolate
```

## 15. `file://` + External `.jsx` CORS Trap — Single-File Delivery Must Inline the Engine

**Pitfall**: Animation HTML used `<script type="text/babel" src="animations.jsx"></script>` to externally load the engine. Local double-click open (`file://` protocol) → Babel Standalone uses XHR to fetch `.jsx` → Chrome reports `Cross origin requests are only supported for protocol schemes: http, https, chrome, chrome-extension...` → entire page goes black, doesn't report `pageerror`, only console error, very easy to misdiagnose as "animation not triggered".

Starting an HTTP server may not save you either — when the local machine has a global proxy, `localhost` also goes through the proxy, returning 502 / connection failures.

**Rules**:

- **Single-file delivery (HTML usable on double-click)** → `animations.jsx` must be **inlined** into the `<script type="text/babel">...</script>` tag, don't use `src="animations.jsx"`
- **Multi-file project (start HTTP server demo)** → can be externally loaded, but clearly state `python3 -m http.server 8000` command on delivery
- Decision criterion: is what you're delivering to the user an "HTML file" or "project directory with server"? The former uses inline
- Stage component / animations.jsx is often 200+ lines — pasting into HTML `<script>` block is completely acceptable, don't fear the size

**Minimum verification**: double-click your generated HTML, **don't** open through any server. If Stage normally displays the animation's first frame, it passes.

## 16. Cross-Scene Color-Inversion Context — Don't Hardcode Colors on In-Frame Elements

**Pitfall**: When making multi-scene animation, `ChapterLabel` / `SceneNumber` / `Watermark` etc. **elements appearing across scenes**, hardcoded `color: '#1A1A1A'` (dark text) in the component. First 4 scenes with light backgrounds OK; on the 5th black-background scene, "05" and watermark directly disappeared — no error, no triggered check, key information invisible.

**Rules**:

- **In-frame elements reused across multiple scenes** (chapter labels / scene numbers / timecodes / watermarks / copyright bars) **prohibited from hardcoding color values**
- Use one of three approaches:
  1. **`currentColor` inheritance**: element only writes `color: currentColor`, parent scene container sets `color: computed value`
  2. **invert prop**: component accepts `<ChapterLabel invert />` to manually toggle light/dark
  3. **Auto-calculate based on background color**: `color: contrast-color(var(--scene-bg))` (CSS 4 new API, or JS judgment)
- Before delivery use Playwright to capture **representative frame of each scene**, eyeball "cross-scene elements" to confirm they're all visible

The hidden danger of this pitfall — **no bug alarm**. Only the human eye or OCR can find it.

## Quick Self-Check Checklist (5 seconds before starting)

- [ ] Every `position: absolute` parent has `position: relative`?
- [ ] Special characters in animation (`␣` `⌘` `emoji`) all exist in the font?
- [ ] Grid/Flex template count matches JS data length?
- [ ] Cross-fade between scene switches, no pure blank >0.3s?
- [ ] DOM measurement code wrapped in `document.fonts.ready.then()`?
- [ ] `render(t)` is pure, or has explicit reset mechanism?
- [ ] Frame 0 is complete initial state, not blank?
- [ ] No "fake chrome" decoration in frame (progress bar / timecode / bottom signature bar colliding with Stage scrubber)?
- [ ] Animation tick first frame synchronously sets `window.__ready = true`? (Animations.jsx has it built-in; handwritten HTML must add it)
- [ ] Stage detects `window.__recording` to force loop=false? (Handwritten HTML must add)
- [ ] Ending Sprite's `fadeOut` set to 0 (video end stops on clear frame)?
- [ ] 60fps MP4 defaults to frame duplication mode (compatibility), high-quality interpolation only with `--minterpolate`?
- [ ] After export, capture frame 0 + end frame to verify they're animation initial/final state?
- [ ] When involving specific brands (Stripe/Anthropic/Lovart/...): completed the "brand asset protocol" (SKILL.md §1.a five steps)? Wrote `brand-spec.md`?
- [ ] Single-file delivery HTML: `animations.jsx` is inlined, not `src="..."`? (External .jsx under file:// causes CORS black screen)
- [ ] Cross-scene elements (chapter labels / watermarks / scene numbers) don't hardcode colors? Visible under each scene's background?
