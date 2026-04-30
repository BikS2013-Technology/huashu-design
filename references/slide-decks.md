# Slide Decks: HTML Slide Production Standards

Making slides is a high-frequency design scenario. This document explains how to make HTML slides well — from architecture choice, single-page design, to the complete path of PDF/PPTX export.

**Coverage of this skill's capabilities**:
- **HTML presentation version (the foundational deliverable, always default and required)** → each page an independent HTML + `assets/deck_index.html` aggregation, keyboard paging in browser, full-screen presentation
- HTML → PDF export → `scripts/export_deck_pdf.mjs` / `scripts/export_deck_stage_pdf.mjs`
- HTML → editable PPTX export → `references/editable-pptx.md` + `scripts/html2pptx.js` + `scripts/export_deck_pptx.mjs` (requires HTML written per the 4 hard constraints)

> **⚠️ HTML is the foundation, PDF/PPTX are derivatives.** Regardless of the final delivery format, you **must** first make the HTML aggregation presentation version (`index.html` + `slides/*.html`); it is the "source" of the slide work. PDF/PPTX are snapshots exported from HTML with a one-line command.
>
> **Why HTML first**:
> - Best for live presentation/demos (projector / screen sharing directly full-screen, keyboard paging, no dependency on Keynote/PPT software)
> - During development, each page can be opened individually with a double-click for verification, no need to re-run the export each time
> - It is the unique upstream of PDF/PPTX export (avoids the "found something to fix in HTML after export and have to re-export" dead loop)
> - Deliverables can be "HTML + PDF" or "HTML + PPTX" double bundles, the recipient uses whichever they prefer
>
> 2026-04-22 moxt brochure real test: After completing the 13-page HTML + index.html aggregation, `export_deck_pdf.mjs` exported PDF in one line, zero changes. The HTML version itself is a deliverable that can be presented directly in the browser.

---

## 🛑 Confirm Delivery Format Before Starting (the hardest checkpoint)

**This decision comes even before "single file or multi-file".** 2026-04-20 options private board project real test: **Not confirming delivery format before starting work = 2-3 hours of rework.**

### Decision tree (HTML-first architecture)

All deliveries start from the same set of HTML aggregation pages (`index.html` + `slides/*.html`). The delivery format only determines **the constraints on how HTML is written** and **the export command**:

```
[Always default · Must do] HTML aggregation presentation version (index.html + slides/*.html)
   │
   ├── Just need browser presentation / local HTML archive   → Done here, HTML has maximum visual freedom
   │
   ├── Also need PDF (print / send in chat / archive)        → Run export_deck_pdf.mjs one-click export
   │                                                            HTML writing free, no visual constraints
   │
   └── Also need editable PPTX (colleagues will edit text)   → Write per 4 hard constraints from the first line of HTML
                                                                Run export_deck_pptx.mjs one-click export
                                                                Sacrifice gradients / web components / complex SVGs
```

### Opening dialog (copy-paste ready)

> Regardless of whether the final delivery is HTML, PDF, or PPTX, I'll first make an HTML aggregation version that can be switched and presented in the browser (`index.html` plus keyboard paging) — this is always the default foundational deliverable. On top of that, I'll then ask whether you also want PDF / PPTX snapshots.
>
> Which export format do you need?
> - **Just HTML** (presentation/archive) → completely free visually
> - **Also PDF** → same as above, plus one export command
> - **Also editable PPTX** (colleagues will edit text in PPT) → I must write per the 4 hard constraints from the first line of HTML, sacrificing some visual capabilities (no gradients, no web components, no complex SVGs).

### Why "if you want PPTX, you have to follow 4 hard constraints from the start"

The premise of editable PPTX is that `html2pptx.js` can translate the DOM element-by-element into PowerPoint objects. It needs **4 hard constraints**:

1. body fixed at 960pt × 540pt (matches `LAYOUT_WIDE`, 13.333″ × 7.5″, not 1920×1080px)
2. All text wrapped in `<p>`/`<h1>`-`<h6>` (forbidden to put text directly in div, forbidden to use `<span>` to carry main text)
3. `<p>`/`<h*>` themselves cannot have background/border/shadow (put on outer div)
4. `<div>` cannot use `background-image` (use `<img>` tag)
5. No CSS gradient, no web components, no complex SVG decorations

