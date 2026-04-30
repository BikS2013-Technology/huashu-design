# Design Critique In-Depth Guide

> Detailed reference for Phase 7. Provides scoring criteria, scenario-specific emphasis, and a common issues checklist.

---

## Scoring Criteria in Detail

### 1. Philosophy Alignment

| Score | Criterion |
|------|------|
| 9-10 | The design perfectly embodies the core spirit of the chosen philosophy; every detail has a philosophical basis |
| 7-8 | Overall direction is correct, core characteristics are present, individual details deviate |
| 5-6 | Intent is visible, but execution mixes in elements from other styles—not pure enough |
| 3-4 | Only superficial imitation, no understanding of the philosophy's core |
| 1-2 | Essentially unrelated to the chosen philosophy |

**Critique points**:
- Were the signature techniques of that designer/institution used?
- Do colors, typography, and layout align with that philosophical system?
- Are there "self-contradictory" elements? (e.g., choosing Kenya Hara but stuffing it full of content)

### 2. Visual Hierarchy

| Score | Criterion |
|------|------|
| 9-10 | The user's gaze flows naturally along the designer's intent; information acquisition has zero friction |
| 7-8 | Primary/secondary relationships are clear, occasionally 1-2 places with unclear hierarchy |
| 5-6 | Title and body can be distinguished, but middle hierarchy is muddled |
| 3-4 | Information is laid flat, no clear visual entry point |
| 1-2 | Chaotic, the user doesn't know where to look first |

**Critique points**:
- Is the font size contrast between heading and body sufficient? (At least 2.5x)
- Do color/weight/size establish 3-4 clear levels?
- Does negative space guide the gaze?
- "Squint test": squint your eyes—is the hierarchy still clear?

### 3. Craft Quality

| Score | Criterion |
|------|------|
| 9-10 | Pixel-precise; alignment, spacing, and color have no flaws |
| 7-8 | Overall refined, with 1-2 minor alignment/spacing issues |
| 5-6 | Basically aligned, but spacing is inconsistent and color use isn't systematic enough |
| 3-4 | Obvious alignment errors, chaotic spacing, too many colors |
| 1-2 | Rough, looks like a draft |

**Critique points**:
- Is a unified spacing system used (e.g., 8pt grid)?
- Is the spacing of similar elements consistent?
- Is the number of colors controlled? (Usually no more than 3-4)
- Is the font family unified? (Usually no more than 2)
- Is edge alignment precise?

### 4. Functionality

| Score | Criterion |
|------|------|
| 9-10 | Every design element serves the goal; zero redundancy |
| 7-8 | Function-oriented and clear, with a small amount of decoration that could be cut |
| 5-6 | Basically usable, but obvious decorative elements distract attention |
| 3-4 | Form over function; users have to work to find information |
| 1-2 | Completely overwhelmed by decoration, has lost the ability to convey information |

