# Design Philosophy Showcases — Sample Asset Index

> 8 scenarios x 3 styles = 24 prebuilt design samples
> Used during Phase 3 design recommendation to directly demonstrate "what this style actually looks like"

## Style Overview

| Code | School | Style Name | Visual Character |
|------|--------|------------|------------------|
| **Pentagram** | Information Architecture | Pentagram / Michael Bierut | Black & white restraint, Swiss grid, strong typographic hierarchy, #E63946 red accent |
| **Build** | Minimalism | Build Studio | Luxury-grade whitespace (70%+), subtle font weights (200-600), #D4A574 warm gold, refined |
| **Takram** | Eastern Philosophy | Takram | Soft tech feel, natural palette (beige/grey/green), rounded corners, charts as art |

## Scenario Quick Reference

### Content Design Scenarios

| # | Scenario | Specs | Pentagram | Build | Takram |
|---|----------|-------|-----------|-------|--------|
| 1 | Social Media Cover | 1200x510 | `cover/cover-pentagram` | `cover/cover-build` | `cover/cover-takram` |
| 2 | PPT Data Slide | 1920x1080 | `ppt/ppt-pentagram` | `ppt/ppt-build` | `ppt/ppt-takram` |
| 3 | Vertical Infographic | 1080x1920 | `infographic/infographic-pentagram` | `infographic/infographic-build` | `infographic/infographic-takram` |

### Website Design Scenarios

| # | Scenario | Specs | Pentagram | Build | Takram |
|---|----------|-------|-----------|-------|--------|
| 4 | Personal Homepage | 1440x900 | `website-homepage/homepage-pentagram` | `website-homepage/homepage-build` | `website-homepage/homepage-takram` |
| 5 | AI Directory | 1440x900 | `website-ai-nav/ainav-pentagram` | `website-ai-nav/ainav-build` | `website-ai-nav/ainav-takram` |
| 6 | AI Writing Tool | 1440x900 | `website-ai-writing/aiwriting-pentagram` | `website-ai-writing/aiwriting-build` | `website-ai-writing/aiwriting-takram` |
| 7 | SaaS Landing Page | 1440x900 | `website-saas/saas-pentagram` | `website-saas/saas-build` | `website-saas/saas-takram` |
| 8 | Developer Documentation | 1440x900 | `website-devdocs/devdocs-pentagram` | `website-devdocs/devdocs-build` | `website-devdocs/devdocs-takram` |

> Each entry has both a `.html` (source) and a `.png` (screenshot) file

## Usage Guide

### Referencing in Phase 3 Recommendations
After recommending a design direction, you can show the prebuilt screenshot for the matching scenario:
```
"This is what the Pentagram style looks like for a social media cover -> [show cover/cover-pentagram.png]"
"Takram style applied to a PPT data slide feels like this -> [show ppt/ppt-takram.png]"
```

### Scenario Matching Priority
1. The user's scenario matches exactly -> show the corresponding scenario directly
2. No exact match but similar type -> show the closest scenario (e.g., "product website" -> show the SaaS landing page)
3. No match at all -> skip the prebuilt sample and proceed straight to Phase 3.5 live generation

### Side-by-Side Style Comparison
The 3 styles for the same scenario are great for side-by-side display, helping users compare directly:
- "Here is the same social media cover, executed in 3 different styles"
- Display order: Pentagram (rational restraint) -> Build (luxury minimalism) -> Takram (soft warmth)

## Content Details

### Social Media Cover (cover/)
- Topic: Claude Code Agent workflow — 8 parallel agent architecture
- Pentagram: Massive red "8" + Swiss grid lines + data bars
- Build: Ultra-thin "Agent" floating in 70% whitespace + warm gold fine lines
- Takram: 8-node radial flow diagram as art piece + beige background

### PPT Data Slide (ppt/)
- Topic: GLM-4.7 open-source model coding breakthrough (AIME 95.7 / SWE-bench 73.8% / tau-squared-Bench 87.4)
- Pentagram: 260px "95.7" anchor + red/grey/light-grey comparative bar chart
- Build: Three groups of 120px ultra-thin numbers floating + warm gold gradient comparison bars
- Takram: SVG radar chart + tri-color overlay + rounded data cards

### Vertical Infographic (infographic/)
- Topic: AI memory system CLAUDE.md optimized from 93KB down to 22KB
- Pentagram: Massive "93->22" numerals + numbered blocks + CSS data bars
- Build: Extreme whitespace + soft-shadow cards + warm gold connector lines
- Takram: SVG ring chart + organic curved flow diagram + frosted glass cards

### Personal Homepage (website-homepage/)
- Topic: Independent developer Alex Chen's portfolio homepage
- Pentagram: 112px name + Swiss grid columns + editorial numbers
- Build: Glass-morphism navigation + floating stat cards + ultra-thin font weights
- Takram: Paper texture + small round avatar + hairline dividers + asymmetric layout

### AI Directory Site (website-ai-nav/)
- Topic: AI Compass — directory of 500+ AI tools
- Pentagram: Square-corner search box + numbered tool list + uppercase category labels
- Build: Rounded search box + refined white tool cards + pill-shaped tags
- Takram: Organic offset card layout + soft category labels + chart-like connectors

### AI Writing Tool (website-ai-writing/)
- Topic: Inkwell — AI writing assistant
- Pentagram: 86px headline + wireframe editor mockup + grid feature columns
- Build: Floating editor card + warm gold CTA + luxurious writing experience
- Takram: Poetic serif headline + organic editor + flow diagram

### SaaS Landing Page (website-saas/)
- Topic: Meridian — business intelligence analytics platform
- Pentagram: Black & white split layout + structured dashboard + 140px "3x" anchor
- Build: Floating dashboard cards + SVG area chart + warm gold gradient
- Takram: Rounded bar chart + flow nodes + soft earth tones

### Developer Documentation (website-devdocs/)
- Topic: Nexus API — unified AI model gateway
- Pentagram: Left sidebar nav + square-corner code blocks + red string highlighting
- Build: Centered floating code card + soft shadows + warm gold icons
- Takram: Beige code blocks + flow diagram connectors + dashed feature cards

## File Statistics

- HTML source files: 24
- PNG screenshots: 24
- Total assets: 48 files

---

**Version**: v1.0
**Created**: 2026-02-13
**Used for**: design-philosophy skill, Phase 3 recommendation step