**This skill's default HTML has high visual freedom** — lots of spans, nested flex, complex SVG, web components (like `<deck-stage>`), CSS gradients — **almost none of it can naturally pass html2pptx's constraints** (real-world test: visually-driven HTML run directly through html2pptx, pass rate < 30%).

### Cost comparison of the two real paths (2026-04-20 real-world pitfalls)

| Path | Approach | Result | Cost |
|------|------|------|------|
| ❌ **Write HTML freely first, fix PPTX after the fact** | Single-file deck-stage + lots of SVG/span decorations | For editable PPTX, only two paths remain:<br>A. Hand-write hundreds of lines of pptxgenjs hardcoded coordinates<br>B. Rewrite 17 pages of HTML into Path A format | 2-3 hours of rework, and the hand-written version has **perpetual maintenance cost** (HTML changes a word, PPTX must be manually re-synced) |
| ✅ **Write per Path A constraints from step one** | Each page an independent HTML + 4 hard constraints + 960×540pt | One command exports 100% editable PPTX, also can be presented full-screen in browser (Path A HTML is standard HTML playable in browser) | When writing HTML, spend an extra 5 minutes thinking "how to wrap text in `<p>`", zero rework |

### What about hybrid delivery?

User says "I want HTML presentation **and** editable PPTX" — **this is not hybrid**, it's PPTX requirements covering HTML requirements. HTML written per Path A can itself be presented full-screen in browser (just add a `deck_index.html` aggregator). **No additional cost.**

User says "I want PPTX **and** animation / web component" — **this is a real conflict**. Tell the user: for editable PPTX you have to sacrifice these visual capabilities. Let them make trade-offs, don't secretly do hand-written pptxgenjs solutions (will become perpetual maintenance debt).

### What if you only learn afterward that PPTX is needed (emergency rescue)

In rare cases: HTML is already written when you discover PPTX is needed. Recommended **fallback flow** (full explanation see end of `references/editable-pptx.md`, "Fallback: Existing visual draft but user insists on editable PPTX"):

1. **First choice: Switch to PDF** (visuals 100% preserved, cross-platform, recipient can view and print) — if recipient's actual need is "presentation/archive", PDF is the best deliverable
2. **Second choice: AI uses the visual draft as blueprint, rewrites an editable HTML version** → export editable PPTX — preserves the design decisions of color/layout/copy, sacrifices gradients, web components, complex SVGs and other visual capabilities
3. **Not recommended: Hand-write pptxgenjs reconstruction** — position, font, alignment all need manual tuning, high maintenance cost, and any subsequent HTML word change requires manual re-sync

Always tell the user the choices, let them decide. **Never start hand-writing pptxgenjs as a first reaction** — that is the last fallback.

---

## 🛑 Before Batch Production: First Make 2 Showcase Pages to Set Grammar

**As long as the deck has ≥ 5 pages, absolutely do not write straight from page 1 to the last page.** The correct order verified in the 2026-04-22 moxt brochure real-world practice:

1. Pick **2 page types with the largest visual difference** to make showcase first (e.g., "cover" + "mood/quote page", or "cover" + "product showcase page")
2. Screenshot for user to confirm grammar (masthead / fonts / colors / spacing / structure / Chinese-English bilingual ratio)
3. Once direction is approved, batch-push the remaining N-2 pages, each reusing the established grammar
4. After all pages are done, combine into HTML aggregation + PDF / PPTX derivatives

**Why**: Writing 13 pages all the way through → user says "wrong direction" = 13 reworks. Make 2 showcase pages first → wrong direction = 2 reworks. Once visual grammar is established, the decision space for the subsequent N pages dramatically shrinks, leaving only "how to put content in".

**Showcase page selection principle**: Pick the two pages with the most different visual structure. If these two pass = all the other middle states will pass.

| Deck Type | Recommended showcase page combinations |
|-----------|---------------------|
| B2B brochure / product launch | Cover + content page (philosophy/emotion page) |
| Brand release | Cover + product feature page |
| Data report | Big-data page + analysis conclusion page |
| Tutorial / courseware | Chapter cover page + specific knowledge point page |

---

## 📐 Publication Grammar Template (moxt verified, reusable)

Suitable for B2B brochure / product launch / long report decks. Reusing this structure per page = 13 pages with completely consistent visuals, 0 rework.

### Per-page skeleton

