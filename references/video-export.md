# Video Export: Exporting HTML Animations as MP4/GIF

After the animation HTML is done, users often ask "can you export it as a video?" This guide gives the complete workflow.

## When to Export

**Timing for export**:
- Animation runs end-to-end and has been visually verified (Playwright screenshots confirming state at each time point is correct)
- The user has watched it in the browser at least once and indicated it looks OK
- **Do not** export while there are unfixed animation bugs—fixing things after export is more expensive

**Trigger phrases the user might say**:
- "Can you export it as a video"
- "Convert to MP4"
- "Make it a GIF"
- "60fps"

## Output Specifications

By default, deliver three formats at once and let the user choose:

| Format | Spec | Suitable scenario | Typical size (30s) |
|---|---|---|---|
| MP4 25fps | 1920×1080 · H.264 · CRF 18 | WeChat public account embed, video account, YouTube | 1-2 MB |
| MP4 60fps | 1920×1080 · minterpolate frame interpolation · H.264 · CRF 18 | High frame rate display, Bilibili, portfolio | 1.5-3 MB |
| GIF | 960×540 · 15fps · palette optimized | Twitter/X, README, Slack preview | 2-4 MB |

## Tool Chain

Two scripts in `scripts/`:

### 1. `render-video.js` — HTML → MP4

Records a 25fps base MP4. Depends on global playwright.

```bash
NODE_PATH=$(npm root -g) node /path/to/claude-design/scripts/render-video.js <html file>
```

Optional parameters:
- `--duration=30` animation duration (seconds)
- `--width=1920 --height=1080` resolution
- `--trim=2.2` seconds to cut from the start of the video (removes reload + font loading time)
- `--fontwait=1.5` font loading wait time (seconds); raise it when there are many fonts

Output: same directory as the HTML, same name `.mp4`.

### 2. `add-music.sh` — MP4 + BGM → MP4

Mixes background music into a silent MP4, picking from the built-in BGM library by scene (mood); can also use your own audio. Auto-matches duration, adds fade in/out.

```bash
bash add-music.sh <input.mp4> [--mood=<name>] [--music=<path>] [--out=<path>]
```

**Built-in BGM library** (in `assets/bgm-<mood>.mp3`):

| `--mood=` | Style | Suitable scenario |
|-----------|------|---------|
| `tech` (default) | Apple Silicon / Apple keynote, minimal synth + piano | Product launches, AI tools, Skill promotion |
| `ad` | upbeat modern electronic, with build + drop | Social media ads, product teasers, promo clips |
| `educational` | Warm and bright, light guitar/electric piano, inviting | Science popularization, tutorial intros, course teasers |
| `educational-alt` | Same category alternative, try a different track | Same as above |
| `tutorial` | lo-fi ambient, almost no presence | Software demos, programming tutorials, long demos |
| `tutorial-alt` | Same category alternative | Same as above |

**Behavior**:
- Music is trimmed to video duration
- 0.3s fade in + 1s fade out (avoid hard cuts)
- Video stream `-c:v copy` no re-encoding, audio AAC 192k
- `--music=<path>` takes priority over `--mood`; can directly specify any external audio
- Passing a wrong mood name will list all available options and not fail silently

**Typical pipeline** (animation export trio + music):
```bash
node render-video.js animation.html                        # screen record
bash convert-formats.sh animation.mp4                      # derive 60fps + GIF
bash add-music.sh animation-60fps.mp4                      # add default tech BGM
# Or for different scenarios:
bash add-music.sh tutorial-demo.mp4 --mood=tutorial
bash add-music.sh product-promo.mp4 --mood=ad --out=promo-final.mp4
```

### 3. `convert-formats.sh` — MP4 → 60fps MP4 + GIF

Generates a 60fps version and GIF from an existing MP4.

```bash
bash /path/to/claude-design/scripts/convert-formats.sh <input.mp4> [gif_width] [--minterpolate]
```

Output (same directory as input):
- `<name>-60fps.mp4` — defaults to `fps=60` frame duplication (broad compatibility); add `--minterpolate` to enable high-quality interpolation
- `<name>.gif` — palette-optimized GIF (default 960 wide, configurable)

**60fps mode selection**:

| Mode | Command | Compatibility | Use case |
|---|---|---|---|
| Frame duplication (default) | `convert-formats.sh in.mp4` | QuickTime/Safari/Chrome/VLC all play | General delivery, upload platforms, social media |
| minterpolate frame interpolation | `convert-formats.sh in.mp4 --minterpolate` | macOS QuickTime/Safari may refuse to open | Bilibili and other display scenarios that need true interpolation; **must test target player locally before delivery** |

Why was the default changed to frame duplication? minterpolate's H.264 elementary stream output has a known compat bug—we previously hit "macOS QuickTime can't open" several times when minterpolate was the default. See `animation-pitfalls.md` §14.

