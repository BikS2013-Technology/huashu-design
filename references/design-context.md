# Design Context: Starting from Existing Context

**This is the one thing that matters most for this skill.**

Good hi-fi design always grows out of existing design context. **Doing hi-fi from thin air is the last resort and will inevitably produce generic work**. So at the start of every design task, ask first: is there anything to reference?

## What Is Design Context

In priority order, high to low:

### 1. The User's Design System / UI Kit
The component library, color tokens, typography spec, and icon system that already exist in the user's own product. **The most ideal situation**.

### 2. The User's Codebase
If the user gave you a code repository, it contains living component implementations. Read those component files:
- `theme.ts` / `colors.ts` / `tokens.css` / `_variables.scss`
- Specific components (Button.tsx, Card.tsx)
- Layout scaffold (App.tsx, MainLayout.tsx)
- Global stylesheets

**Read the code and copy exact values**: hex codes, spacing scale, font stack, border radius. Don't redraw from memory.

### 3. The User's Already-Released Product
If the user has a live product but didn't provide code, use Playwright or have the user provide screenshots.

```bash
# Use Playwright to screenshot a public URL
npx playwright screenshot https://example.com screenshot.png --viewport-size=1920,1080
```

Lets you see the real visual vocabulary.

### 4. Brand Guidelines / Logo / Existing Assets
The user might have: logo files, brand color spec, marketing materials, slide templates. These are all context.

### 5. Competitor References
The user says "like XX website"—have them provide the URL or screenshot. **Do not** work from a vague impression in your training data.

### 6. Known Design Systems (fallback)
If none of the above exists, use a recognized design system as a base:
- Apple HIG
- Material Design 3
- Radix Colors (palette)
- shadcn/ui (components)
- Tailwind default palette

Tell the user clearly what you're using, so they know it's a starting point not a final draft.

## The Process for Obtaining Context

### Step 1: Ask the User

The mandatory question checklist at the start of a task (from `workflow.md`):

```markdown
1. Do you have an existing design system / UI kit / component library? Where?
2. Are there brand guidelines, color/font specifications?
3. Can you give me screenshots or URLs of existing products?
4. Is there a codebase I can read?
```

### Step 2: When the User Says "No," Help Them Find It

Don't give up directly. Try:

```markdown
Let me see if there are any clues:
- Do your previous projects have related designs?
- What colors/typography does your company's marketing site use?
- What style is your product's logo? Can you give me one?
- Are there any products you admire as a reference?
```

### Step 3: Read All the Context You Can Find

If the user gave a codebase path, you read:
1. **First list the file structure**: find files related to style/theme/component
2. **Read theme/token files**: lift specific hex/px values
3. **Read 2-3 representative components**: see the visual vocabulary (hover state, shadow, border, padding pattern)
4. **Read global stylesheet**: base reset, font loading
5. **If there's a Figma link/screenshot**: look at the image, but **trust the code more**

**Important**: **Do not** glance once and work from impression. You've only really lifted the system if you've come away with 30+ specific values.

### Step 4: Vocalize the System You're Going to Use

After reading the context, tell the user the system you're going to use:

```markdown
Based on your codebase and product screenshots, the design system I've extracted:

**Colors**
- Primary: #C27558 (from tokens.css)
- Background: #FDF9F0
- Text: #1A1A1A
- Muted: #6B6B6B

**Typography**
- Display: Instrument Serif (from @font-face in global.css)
- Body: Geist Sans
- Mono: JetBrains Mono

**Spacing** (from your scale system)
- 4, 8, 12, 16, 24, 32, 48, 64

**Shadow pattern**
- `0 1px 2px rgba(0,0,0,0.04)` (subtle card)
- `0 10px 40px rgba(0,0,0,0.1)` (elevated modal)

**Border-radius**
- Small components 4px, cards 12px, buttons 8px

**Component vocabulary**
- Button: filled primary, outlined secondary, ghost tertiary, all 8px corner radius
- Card: white background, subtle shadow, no border

I'll start with this system. Confirm OK?
```

Only start working after the user confirms.

## Designing from Thin Air (Fallback When There's No Context)

**Strong warning**: The output quality will drop noticeably in this scenario. Tell the user clearly.

```markdown
You don't have design context, so I can only work from general intuition.
The output will be "looks OK but lacks distinctiveness."
Are you willing to continue, or supply some reference material first?
```

If the user insists you proceed, make decisions in this order:

### 1. Choose an Aesthetic Direction
Don't give a generic result. Pick a definite direction:
- brutally minimal
- editorial/magazine
- brutalist/raw
- organic/natural
- luxury/refined
- playful/toy
- retro-futuristic
- soft/pastel

Tell the user which one you chose.

### 2. Pick a Known Design System as the Skeleton
- Use Radix Colors for palette (https://www.radix-ui.com/colors)
- Use shadcn/ui for component vocabulary (https://ui.shadcn.com)
- Use Tailwind spacing scale (multiples of 4)

### 3. Choose a Distinctive Font Pairing

Don't use Inter/Roboto. Recommended pairings (free from Google Fonts):
- Instrument Serif + Geist Sans
- Cormorant Garamond + Inter Tight
- Bricolage Grotesque + Söhne (paid)
- Fraunces + Work Sans (note: Fraunces has been overused by AI)
- JetBrains Mono + Geist Sans (technical feel)

### 4. Every Key Decision Has Reasoning

Don't choose silently. Write it in HTML comments:

```html
<!--
Design decisions:
- Primary color: warm terracotta (oklch 0.65 0.18 25) — fits the "editorial" direction  
- Display: Instrument Serif for humanist, literary feel
- Body: Geist Sans for cleanness contrast
- No gradients — committed to minimal, no AI slop
- Spacing: 8px base, golden ratio friendly (8/13/21/34)
-->
```

## Import Strategy (When the User Gave a Codebase)

If the user says "import this codebase as a reference":

### Small (<50 files)
Read everything, internalize the context.

### Medium (50-500 files)
Focus on:
- `src/components/` or `components/`
- All files related to styles/tokens/theme
- 2-3 representative full-page components (Home.tsx, Dashboard.tsx)

### Large (>500 files)
Have the user point out the focus:
- "I want to do the settings page" → read existing settings-related
- "I want to do a new feature" → read overall shell + closest reference
- Don't pursue completeness; pursue accuracy

## Working with Figma / Design Drafts

If the user provides a Figma link:

- **Don't** expect that you can directly "convert Figma to HTML"—that requires extra tools
- Figma links are usually not publicly accessible
- Have the user: export as **screenshots** to send you + tell you specific color/spacing values

If only a Figma screenshot is provided, tell the user:
- I can see the visuals, but I can't extract precise values
- Please tell me the key numbers (hex, px), or export as code (Figma supports this)

## Final Reminder

**The ceiling on a project's design quality is determined by the quality of context you receive**.

Spending 10 minutes collecting context is more valuable than spending 1 hour drawing hi-fi from thin air.

**When facing a no-context situation, prioritize asking the user for it rather than forcing through**.