**Critique points**:
- If you delete any one element, would the design get worse? (If not, it should be deleted)
- Are CTAs/key information in the most prominent positions?
- Are there elements that are "added because they look nice"?
- Does information density match the medium? (PPT shouldn't be too dense; PDF can be denser)

### 5. Originality

| Score | Criterion |
|------|------|
| 9-10 | Refreshing; finds unique expression within that philosophical framework |
| 7-8 | Has its own ideas, not just template application |
| 5-6 | Conventional, looks like a template |
| 3-4 | Heavy use of clichés (e.g., gradient orbs representing AI) |
| 1-2 | Completely a template or stock material collage |

**Critique points**:
- Are common clichés avoided? (See "Common Issues Checklist" below)
- Is there personal expression while still following the design philosophy?
- Are there "unexpected but very reasonable" design decisions?

---

## Scenario-Specific Critique Emphasis

Different output types have different critique priorities:

| Scenario | Most Important Dimension | Secondarily Important | Can Relax |
|------|-----------|--------|--------|
| WeChat public account cover/illustrations | Originality, Visual Hierarchy | Philosophy Alignment | Functionality (single image doesn't involve interaction) |
| Infographic | Functionality, Visual Hierarchy | Craft Quality | Originality (accuracy first) |
| PPT/Keynote | Visual Hierarchy, Functionality | Craft Quality | Originality (clarity first) |
| PDF/Whitepaper | Craft Quality, Functionality | Visual Hierarchy | Originality (professionalism first) |
| Landing page/Website | Functionality, Visual Hierarchy | Originality | — (comprehensive requirements) |
| App UI | Functionality, Craft Quality | Visual Hierarchy | Philosophy Alignment (usability first) |
| Xiaohongshu illustration | Originality, Visual Hierarchy | Philosophy Alignment | Craft Quality (atmosphere first) |

---

## Top 10 Common Design Problems

### 1. AI Tech Cliché
**Issue**: Gradient orbs, digital rain, blue circuit boards, robot faces
**Why it's a problem**: Users are visually fatigued by these and can't distinguish you from others
**Fix**: Use abstract metaphors instead of literal symbols (e.g., use the metaphor of "dialogue" rather than chat bubble icons)

### 2. Insufficient Font Size Hierarchy
**Issue**: Heading and body are too close in size (<2.5x)
**Why it's a problem**: Users can't quickly locate key information
**Fix**: Headings at least 3x of body (e.g., body 16px → heading 48-64px)

### 3. Too Many Colors
**Issue**: Using 5+ colors with no primary/secondary
**Why it's a problem**: Visual chaos, weak brand feel
**Fix**: Limit to 1 primary + 1 secondary + 1 accent + grayscale

### 4. Inconsistent Spacing
**Issue**: Element spacing is arbitrary, no system
**Why it's a problem**: Looks unprofessional, chaotic visual rhythm
**Fix**: Establish an 8pt grid system (spacing only uses 8/16/24/32/48/64px)

### 5. Insufficient Negative Space
**Issue**: All space filled with content
**Why it's a problem**: Information crowding causes reading fatigue, actually lowering information transfer efficiency
**Fix**: Negative space should be at least 40% of total area (60%+ for minimal style)

### 6. Too Many Fonts
**Issue**: Using 3+ fonts
**Why it's a problem**: Visual noise, weakens unified feel
**Fix**: At most 2 fonts (1 heading + 1 body), use weight and size to create variation

### 7. Inconsistent Alignment
**Issue**: Some left-aligned, some centered, some right-aligned
**Why it's a problem**: Destroys visual order
**Fix**: Choose one alignment method (left-align recommended), unify globally

### 8. Decoration Over Content
**Issue**: Background patterns/gradients/shadows steal the spotlight from main content
**Why it's a problem**: Cart before the horse—users come to see info, not decoration
**Fix**: "If I delete this decoration, would the design get worse?" If not, delete it

### 9. Cyber Neon Overuse
**Issue**: Deep blue background (#0D1117) + neon glow effects
**Why it's a problem**: Default aesthetic forbidden zone (this skill's taste baseline), and has become one of the biggest clichés—users can override per their own brand
**Fix**: Choose more distinctive color schemes (reference the color systems of the 20 styles)

### 10. Information Density Mismatched with Medium
**Issue**: Stuffing a whole page of text into PPT / cramming 10 elements into a cover image
**Why it's a problem**: Different media have different optimal information densities
**Fix**:
- PPT: 1 core point per slide
- Cover image: 1 visual focal point
- Infographic: layered display
- PDF: can be denser, but needs clear navigation

---

## Critique Output Template

```
## Design Critique Report

**Overall Score**: X.X/10 [Excellent (8+)/Good (6-7.9)/Needs Improvement (4-5.9)/Failing (<4)]

**Sub-scores**:
- Philosophy Alignment: X/10 [one-sentence note]
- Visual Hierarchy: X/10 [one-sentence note]
- Craft Quality: X/10 [one-sentence note]
- Functionality: X/10 [one-sentence note]
- Originality: X/10 [one-sentence note]

### Strengths (Keep)
- [Specifically point out what's done well, described in design language]

### Issues (Fix)
[Sorted by severity]

**1. [Issue name]** — ⚠️ Critical / ⚡ Important / 💡 Optimization
- Current: [describe the status]
- Issue: [why this is a problem]
- Fix: [specific action with values]

### Quick Wins Checklist
If you only have 5 minutes, prioritize these 3:
- [ ] [Most impactful fix]
- [ ] [Second most important fix]
- [ ] [Third most important fix]
```

---

**Version**: v1.0
**Updated**: 2026-02-13