```
┌─ masthead (top strip + horizontal line)─────┐
│  [logo 22-28px] · A Product Brochure                Issue · Date · URL │
├──────────────────────────────────────────┤
│                                          │
│  ── kicker (green short bar + uppercase label)   │
│  CHAPTER XX · SECTION NAME                 │
│                                          │
│  H1 (Chinese Noto Serif SC 900)           │
│  Key words separately in brand main color │
│                                          │
│  English subtitle (Lora italic)            │
│  ─────────── divider ──────────            │
│                                          │
│  [specific content: two-column 60/40 / 2x2 grid / list] │
│                                          │
├──────────────────────────────────────────┤
│ section name                     XX / total │
└──────────────────────────────────────────┘
```

### Style conventions (copy directly)

- **H1**: Chinese Noto Serif SC 900, font size 80-140px depending on info volume, key words separately in brand main color (don't pile color throughout)
- **English sub**: Lora italic 26-46px, brand signature words (e.g. "AI team") bold + main color italic
- **Body**: Noto Serif SC 17-21px, line-height 1.75-1.85
- **accent highlight**: Use main color bold to highlight keywords in the body, no more than 3 places per page (more loses the anchor function)
- **Background**: Warm cream base #FAFAFA + extremely faint radial-gradient noise (`rgba(33,33,33,0.015)`) to add paper feel

### The visual lead must be differentiated

13 pages all being "text + a screenshot" is too monotonous. **Rotate the visual lead type per page**:

| Visual type | Suitable section |
|---------|---------------|
| Cover layout (big text + masthead + pillar) | Home / chapter cover |
| Single-character portrait (oversized single momo, etc.) | Introducing a single concept/character |
| Multi-character group photo / avatar cards in a row | Team / user case |
| Timeline cards progressing | Showing "long-term relationship", "evolution" |
| Knowledge graph / connection node diagram | Showing "collaboration", "flow" |
| Before/After comparison cards + middle arrow | Showing "change", "difference" |
| Product UI screenshot + outlined device frame | Specific feature showcase |
| Big quote big-quote (half-page big text) | Mood page / problem page / quote page |
| Real avatar + quote card (2×2 or 1×4) | User testimonials / use scenarios |
| Big-text closing + URL ellipse button | CTA / ending |

---

## ⚠️ Common Pitfalls (moxt real-world summary)

### 1. Emojis don't render during Chromium / Playwright export

Chromium doesn't ship with color emoji fonts by default. When `page.pdf()` or `page.screenshot()` runs, emojis display as empty boxes.

**Countermeasure**: Use Unicode text symbols (`✦` `✓` `✕` `→` `·` `—`) as substitutes, or change directly to plain text ("Email · 23" instead of "📧 23 emails").

### 2. `export_deck_pdf.mjs` errors with `Cannot find package 'playwright'`

Reason: ESM module resolution looks for `node_modules` upward from the script's location. The script is in `~/.claude/skills/huashu-design/scripts/`, no dependencies there.

**Countermeasure**: Copy the script to the deck project directory (e.g. `brochure/build-pdf.mjs`), run `npm install playwright pdf-lib` at the project root, then `node build-pdf.mjs --slides slides --out output/deck.pdf`.

### 3. Google Fonts haven't finished loading when screenshot is taken → Chinese displays as system default heiti

Before Playwright screenshot/PDF, at least `wait-for-timeout=3500` to let webfonts download and paint. Or self-host the fonts to `shared/fonts/` to reduce network dependency.

### 4. Information density imbalance: content page stuffed too full

The first version of the moxt philosophy page used 2×2 = 4 segments + 3 tenets at the bottom = 7 blocks of content, cramped and repetitive. Changed to 1×3 = 3 segments and the breath came back immediately.

**Countermeasure**: Each page is controlled to "1 core piece of information + 3-4 supporting points + 1 visual lead", split to a new page if exceeded. **Less is more** — the audience looks at one page for 10 seconds; giving them 1 memory point is easier to remember than 4.

---

## 🛑 First Decide Architecture: Single File or Multi-File?

**This choice is the first step in making slides; mistakes will cause repeated pitfalls. Read this section before acting.**

### Comparison of the two architectures

| Dimension | Single file + `deck_stage.js` | **Multi-file + `deck_index.html` aggregator** |
|------|--------------------------|--------------------------------------|
| Code structure | One HTML, all slides are `<section>` | Each page an independent HTML, `index.html` aggregates with iframe |
| CSS scope | ❌ Global, one page's styles may affect all pages | ✅ Naturally isolated, iframes each their own world |
| Verification granularity | ❌ Need JS goTo to switch to a page | ✅ Single page file double-click can be viewed in browser |
| Parallel development | ❌ One file, multiple agents editing will conflict | ✅ Multiple agents can do different pages in parallel, zero conflict merge |
| Debugging difficulty | ❌ One CSS error, the whole deck flips | ✅ One page error only affects itself |
| Embedded interaction | ✅ Cross-page shared state is simple | 🟡 Inter-iframe needs postMessage |
| Print PDF | ✅ Built-in | ✅ Aggregator beforeprint traverses iframes |
| Keyboard navigation | ✅ Built-in | ✅ Built into aggregator |

### Which to choose? (Decision tree)

```
│ Q: How many pages does the deck plan to have?
├── ≤10 pages, needs in-deck animation or cross-page interaction, pitch deck → single file
└── ≥10 pages, academic lecture, courseware, long deck, multi-agent parallel → multi-file (recommended)
```

**Default to the multi-file path**. It's not a "backup", it's the **main path for long decks and team collaboration**. Reason: every advantage of the single-file architecture (keyboard navigation, print, scale) the multi-file has too, while the scope isolation and verifiability of multi-file are things single-file can't make up for.

### Why is this rule so hard? (Real incident record)

The single-file architecture once stepped on four pitfalls in a row during AI psychology lecture deck production:

1. **CSS specificity override**: `.emotion-slide { display: grid }` (specificity 10) overrode `deck-stage > section { display: none }` (specificity 2), causing all pages to render and stack at the same time.
2. **Shadow DOM slot rules suppressed by outer CSS**: `::slotted(section) { display: none }` couldn't block outer rule overrides; sections refused to hide.
3. **localStorage + hash navigation race**: After refresh, instead of jumping to the hash position, it stayed at the old position recorded in localStorage.
4. **High verification cost**: Must `page.evaluate(d => d.goTo(n))` to screenshot a page, twice as slow as direct `goto(file://.../slides/05-X.html)` and often errors out.

The root cause of all of them is the **single global namespace** — the multi-file architecture eliminates these problems at the physical level.

---

## Path A (default): Multi-File Architecture

### Directory structure

```
MyDeck/
├── index.html              # Copied from assets/deck_index.html, change MANIFEST
├── shared/
│   ├── tokens.css          # Shared design tokens (palette/font sizes/common chrome)
│   └── fonts.html          # <link> import Google Fonts (each page includes)
└── slides/
    ├── 01-cover.html       # Each file is a complete 1920×1080 HTML
    ├── 02-agenda.html
    ├── 03-problem.html
    └── ...
```

### Template skeleton for each slide

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
<meta charset="UTF-8">
<title>P05 · Chapter Title</title>
<link href="https://fonts.googleapis.com/css2?family=..." rel="stylesheet">
<link rel="stylesheet" href="../shared/tokens.css">
<style>
  /* Styles unique to this page. Any class name will not pollute other pages. */
  body { padding: 120px; }
  .my-thing { ... }
</style>
</head>
<body>
  <!-- 1920×1080 content (locked by body's width/height in tokens.css) -->
  <div class="page-header">...</div>
  <div>...</div>
  <div class="page-footer">...</div>
</body>
</html>
```

**Key constraints**:
- `<body>` is the canvas, lay out directly on it. Do not wrap with `<section>` or other wrappers.
- `width: 1920px; height: 1080px` is locked by the `body` rule in `shared/tokens.css`.
- Reference `shared/tokens.css` for shared design tokens (palette, font sizes, page-header/footer, etc.).
- Font `<link>` is written per page (font import alone isn't expensive, and ensures each page is independently openable).

### Aggregator: `deck_index.html`

**Copy directly from `assets/deck_index.html`**. You only need to change one place — the `window.DECK_MANIFEST` array, listing all slide filenames and human-readable labels in order:

```js
window.DECK_MANIFEST = [
  { file: "slides/01-cover.html",    label: "Cover" },
  { file: "slides/02-agenda.html",   label: "Agenda" },
  { file: "slides/03-problem.html",  label: "Problem Statement" },
  // ...
];
```

The aggregator has built-in: keyboard navigation (←/→/Home/End/number keys/P print), scale + letterbox, bottom-right counter, localStorage memory, hash jump, print mode (traverse iframes per page output PDF).

### Single-page verification (this is the killer advantage of multi-file architecture)

Each slide is independent HTML. **After completing one, double-click to open in browser to view**:

```bash
open slides/05-personas.html
```

Playwright screenshot is also direct `goto(file://.../slides/05-personas.html)`, no need for JS jump, and won't be interfered with by other pages' CSS. This makes the workflow cost of "change a little, verify a little" close to zero.

### Parallel development

Split each slide's task to different agents, run simultaneously — HTML files are independent of each other, no conflicts when merging. Long decks using this parallel approach can compress production time to 1/N.

### What to put in `shared/tokens.css`

Only put things **truly shared across pages**:

- CSS variables (palette, font size scale, spacing scale)
- `body { width: 1920px; height: 1080px; }` canvas locking
- `.page-header` / `.page-footer` chrome that every page uses identically

**Do not** stuff single-page layout classes here — that would degrade back to the single-file architecture's global pollution problem.

---

## Path B (small deck): Single File + `deck_stage.js`

Suitable for ≤10 pages, requiring cross-page shared state (e.g., a React tweaks panel to control all pages), or making a pitch deck demo that requires extreme compactness.

### Basic usage

1. Read `assets/deck_stage.js` content, embed in HTML's `<script>` (or `<script src="deck_stage.js">`)
2. Wrap slides with `<deck-stage>` in body
3. 🛑 **The script tag must be after `</deck-stage>`** (see hard constraint below)

```html
<body>

  <deck-stage>
    <section>
      <h1>Slide 1</h1>
    </section>
    <section>
      <h1>Slide 2</h1>
    </section>
  </deck-stage>

  <!-- ✅ Correct: script after deck-stage -->
  <script src="deck_stage.js"></script>

</body>
```

### 🛑 Script position hard constraint (2026-04-20 real-world pitfall)

**You cannot put `<script src="deck_stage.js">` in `<head>`.** Even if it can define `customElements` in `<head>`, when the parser parses the `<deck-stage>` start tag it will trigger `connectedCallback` — at this point the child `<section>`s haven't been parsed yet, `_collectSlides()` gets an empty array, the counter shows `1 / 0`, and all pages stack and render simultaneously.

**Three compliant writings** (any one):

```html
<!-- ✅ Most recommended: script after </deck-stage> -->
</deck-stage>
<script src="deck_stage.js"></script>

<!-- ✅ Also OK: script in head but with defer -->
<head><script src="deck_stage.js" defer></script></head>

<!-- ✅ Also OK: module script naturally defers -->
<head><script src="deck_stage.js" type="module"></script></head>
```

`deck_stage.js` itself has built-in `DOMContentLoaded` delayed collection defense, even if script is in head it won't completely blow up — but `defer` or putting it at the bottom of body is still a cleaner approach, avoiding reliance on the defense branch.

### ⚠️ Single-file architecture CSS pitfalls (must read)

The most common pitfall of single-file architecture — **the `display` property is stolen by single-page styles**.

Common error pose 1 (writing display: flex directly to section):

```css
/* ❌ External CSS specificity 2, overrode shadow DOM's ::slotted(section){display:none} (also 2) */
deck-stage > section {
  display: flex;            /* All pages will stack and render simultaneously! */
  flex-direction: column;
  padding: 80px;
  ...
}
```

Common error pose 2 (section has higher-specificity class):

```css
.emotion-slide { display: grid; }   /* Specificity: 10, worse */
```

Both will cause **all slides to stack and render simultaneously** — the counter may show `1 / 10` pretending normal, but visually page 1 is over page 2 over page 3.

### ✅ Starter CSS (copy directly when starting work, don't trip pitfalls)

**Section itself** only handles "visible/invisible"; **layout (flex/grid etc.) is written on `.active`**:

```css
/* section only defines non-display common styles */
deck-stage > section {
  background: var(--paper);
  padding: 80px 120px;
  overflow: hidden;
  position: relative;
  /* ⚠️ Don't write display here! */
}

/* Lock "non-active is hidden" — specificity + weight double insurance */
deck-stage > section:not(.active) {
  display: none !important;
}

/* Activated page only writes the needed display + layout */
deck-stage > section.active {
  display: flex;
  flex-direction: column;
  justify-content: center;
}

/* Print mode: all pages need to be displayed, override :not(.active) */
@media print {
  deck-stage > section { display: flex !important; }
  deck-stage > section:not(.active) { display: flex !important; }
}
```

Alternative: **Put single-page flex/grid on the inner wrapper `<div>`**, section itself is always just a `display: block/none` switcher. This is the cleanest approach:

```html
<deck-stage>
  <section>
    <div class="slide-content flex-layout">...</div>
  </section>
</deck-stage>
```

### Custom dimensions

```html
<deck-stage width="1080" height="1920">
  <!-- 9:16 portrait -->
</deck-stage>
```

---

## Slide Labels

Both Deck_stage and deck_index will tag each page (counter display). Give them **more meaningful** labels:

**Multi-file**: Write `{ file, label: "04 Problem Statement" }` in `MANIFEST`
**Single file**: Add `<section data-screen-label="04 Problem Statement">` on the section

**Key: Slide numbers start from 1, not 0**.

When the user says "slide 5", they mean the 5th, never array position `[4]`. Humans don't speak in 0-indexed.

---

## Speaker Notes

**Don't add by default**, only add when the user explicitly requests.

Adding speaker notes lets you reduce the text on slides to the minimum, focus on impactful visuals — notes carry the complete script.

### Format

**Multi-file**: Write in `<head>` of `index.html`:

```html
<script type="application/json" id="speaker-notes">
[
  "Script for slide 1...",
  "Script for slide 2...",
  "..."
]
</script>
```

**Single file**: Same position.

### Notes writing essentials

- **Complete**: Not an outline, the actual words you'll speak
- **Conversational**: Like everyday speech, not written language
- **Corresponding**: The Nth in the array corresponds to the Nth slide
- **Length**: 200-400 words is best
- **Emotional line**: Mark stresses, pauses, emphasis points

---

## Slide Design Patterns

### 1. Establish a system (must do)

After exploring design context, **first verbally state the system you'll use**:

```markdown
Deck system:
- Background colors: at most 2 (90% white + 10% dark section divider)
- Typography: display uses Instrument Serif, body uses Geist Sans
- Rhythm: section divider uses full-bleed color + white text, regular slides white background
- Imagery: hero slide uses full-bleed photo, data slide uses chart

I'll proceed with this system, let me know if there are issues.
```

Continue after user confirmation.

### 2. Common slide layouts

- **Title slide**: Solid color background + huge title + subtitle + author/date
- **Section divider**: Color background + chapter number + chapter title
- **Content slide**: White background + title + 1-3 bullet points
- **Data slide**: Title + large chart/number + brief explanation
- **Image slide**: Full-bleed photo + small caption at bottom
- **Quote slide**: Whitespace + huge quote + attribution
- **Two-column**: Left-right comparison (vs / before-after / problem-solution)

Use at most 4-5 layouts in one deck.

### 3. Scale (re-emphasized)

- Body text minimum **24px**, ideally 28-36px
- Title **60-120px**
- Hero text **180-240px**
- Slides are for viewing from 10 meters away, text needs to be large enough

### 4. Visual rhythm

A deck needs **intentional variety**:

- Color rhythm: mostly white background + occasional color section divider + occasional dark fragment
- Density rhythm: a few text-heavy + a few image-heavy + a few quote whitespace
- Font size rhythm: normal titles + occasional huge hero text

**Don't make every slide look the same** — that's a PPT template, not design.

### 5. Spatial breathing (data-dense pages must read)

**The most common pitfall for newcomers**: cramming all the information that can be put into one page.

Information density ≠ effective information delivery. Academic/lecture decks especially need restraint:

- List/matrix page: don't draw all N elements at the same size. Use **primary/secondary layering** — enlarge the 5 you want to talk about today as the leads, shrink the remaining 16 as background hints.
- Big number page: the number itself is the visual lead. Surrounding caption shouldn't exceed 3 lines, otherwise the audience's eyes will jump back and forth.
- Quote page: have whitespace separating the quote and attribution, don't stick them together.

Self-audit against "Is the data the lead?" and "Is the text crowded together?" — revise until the whitespace makes you a little uneasy.

---

## Print to PDF

**Multi-file**: `deck_index.html` has handled the `beforeprint` event, outputting PDF page by page.

**Single-file**: `deck_stage.js` handles the same.

The print styles are already written, no need to write additional `@media print` CSS.

---

## Export to PPTX / PDF (self-service scripts)

HTML-first is the first-class citizen. But users often need PPTX/PDF delivery. Provide two general scripts that **any multi-file deck can use**, located under `scripts/`:

### `export_deck_pdf.mjs` — Export vector PDF (multi-file architecture)

```bash
node scripts/export_deck_pdf.mjs --slides <slides-dir> --out deck.pdf
```

**Features**:
- Text **stays vector** (copyable, searchable)
- 100% visual fidelity (Playwright embedded Chromium renders then prints)
- **No need to change a single word of HTML**
- Each slide independent `page.pdf()`, then merged with `pdf-lib`

**Dependencies**: `npm install playwright pdf-lib`

**Limitation**: PDF cannot edit text again — to change, go back to HTML to change.

### `export_deck_stage_pdf.mjs` — Single-file deck-stage architecture exclusive ⚠️

**When to use**: deck is a single HTML file + `<deck-stage>` web component wrapping N `<section>`s (i.e., Path B architecture). At this point `export_deck_pdf.mjs`'s "one `page.pdf()` per HTML" approach doesn't work, need to use this exclusive script.

```bash
node scripts/export_deck_stage_pdf.mjs --html deck.html --out deck.pdf
```

**Why can't reuse export_deck_pdf.mjs** (2026-04-20 real-world pitfall record):

1. **Shadow DOM beats `!important`**: deck-stage's shadow CSS has `::slotted(section) { display: none }` (only the active one is `display: block`). Even using `@media print { deck-stage > section { display: block !important } }` in light DOM can't suppress it — after `page.pdf()` triggers print media, Chromium's final render only has the active one, resulting in **the whole PDF having only 1 page** (a repeat of the current active slide).

2. **Loop goto each page still outputs only 1 page**: The intuitive solution "navigate once for each `#slide-N` then `page.pdf({pageRanges:'1'})`" also fails — because the print CSS's rule `deck-stage > section { display: block }` outside the shadow DOM is overridden, the final render is always the first in the section list (not the page you navigated to). Result: 17 loop iterations get 17 P01 covers.

3. **absolute child elements run to the next page**: Even if you successfully render all sections, if the section itself is `position: static`, its absolute-positioned `cover-footer`/`slide-footer` will be positioned relative to the initial containing block — when section is print-forced to 1080px height, the absolute footer may be pushed to the next page (manifests as PDF having 1 more page than the section count, with the extra page only containing the orphan footer).

**Fix strategy** (script already implemented):

```js
// After opening HTML, use page.evaluate to extract sections from deck-stage slot,
// directly attach to body's normal div, and inline style to ensure position:relative + fixed dimensions
await page.evaluate(() => {
  const stage = document.querySelector('deck-stage');
  const sections = Array.from(stage.querySelectorAll(':scope > section'));
  document.head.appendChild(Object.assign(document.createElement('style'), {
    textContent: `
      @page { size: 1920px 1080px; margin: 0; }
      html, body { margin: 0 !important; padding: 0 !important; }
      deck-stage { display: none !important; }
    `,
  }));
  const container = document.createElement('div');
  sections.forEach(s => {
    s.style.cssText = 'width:1920px!important;height:1080px!important;display:block!important;position:relative!important;overflow:hidden!important;page-break-after:always!important;break-after:page!important;background:#F7F4EF;margin:0!important;padding:0!important;';
    container.appendChild(s);
  });
  // Last page no page break, avoid trailing blank page
  sections[sections.length - 1].style.pageBreakAfter = 'auto';
  sections[sections.length - 1].style.breakAfter = 'auto';
  document.body.appendChild(container);
});

await page.pdf({ width: '1920px', height: '1080px', printBackground: true, preferCSSPageSize: true });
```

**Why this works**:
- Pull sections from shadow DOM slot to a normal div in light DOM — completely bypass the `::slotted(section) { display: none }` rule
- Inline `position: relative` makes absolute child elements positioned relative to the section, won't overflow
- `page-break-after: always` lets the browser print each section as its own page
- `:last-child` no page break avoids trailing blank page

**Note when verifying with `mdls -name kMDItemNumberOfPages`**: macOS Spotlight metadata is cached; after PDF rewrite, run `mdimport file.pdf` to force refresh, otherwise it shows the old page count. Counting files with `pdfinfo` or `pdftoppm` is the real count.

---

### `export_deck_pptx.mjs` — Export editable PPTX

```bash
# Only mode: text boxes natively editable (fonts will fall back to system fonts)
node scripts/export_deck_pptx.mjs --slides <dir> --out deck.pptx
```

How it works: `html2pptx` reads computedStyle element-by-element to translate the DOM into PowerPoint objects (text frame / shape / picture). Text becomes real text boxes, double-click in PPT to edit.

**Hard constraints** (HTML must satisfy, otherwise that page is skipped, detailed explanation see `references/editable-pptx.md`):
- All text must be in `<p>`/`<h1>`-`<h6>`/`<ul>`/`<ol>` (forbidden bare text div)
- `<p>`/`<h*>` tags themselves cannot have background/border/shadow (put on outer div)
- Don't use `::before`/`::after` to insert decorative text (pseudo-elements can't be extracted)
- inline elements (span/em/strong) cannot have margins
- Don't use CSS gradient (not renderable)
- div doesn't use `background-image` (use `<img>`)

The script has a built-in **automatic preprocessor** — automatically wraps "bare text in leaf divs" into `<p>` (preserving class). This solves the most common violation (bare text). But other violations (border on p, margin on span, etc.) still require HTML source compliance.

**Font fallback caveat**:
- Playwright uses webfonts to measure text-box dimensions; PowerPoint/Keynote uses local fonts to render
- When the two differ there will be **overflow or misalignment** — every page must be visually checked
- Recommend installing the fonts used in HTML on the target machine, or fallback to `system-ui`

**Don't go this path for visual-priority scenarios** → switch to `export_deck_pdf.mjs` to output PDF. PDF visuals are 100% faithful, vector, cross-platform, text searchable — it's the true destination of visual-priority decks, not some "non-editable compromise".

### Make HTML export-friendly from the start

For the most stable deck performance: **write HTML per the 4 hard constraints of editable from the start**. This way `export_deck_pptx.mjs` can directly pass everything. The extra cost is small:

```html
<!-- ❌ Bad -->
<div class="title">Key findings</div>

<!-- ✅ Good (p wrap, class inheritance) -->
<p class="title">Key findings</p>

<!-- ❌ Bad (border on p) -->
<p class="stat" style="border-left: 3px solid red;">41%</p>

<!-- ✅ Good (border on outer div) -->
<div class="stat-wrap" style="border-left: 3px solid red;">
  <p class="stat">41%</p>
</div>
```

### When to choose which

| Scenario | Recommended |
|------|------|
| For organizer/archive | **PDF** (universal, high fidelity, text searchable) |
| Send to collaborators for them to tweak text | **PPTX editable** (accept font fallback) |
| Presenting on-site, not changing content | **PDF** (vector fidelity, cross-platform) |
| HTML is the primary presentation medium | Play directly in browser, export is just backup |

## Deep path for export to editable PPTX (long-term projects only)

If your deck will be maintained long-term, modified repeatedly, with team collaboration — recommend **writing HTML per html2pptx constraints from the start**, this way `export_deck_pptx.mjs` can directly pass everything. See `references/editable-pptx.md` (4 hard constraints + HTML template + common errors quick reference + fallback flow for existing visual drafts).

---

## Common Issues

**Multi-file: page in iframe doesn't open / blank screen**
→ Check whether the `file` path in `MANIFEST` is correct relative to `index.html`. Use browser DevTools to see if iframe's src can be accessed directly.

**Multi-file: a page's styles conflict with another page**
→ Impossible (iframe isolation). If you feel a conflict, that's caching — Cmd+Shift+R force refresh.

**Single file: multiple slides render and stack simultaneously**
→ CSS specificity issue. See "Single-file architecture CSS pitfalls" section above.

**Single file: scaling looks wrong**
→ Check if all slides are directly attached to `<deck-stage>` as `<section>`. Cannot wrap with `<div>` in between.

**Single file: want to jump to a specific slide**
→ Add hash to URL: `index.html#slide-5` jumps to slide 5.

**Both architectures: text position inconsistent on different screens**
→ Use fixed dimensions (1920×1080) and `px` units, don't use `vw`/`vh` or `%`. Scaling handled uniformly.

---

## Verification Checklist (must pass after deck is done)

1. [ ] Open `index.html` (or main HTML) directly in browser, check no broken images on home page, fonts loaded
2. [ ] Press → key to flip to each page, no blank pages, no layout misalignment
3. [ ] Press P key for print preview, each page is exactly one A4 (or 1920×1080) without cropping
4. [ ] Randomly pick 3 pages Cmd+Shift+R force refresh, localStorage memory works normally
5. [ ] Playwright batch screenshot (single-page architecture: traverse `slides/*.html`; single-file architecture: use goTo to switch), human visual review
6. [ ] Search for `TODO` / `placeholder` remnants, confirm all cleaned up
