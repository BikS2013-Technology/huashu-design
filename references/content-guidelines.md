# Content Guidelines: Anti-AI-Slop, Content Standards, Scale Specs

The trap most easily fallen into in AI design. This is a "what NOT to do" list, more important than "what to do" — because AI slop is the default, and if you don't actively avoid it, it will happen.

## Complete AI Slop Blacklist

### Visual traps

**❌ Aggressive gradient backgrounds**
- Purple → pink → blue full-screen gradient (the typical signature of AI-generated webpages)
- Rainbow gradients in any direction
- Mesh gradient covering the whole background
- ✅ If you must use gradients: subtle, single-color family, intentional accents (e.g., button hover)

**❌ Rounded cards + left-border accent color**
```css
/* This is the typical signature of an AI-flavored card */
.card {
  border-radius: 12px;
  border-left: 4px solid #3b82f6;
  padding: 16px;
}
```
This kind of card floods AI-generated dashboards. Want emphasis? Use a more design-oriented approach: background-color contrast, font-weight/size contrast, plain dividers, or just don't separate into cards.

**❌ Emoji decorations**
Unless the brand itself uses emojis (e.g., Notion, Slack), don't put emojis on the UI. **Especially don't**:
- 🚀 ⚡️ ✨ 🎯 💡 before headings
- ✅ in feature lists
- → in CTA buttons (a standalone arrow is OK; emoji arrows are not)

If you don't have icons, use a real icon library (Lucide/Heroicons/Phosphor), or use placeholders.

**❌ SVG-drawn imagery**
Don't try to draw with SVG: people, scenes, devices, objects, abstract art. AI-drawn SVG imagery has an instantly recognizable AI flavor — childish and cheap. **A gray rectangle + the text label "illustration placeholder 1200×800" is 100× better than a poorly executed SVG hero illustration.**

The only scenarios where SVG is acceptable:
- True icons (16×16 to 32×32 level)
- Geometric shapes as decorative elements
- Charts for data viz

**❌ Excessive iconography**
Not every heading/feature/section needs an icon. Overusing icons makes the interface look like a toy. Less is more.

