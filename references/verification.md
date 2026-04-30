# Verification: Output Validation Process

Some design-agent native environments (e.g., Claude.ai Artifacts) have a built-in `fork_verifier_agent` that spawns a subagent to use an iframe screenshot for checking. Most agent environments (Claude Code / Codex / Cursor / Trae / etc.) don't have this built-in capability — Playwright manually covers the same verification scenarios.

## Verification checklist

After every HTML output, run through this checklist:

### 1. Browser render check (mandatory)

The most basic: **does the HTML even open?** On macOS:

```bash
open -a "Google Chrome" "/path/to/your/design.html"
```

Or use Playwright screenshot (next section).

### 2. Console error check

The most common HTML-file issue is a JS error producing a white screen. Run via Playwright:

```bash
python ~/.claude/skills/claude-design/scripts/verify.py path/to/design.html
```

This script:
1. Opens the HTML in headless chromium
2. Saves a screenshot to the project directory
3. Captures console errors
4. Reports status

See `scripts/verify.py`.

### 3. Multi-viewport check

For responsive designs, screenshot multiple viewports:

```bash
python verify.py design.html --viewports 1920x1080,1440x900,768x1024,375x667
```

### 4. Interaction check

Tweaks, animations, button switching — default static screenshots can't see those. **Recommended: have the user click through in their own browser**, or record a video with Playwright:

```python
page.video.record('interaction.mp4')
```

### 5. Per-slide check for decks

For deck-style HTML, screenshot one by one:

```bash
python verify.py deck.html --slides 10  # capture the first 10
```

Generates `deck-slide-01.png`, `deck-slide-02.png`... for quick browsing.

## Playwright Setup

First-time setup:

```bash
# If not installed yet
npm install -g playwright
npx playwright install chromium

# Or the Python version
pip install playwright
playwright install chromium
```

If the user already has Playwright installed globally, just use it.

## Screenshot best practices

### Full-page screenshot

```python
page.screenshot(path='full.png', full_page=True)
```

### Viewport screenshot

```python
page.screenshot(path='viewport.png')  # default: visible area only
```

### Specific element screenshot

```python
element = page.query_selector('.hero-section')
element.screenshot(path='hero.png')
```

### Hi-DPI screenshot

```python
page = browser.new_page(device_scale_factor=2)  # retina
```

### Wait for animation to finish before capturing

```python
page.wait_for_timeout(2000)  # wait 2s for animation to settle
page.screenshot(...)
```

## Sending screenshots to the user

### Open the local screenshot directly

```bash
open screenshot.png
```

The user views it in their own Preview / Figma / VSCode / browser.

### Upload to an image host and share the link

If you need to show a remote collaborator (Slack / Lark / WeChat), have the user upload via their own image-host tool or MCP:

```bash
python ~/Documents/writing/tools/upload_image.py screenshot.png
```

Returns a permanent ImgBB link, pastable anywhere.

## When verification reveals errors

### White-screen page

There must be a console error. Check first:

1. The React+Babel script tag's integrity hash (see `react-setup.md`)
2. `const styles = {...}` naming collisions
3. Cross-file components not exported to `window`
4. JSX syntax errors (babel.min.js suppresses errors; switch to non-minified babel.js)

### Animation stutter

- Record with Chrome DevTools Performance tab
- Look for layout thrashing (frequent reflows)
- Prefer `transform` and `opacity` for animation (GPU-accelerated)

### Wrong font

- Check the `@font-face` url is reachable
- Check fallback font
- Slow CJK font loading: show fallback first, swap on load

### Layout broken

- Check `box-sizing: border-box` is applied globally
- Check `*  margin: 0; padding: 0` reset
- Open gridlines in Chrome DevTools to see actual layout

## Verification = the designer's second pair of eyes

**Always go through it yourself.** AI-written code often has:

- Looks right but interaction has bugs
- Static screenshot looks fine but breaks on scroll
- Wide-screen looks good but narrow-screen breaks
- Forgot to test Dark mode
- Some components don't respond after Tweaks switch

**The last minute of verification can save an hour of rework.**

## Common verification script commands

```bash
# Basic: open + screenshot + capture errors
python verify.py design.html

# Multi-viewport
python verify.py design.html --viewports 1920x1080,375x667

# Multi-slide
python verify.py deck.html --slides 10

# Output to specified directory
python verify.py design.html --output ./screenshots/

# headless=false, open a real browser for you to see
python verify.py design.html --show
```