`gif_width` parameter:
- 960 (default) — universal for social platforms
- 1280 — clearer but larger file
- 600 — Twitter/X load priority

## Complete Workflow (Standard Recommended)

After the user says "export as video":

```bash
cd <project directory>

# Assume $SKILL points to the root of this skill (replace with your install location)

# 1. Record 25fps base MP4
NODE_PATH=$(npm root -g) node "$SKILL/scripts/render-video.js" my-animation.html

# 2. Derive 60fps MP4 and GIF
bash "$SKILL/scripts/convert-formats.sh" my-animation.mp4

# Output checklist:
# my-animation.mp4         (25fps · 1-2 MB)
# my-animation-60fps.mp4   (60fps · 1.5-3 MB)
# my-animation.gif         (15fps · 2-4 MB)
```

## Technical Details (For Troubleshooting)

### Pitfalls of Playwright recordVideo

- Frame rate is fixed at 25fps; can't directly record 60fps (Chromium headless compositor upper limit)
- Recording starts from when the context is created, must use `trim` to cut the front loading time
- Default webm format; need ffmpeg to convert to H.264 MP4 for universal playback

`render-video.js` already handles the above issues.

### ffmpeg minterpolate Parameters

Current configuration: `minterpolate=fps=60:mi_mode=mci:mc_mode=aobmc:me_mode=bidir:vsbmc=1`

- `mi_mode=mci` — motion compensation interpolation
- `mc_mode=aobmc` — adaptive overlapped block motion compensation
- `me_mode=bidir` — bidirectional motion estimation
- `vsbmc=1` — variable size block motion compensation

Works well for CSS **transform animations** (translate/scale/rotate).
For **pure fades** may produce slight ghosting—if the user dislikes it, fall back to simple frame duplication:

```bash
ffmpeg -i input.mp4 -r 60 -c:v libx264 ... output.mp4
```

### Why GIF Palette Needs Two Stages

GIFs only support 256 colors. A one-pass GIF squeezes the entire animation's colors into a 256-color general palette, which gets blurry for delicate combinations like beige background + orange.

Two stages:
1. `palettegen=stats_mode=diff` — first scan the whole clip and generate **an optimal palette specific to this animation**
2. `paletteuse=dither=bayer:bayer_scale=5:diff_mode=rectangle` — encode using this palette; rectangle diff only updates changed regions, greatly reducing file size

For fade transitions, `dither=bayer` is smoother than `none`, but the file is a bit bigger.

## Pre-flight Check (Before Export)

A 30-second self-check before export:

- [ ] HTML has been run through completely in the browser, no console errors
- [ ] Frame 0 of the animation is the complete initial state (not a blank loading state)
- [ ] The last frame of the animation is a stable closing state (not cut off mid-way)
- [ ] Fonts/images/emoji all render correctly (refer to `animation-pitfalls.md`)
- [ ] The Duration parameter matches the actual animation duration in the HTML
- [ ] In the HTML the Stage detects `window.__recording` and forces loop=false (mandatory check for hand-written Stage; built into `assets/animations.jsx`)
- [ ] The closing Sprite has `fadeOut={0}` (don't fade out on the video's last frame)
- [ ] Includes "Created by Huashu-Design" watermark (mandatory only for animation scenarios; for third-party brand work add prefix "Unofficial · ". See SKILL.md §"Skill promotion watermark")

## Notes Attached at Delivery

Standard explanation format to give the user after export is complete:

```
**Complete delivery**

| File | Format | Spec | Size |
|---|---|---|---|
| foo.mp4 | MP4 | 1920×1080 · 25fps · H.264 | X MB |
| foo-60fps.mp4 | MP4 | 1920×1080 · 60fps (motion interpolation) · H.264 | X MB |
| foo.gif | GIF | 960×540 · 15fps · palette optimized | X MB |

**Notes**
- 60fps uses minterpolate for motion estimation interpolation; works well for transform animations
- GIF uses palette optimization; a 30s animation can compress to around 3MB

Let me know if you need a different size or frame rate.
```

## Common Follow-up User Requests

| User says | Response |
|---|---|
| "Too big" | MP4: raise CRF to 23-28; GIF: lower resolution to 600 or fps to 10 |
| "GIF is too blurry" | Raise `gif_width` to 1280; or suggest using MP4 instead (WeChat Moments also supports it) |
| "Want vertical 9:16" | Change HTML source to `--width=1080 --height=1920`, re-record |
| "Add a watermark" | ffmpeg add `-vf "drawtext=..."` or `overlay=` a PNG |
| "Want transparent background" | MP4 doesn't support alpha; use WebM VP9 + alpha or APNG |
| "Want lossless" | Change CRF to 0 + preset veryslow (file will be 10x larger) |
