# Editable PPTX Export: HTML Hard Constraints + Sizing Decisions + Common Errors

This document covers the path of **using `scripts/html2pptx.js` + `pptxgenjs` to translate HTML element-by-element into truly editable PowerPoint text frames**, which is also the only path supported by `export_deck_pptx.mjs`.

> **Core prerequisite**: To take this path, the HTML must be written from line one according to the 4 constraints below. **Don't write first then convert** — after-the-fact remediation triggers 2-3 hours of rework (real-world pitfall from the 2026-04-20 Options Private Board project).
>
> For visual-freedom-priority scenarios (animations / web components / CSS gradients / complex SVGs), switch to the PDF path (`export_deck_pdf.mjs` / `export_deck_stage_pdf.mjs`). **Don't** expect pptx export to balance visual fidelity and editability — this is a physical constraint of the PPTX file format itself (see "Why the 4 constraints aren't a Bug but physical constraints" at the end).

---

## Canvas Size: Use 960×540pt (LAYOUT_WIDE)

PPTX units are **inch** (physical size), not px. Decision principle: the body's computedStyle dimensions must **match the presentation layout's inch dimensions** (±0.1", strictly checked by `validateDimensions` in `html2pptx.js`).

### Comparison of 3 candidate sizes

| HTML body | Physical size | Corresponding PPT layout | When to choose |
|---|---|---|---|
| **`960pt × 540pt`** | **13.333″ × 7.5″** | **pptxgenjs `LAYOUT_WIDE`** | ✅ **Default recommendation** (modern PowerPoint 16:9 standard) |
| `720pt × 405pt` | 10″ × 5.625″ | Custom | Only when user specifies "old-version PowerPoint Widescreen" template |
| `1920px × 1080px` | 20″ × 11.25″ | Custom | ❌ Non-standard size, fonts appear unusually small after projection |

**Don't think of HTML size as resolution.** PPTX is a vector document; body size determines **physical size** not clarity. An oversized body (20″×11.25″) won't make text clearer — it only makes the font-pt relatively smaller against the canvas, looking worse when projected/printed.

### Three equivalent body writing styles

```css
body { width: 960pt;  height: 540pt; }    /* Clearest, recommended */
body { width: 1280px; height: 720px; }    /* Equivalent, px convention */
body { width: 13.333in; height: 7.5in; }  /* Equivalent, inch intuition */
```

Matching pptxgenjs code:

```js
const pptx = new pptxgen();
pptx.layout = 'LAYOUT_WIDE';  // 13.333 × 7.5 inch, no customization needed
```

---

## 4 Hard Constraints (violations cause direct errors)

`html2pptx.js` translates HTML DOM elements one-by-one into PowerPoint objects. PowerPoint's format constraints projected onto HTML = the 4 rules below.

### Rule 1: Text cannot be written directly inside DIV — must be wrapped in `<p>` or `<h1>`-`<h6>`

```html
<!-- ❌ Wrong: text directly in div -->
<div class="title">Q3 Revenue Growth 23%</div>

<!-- ✅ Correct: text in <p> or <h1>-<h6> -->
<div class="title"><h1>Q3 Revenue Growth 23%</h1></div>
<div class="body"><p>New users are the main driver</p></div>
```

**Why**: PowerPoint text must exist within a text frame, and text frames correspond to paragraph-level HTML elements (p/h*/li). A bare `<div>` has no corresponding text container in PPTX.

**Also can't use `<span>` to hold primary text** — span is an inline element and can't be aligned independently as a text frame. Span can only be **nested inside p/h\*** for local styling (bold, color change).

### Rule 2: CSS gradients not supported — only solid colors

```css
/* ❌ Wrong */
background: linear-gradient(to right, #FF6B6B, #4ECDC4);

/* ✅ Correct: solid color */
background: #FF6B6B;

/* ✅ If multi-color stripes are needed, use flex children with individual solid colors */
.stripe-bar { display: flex; }
.stripe-bar div { flex: 1; }
.red   { background: #FF6B6B; }
.teal  { background: #4ECDC4; }
```

**Why**: PowerPoint shape fill only supports solid/gradient-fill, but pptxgenjs's `fill: { color: ... }` only maps to solid. Going through PowerPoint native gradient requires writing additional structure, which the current toolchain doesn't support.

### Rule 3: Background/border/shadow can only be on DIV, not on text tags

```html
<!-- ❌ Wrong: <p> has background color -->
<p style="background: #FFD700; border-radius: 4px;">Key content</p>

<!-- ✅ Correct: outer div carries background/border, <p> only handles text -->
<div style="background: #FFD700; border-radius: 4px; padding: 8pt 12pt;">
  <p>Key content</p>
</div>
```

**Why**: In PowerPoint, shape (rectangle/rounded rectangle) and text frame are two separate objects. HTML's `<p>` only translates to a text frame; background/border/shadow belong to the shape — must be written on the **div wrapping the text**.

### Rule 4: DIV cannot use `background-image` — use `<img>` tag

```html
<!-- ❌ Wrong -->
<div style="background-image: url('chart.png')"></div>

<!-- ✅ Correct -->
<img src="chart.png" style="position: absolute; left: 50%; top: 20%; width: 300pt; height: 200pt;" />
```

**Why**: `html2pptx.js` only extracts image paths from `<img>` elements; it doesn't parse CSS `background-image` URLs.

---

## Path A HTML Template Skeleton

Each slide is an independent HTML file, scope-isolated from the others (avoiding CSS pollution from single-file decks).

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
<meta charset="UTF-8">
<style>
  * { margin: 0; padding: 0; box-sizing: border-box; }
  body {
    width: 960pt; height: 540pt;           /* ⚠️ Match LAYOUT_WIDE */
    font-family: system-ui, -apple-system, "PingFang SC", sans-serif;
    background: #FEFEF9;                    /* Solid color, no gradients */
    overflow: hidden;
  }
  /* DIV handles layout/background/border */
  .card {
    position: absolute;
    background: #1A4A8A;                    /* Background on DIV */
    border-radius: 4pt;
    padding: 12pt 16pt;
  }
  /* Text tags only handle font styling, no background/border */
  .card h2 { font-size: 24pt; color: #FFFFFF; font-weight: 700; }
  .card p  { font-size: 14pt; color: rgba(255,255,255,0.85); }
</style>
</head>
<body>

  <!-- Title area: outer div positions, inner text tags -->
  <div style="position: absolute; top: 40pt; left: 60pt; right: 60pt;">
    <h1 style="font-size: 36pt; color: #1A1A1A; font-weight: 700;">Use assertion sentences for titles, not topic words</h1>
    <p style="font-size: 16pt; color: #555555; margin-top: 10pt;">Subtitle supplementary explanation</p>
  </div>

  <!-- Content card: div handles background, h2/p handle text -->
  <div class="card" style="top: 130pt; left: 60pt; width: 240pt; height: 160pt;">
    <h2>Point One</h2>
    <p>Brief explanation text</p>
  </div>

  <!-- List: use ul/li, don't manually use • symbol -->
  <div style="position: absolute; top: 320pt; left: 60pt; width: 540pt;">
    <ul style="font-size: 16pt; color: #1A1A1A; padding-left: 24pt; list-style: disc;">
      <li>First key point</li>
      <li>Second key point</li>
      <li>Third key point</li>
    </ul>
  </div>

  <!-- Illustration: use <img> tag, not background-image -->
  <img src="illustration.png" style="position: absolute; right: 60pt; top: 110pt; width: 320pt; height: 240pt;" />

</body>
</html>
```

---

## Common Error Quick Reference

| Error message | Cause | Fix |
|---------|------|---------|
| `DIV element contains unwrapped text "XXX"` | Bare text in div | Wrap text in `<p>` or `<h1>`-`<h6>` |
| `CSS gradients are not supported` | Used linear/radial-gradient | Change to solid color, or use flex children to segment |
| `Text element <p> has background` | `<p>` tag has background color | Wrap with `<div>` to carry background, `<p>` only writes text |
| `Background images on DIV elements are not supported` | div used background-image | Change to `<img>` tag |
| `HTML content overflows body by Xpt vertically` | Content exceeds 540pt | Reduce content or shrink font size, or `overflow: hidden` to truncate |
| `HTML dimensions don't match presentation layout` | body size doesn't match pres layout | Use `960pt × 540pt` for body with `LAYOUT_WIDE`; or defineLayout for custom size |
| `Text box "XXX" ends too close to bottom edge` | Large-font `<p>` is < 0.5 inch from body bottom edge | Move up, leave sufficient bottom margin; PPT bottom itself gets partially obscured |

---

## Basic Workflow (3 steps to PPTX)

### Step 1: Write each page as independent HTML following the constraints

```
MyDeck/
├── slides/
│   ├── 01-cover.html    # Each file is a complete 960×540pt HTML
│   ├── 02-agenda.html
│   └── ...
└── illustration/        # All images referenced by <img>
    ├── chart1.png
    └── ...
```

### Step 2: Write build.js calling `html2pptx.js`

```js
const pptxgen = require('pptxgenjs');
const html2pptx = require('../scripts/html2pptx.js');  // This skill's script

(async () => {
  const pres = new pptxgen();
  pres.layout = 'LAYOUT_WIDE';  // 13.333 × 7.5 inch, matches HTML's 960×540pt

  const slides = ['01-cover.html', '02-agenda.html', '03-content.html'];
  for (const file of slides) {
    await html2pptx(`./slides/${file}`, pres);
  }

  await pres.writeFile({ fileName: 'deck.pptx' });
})();
```

### Step 3: Open and inspect

- Open exported PPTX in PowerPoint/Keynote
- Double-click any text — it should be directly editable (if it's an image, rule 1 was violated)
- Verify overflow: each page should be within body bounds, not truncated

---

## This Path vs Other Options (when to choose what)

| Need | Choose what |
|------|------|
| Colleagues will edit text in PPTX / send to non-technical people for further editing | **This document's path** (editable, must write HTML from scratch following the 4 constraints) |
| Just for presentation / archive, no further editing | `export_deck_pdf.mjs` (multi-file) or `export_deck_stage_pdf.mjs` (single-file deck-stage), produces vector PDF |
| Visual freedom priority (animations, web components, CSS gradients, complex SVGs), accept non-editable | **PDF** (same as above) — PDF preserves fidelity and is cross-platform, more suitable than "image PPTX" |

**Never force-run html2pptx on visually-free-written HTML** — measured pass rate of visually-driven HTML is < 30%, and remaining page-by-page transformation is slower than rewriting. This scenario should produce PDF, not force PPTX.

---

## Fallback: Already Have Visual Draft But User Insists on Editable PPTX

Occasionally you'll encounter this scenario: you/the user have already written a visually-driven HTML (gradients, web components, complex SVGs all used), originally most suitable for PDF output, but the user explicitly says "no, must be editable PPTX".

**Don't force-run `html2pptx` expecting it to pass** — measured visual-driven HTML pass rate on html2pptx is <30%, the remaining 70% will error or distort. The correct fallback is:

### Step 1 · Inform Limitations First (transparent communication)

In one sentence, tell the user three things clearly:

> "Your current HTML uses [specifically list: gradients / web components / complex SVGs / ...]. Direct conversion to editable PPTX will fail. I have two options:
> - A. **Output PDF** (recommended) — visual 100% preserved, recipients can view and print but can't edit text
> - B. **Use the visual draft as blueprint, rewrite an editable HTML version** (preserving design decisions for color/layout/copy, but reorganizing HTML structure following the 4 hard constraints, **sacrificing** gradients, web components, complex SVGs and other visual capabilities) → then export editable PPTX
>
> Which would you choose?"

Don't make option B sound easygoing — explicitly inform **what will be lost**. Let the user make the trade-off.

### Step 2 · If User Chooses B: AI Proactively Rewrites, Don't Ask User to Write

The doctrine here is: **the user provides design intent, you're responsible for translating it into compliant implementation**. Not making the user learn the 4 hard constraints and rewrite themselves.

Principles followed when rewriting:
- **Preserve**: color system (primary/secondary/neutral), information hierarchy (title/subtitle/body/caption), core copy, layout skeleton (top/middle/bottom / left/right columns / grid), page rhythm
- **Downgrade**: CSS gradients → solid colors or flex segments, web components → paragraph-level HTML, complex SVG → simplified `<img>` or solid geometric shapes, shadows → remove or reduce to extremely faint, custom fonts → align to system fonts
- **Rewrite**: bare text → wrap in `<p>` / `<h*>`, `background-image` → `<img>` tag, background/border on `<p>` → outer div carries it

### Step 3 · Produce Comparison Checklist (transparent delivery)

After rewriting, give the user a before/after comparison so they know which visual details were simplified:

```
Original design → editable version adjustments
- Title area purple gradient → primary #5B3DE8 solid background
- Data card shadow → removed (replaced with 2pt outline for distinction)
- Complex SVG line chart → simplified to <img> PNG (generated from HTML screenshot)
- Hero area web component animation → static first frame (web component can't be translated)
```

### Step 4 · Export & Dual-Format Delivery

- `editable` version HTML → run `scripts/export_deck_pptx.mjs` to produce editable PPTX
- **Recommended to also keep** the original visual draft → run `scripts/export_deck_pdf.mjs` to produce high-fidelity PDF
- Dual-format delivery to user: PDF of visual draft + editable PPTX, each serving its purpose

### When to Directly Refuse Option B

In some scenarios the rewriting cost is too high; advise the user to give up on editable PPTX:
- HTML core value is animation or interaction (after rewriting, only static first frame remains, information loss 50%+)
- Pages > 30, rewriting cost over 2 hours
- Visual design deeply depends on precise SVG / custom filters (after rewriting, almost unrelated to original)

At this point tell the user: "This deck has too high a rewriting cost; recommend producing PDF instead of PPTX. If the recipient really needs pptx format, accept that the visuals will be significantly simplified — would you like to switch to PDF?"

---

## Why the 4 Constraints Aren't a Bug but Physical Constraints

These 4 aren't because the `html2pptx.js` author was lazy — they're the result of **PowerPoint file format (OOXML) constraints themselves** projected onto HTML:

- In PPTX, text must be in a text frame (`<a:txBody>`), corresponding to paragraph-level HTML elements
- PPTX shape and text frame are two separate objects, can't draw background and write text on the same element simultaneously
- PPTX shape fill has limited support for gradients (only some preset gradients, doesn't support arbitrary-angle CSS gradients)
- PPTX picture object must reference real image files, not CSS properties

After understanding this, **don't expect the tool to get smarter** — HTML writing must adapt to PPTX format, not the other way around.