**❌ "Data slop"**
Fabricated stats as decoration:
- "10,000+ happy customers" (you don't even know if there are any)
- "99.9% uptime" (don't write it without real data)
- Decorative "metric cards" composed of icon+number+phrase
- Mock tables with fake data dressed up flashily

If there's no real data, leave a placeholder or ask the user.

**❌ "Quote slop"**
Fabricated user testimonials, celebrity quotations decorating pages. Leave a placeholder and ask the user for a real quote.

### Font traps

**❌ Avoid these overused fonts**:
- Inter (the default for AI-generated webpages)
- Roboto
- Arial / Helvetica
- Pure system font stack
- Fraunces (AI discovered it and overused it)
- Space Grotesk (AI's recent favorite)

**✅ Use display+body pairings with character**. Direction inspirations:
- Serif display + sans-serif body (editorial feel)
- Mono display + sans body (technical feel)
- Heavy display + light body (contrast)
- Variable font for hero weight animation

Font resources:
- Lesser-known good options on Google Fonts (Instrument Serif, Cormorant, Bricolage Grotesque, JetBrains Mono)
- Open-source font sites (Fraunces' sibling fonts, Adobe Fonts)
- Don't invent font names out of thin air

### Color traps

**❌ Inventing colors out of thin air**
Don't design an entire unfamiliar color set from scratch. It's usually inharmonious.

**✅ Strategy**:
1. Has brand color → use brand color, fill missing color tokens with oklch interpolation
2. No brand color but has reference → eyedrop colors from reference product screenshots
3. Completely from scratch → choose a known color system (Radix Colors / Tailwind default palette / Anthropic brand), don't tune your own

**Defining colors with oklch** is the most modern approach:
```css
:root {
  --primary: oklch(0.65 0.18 25);      /* warm terracotta */
  --primary-light: oklch(0.85 0.08 25); /* same family, light */
  --primary-dark: oklch(0.45 0.20 25);  /* same family, dark */
}
```
oklch guarantees that hue doesn't drift when adjusting lightness, much better than hsl.

**❌ Casually inverting colors for night mode**
Not a simple color invert. Good dark mode requires re-tuning saturation, contrast, accent colors. If you don't want to do dark mode, don't do it.

### Layout traps

**❌ Excessive bento grid overflow**
Every AI-generated landing page wants to do bento. Unless your information structure genuinely fits bento, use other layouts.

**❌ Big hero + 3-column features + testimonials + CTA**
This landing page template has been used to death. If you want to innovate, actually innovate.

**❌ Every card in a card grid looks the same**
Asymmetric, different-sized cards, some with images and some text-only, some spanning columns — this is what real designer work looks like.

## Content Guidelines

### 1. Don't add filler content

Every element must earn its place. Empty space is a design problem solved by **composition** (contrast, rhythm, whitespace), **not** by filling with content.

**Questions to identify filler**:
- If you removed this content, would the design get worse? If "no", remove it.
- What real problem does this element solve? If it's "to make the page less empty", delete it.
- Does this stat/quote/feature have real data backing it? If not, don't make it up.

"One thousand no's for every yes."

### 2. Ask before adding material

You think adding another paragraph/page/section would be better? Ask the user first, don't add unilaterally.

Reasons:
- Users know their audience better than you
- Adding content has a cost; users may not want it
- Adding unilaterally violates the "junior designer reporting to client" relationship

### 3. Create a system up front

After exploring the design context, **state verbally first the system you want to use** and let the user confirm:

```markdown
My design system:
- Color: #1A1A1A primary + #F0EEE6 background + #D97757 accent (from your brand)
- Typography: Instrument Serif for display + Geist Sans for body
- Rhythm: section title uses full-bleed colored background + white text; regular section uses white background
- Imagery: hero uses full-bleed photo, feature section uses placeholder waiting for you to provide
- Use at most 2 background colors, avoid clutter

Confirm this direction and I'll start.
```

After user confirms, then act. This check-in avoids "halfway through finding out the direction was wrong".

## Scale Specifications

### Slides (1920×1080)

- Body text minimum **24px**, ideal 28-36px
- Title 60-120px
- Section title 80-160px
- Hero headline can use 180-240px large text
- Never use <24px text on slides

### Print documents

- Body text minimum **10pt** (≈13.3px), ideal 11-12pt
- Title 18-36pt
- Caption 8-9pt

### Web and mobile

- Body text minimum **14px** (16px for senior-friendly)
- Mobile body **16px** (avoid iOS auto-zoom)
- Hit target (clickable element) minimum **44×44px**
- Line height 1.5-1.7 (Chinese 1.7-1.8)

### Contrast

- Body text vs background **at least 4.5:1** (WCAG AA)
- Large text vs background **at least 3:1**
- Use Chrome DevTools' accessibility tool to check

## CSS Power Tools

**Advanced CSS features** are a designer's friend, use boldly:

### Typography

```css
/* Make heading wrapping more natural, no orphan word on the last line */
h1, h2, h3 { text-wrap: balance; }

/* Body text wrapping, avoid widows and orphans */
p { text-wrap: pretty; }

/* Chinese typography essentials: punctuation squeezing, line-start/end control */
p { 
  text-spacing-trim: space-all;
  hanging-punctuation: first;
}
```

### Layout

```css
/* CSS Grid + named areas = explosive readability */
.layout {
  display: grid;
  grid-template-areas:
    "header header"
    "sidebar main"
    "footer footer";
  grid-template-columns: 240px 1fr;
  grid-template-rows: auto 1fr auto;
}

/* Subgrid for aligning card content */
.card { display: grid; grid-template-rows: subgrid; }
```

### Visual effects

```css
/* Designer scrollbar */
* { scrollbar-width: thin; scrollbar-color: #666 transparent; }

/* Glassmorphism (use sparingly) */
.glass {
  backdrop-filter: blur(20px) saturate(150%);
  background: color-mix(in oklch, white 70%, transparent);
}

/* View transitions API for silky page switching */
@view-transition { navigation: auto; }
```

### Interaction

```css
/* :has() selector makes conditional styling easy */
.card:has(img) { padding-top: 0; } /* cards with images have no top padding */

/* container queries make components truly responsive */
@container (min-width: 500px) { ... }

/* New color-mix function */
.button:hover {
  background: color-mix(in oklch, var(--primary) 85%, black);
}
```

## Decision Quick Reference: When You Hesitate

- Want to add a gradient? → Probably don't add it
- Want to add an emoji? → Don't add it
- Want to add rounded corners + border-left accent to a card? → Don't, use other approaches
- Want to use SVG to draw a hero illustration? → Don't draw it, use a placeholder
- Want to add a quote as decoration? → Ask the user first if they have a real quote
- Want to add a row of icon features? → Ask first if icons are needed; probably not
- Use Inter? → Switch to something with more character
- Use a purple gradient? → Switch to a color scheme with rationale

**When you feel "adding it would look better" — that's usually a sign of AI slop**. Make the simplest version first, only add when the user requests.
