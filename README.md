# EARTH

Sacred animated logo. A glowing orb at the center, with branching tendrils radiating into a luminous spherical canopy. Symmetric, breathing, alive.

**Live:** https://live-xi-flame.vercel.app
**Studio mode:** https://live-xi-flame.vercel.app/?studio (or press `S` on the live URL to toggle)

---

## What this is

A single self-contained HTML file that renders a generative tree-of-life logo using Three.js + custom GLSL shaders + post-processing bloom. No build step, no bundler — just open `index.html` in any modern browser.

- **6-fold rotational symmetry** + Y-mirror = 12-way sacred geometry
- **45-second seamless rotation loop** (every motion is integer cycles per loop, so it's pixel-identical at frame 0 and frame N)
- **Live geometry generation** — branches built procedurally with recursive splitting, sphere-clamping, drape effects, and brush-tip leaves
- **Vertex shader animation** — radial breath, Y-bob, body sway, leaf rustle (all symmetry-preserving)
- **Fragment shader lighting** — front/back proximity, crown fill, closure glow, primary stem boost, root thickness, layered chunked fade
- **Post-processing** — ACES filmic tonemapping + Unreal bloom

---

## How to use

### Just view it
Open `index.html` in any modern browser (Chrome recommended). Internet required for the first load (pulls Three.js from `esm.sh` CDN, then cached).

### Studio mode
Add `?studio` to the URL or press `S` on the page. A panel appears in the top-right with ~30 sliders for live tweaking:

- **Visual** — bloom, radius, threshold, glow, tint R/G/B
- **Camera** — zoom, fov, wobble
- **Motion** — loop sec, motion (master), breath, bob, sway, rustle
- **Geometry** — round (Y stretch), clamp (sphere adherence), roundness (alpha fade)
- **Branches** — primary thick, branch thick, roots
- **Orb** — orb size, orb intensity, orb color, core color, hot color
- **Other** — front (depth lighting), primary color

### Studio actions
- **⚡ RANDOM** — randomize all sliders within sensible aesthetic ranges
- **undo** — step back through your changes
- **reset** — restore all defaults
- **save** — store current settings as a named preset (in browser localStorage)
- **del** — delete the currently selected preset
- **export** — copy all your saved presets as JSON for sharing
- **import** — paste JSON to merge presets from someone else
- **load preset dropdown** — switch to a saved preset
- **REC** — record a video of the canvas (toggle to start/stop, downloads webm)

### URL parameters
Every slider value can be set via URL:
```
https://live-xi-flame.vercel.app/?studio&bloom=1.2&loop=30&tintG=1.4&orbColor=ff8800
```
The studio panel auto-updates the URL as you drag sliders, so the address bar always reflects your current configuration. Bookmark or share to lock in a look.

### Keyboard
- `S` — toggle studio panel
- `+` / `-` — scale studio panel up/down (for browser zoom-out usability)
- `0` — reset studio panel size

### Mouse interactions
- **Click + drag near the orb (~45 unit radius)** → unravel a glowing yarn trail from the core; the tree fades in chunked layers (leaves first, trunk last) over 5 seconds. Release → everything reassembles.
- **Click + drag anywhere outside the orb** → spin/toss the tree with momentum. Fast flicks = wild spins. Releases decay smoothly back to natural slow rotation.

---

## Architecture

### Single file, ~1100 lines
`index.html` contains everything: HTML structure, CSS for studio panel, all JavaScript (geometry generation, shaders, interaction, studio UI), and inline GLSL shader source.

### Dependencies (all CDN-loaded at runtime)
- `three@0.160.0` — core Three.js
- `EffectComposer`, `RenderPass`, `UnrealBloomPass` — Three.js post-processing addons

### Key sections of the code

| Lines (approx.) | What |
|---|---|
| 1–100 | HTML + studio panel CSS + slider DOM |
| 100–150 | Defaults, URL param parsing, color helpers |
| 150–230 | Geometry helpers: `softSpherePull`, `repelFromWaist`, `drapeOuterRim`, `getPerp` |
| 230–330 | `buildBranch` — recursive branch generation with motion phases, drape, micro-tip leaves |
| 330–410 | Setup geometry: trunk, wedge primaries, crown fill, rotational replication, Y-mirror |
| 410–500 | BufferGeometry build — pushes each branch as line segments + parallel strands for thickness |
| 500–710 | ShaderMaterial: vertex shader (motion), fragment shader (lighting + chunked fade) |
| 710–820 | Orb meshes (6 outer halo shells + core + hot center), composer setup |
| 820–880 | Yarn trail interaction system + dual-mode pointer handlers |
| 880–950 | `animate()` loop with rotation, motion, momentum, chunked fade |
| 950–end | Studio panel slider wiring, preset load/save/export, REC button |

### Symmetry preservation
All vertex shader motion uses only `r` (length(xz)), `r3` (length(xyz)), and `|y|` — values invariant under Y-axis rotation and Y-axis mirroring. So the 6-fold rotational symmetry and Y-mirror are exactly preserved at every frame regardless of motion amplitude.

### Loop seamlessness
Motion frequencies are integer cycles per loop (e.g., breath = 4 cycles, bob = 6 cycles, sway = 3/5 cycles, rustle = 11/6/10 cycles). At frame 0 and frame `loop * 60`, every uniform's sin/cos value is identical, so the rendered frame is bit-identical. Means the recorded MP4 loops perfectly.

---

## Deploy

This is hosted on Vercel as a static site:
```bash
cd live/
vercel deploy --prod --yes
```

Project name: `live` under team `jeffs-projects-6086830a`.

The production alias (https://live-xi-flame.vercel.app) updates immediately on each deploy.

---

## Embedding on another site

### As a video (lightweight, no interactivity)
Use the REC button to capture a loop, save the webm, convert to mp4 with ffmpeg:
```bash
ffmpeg -i earth.webm -c:v libx264 -pix_fmt yuv420p -crf 18 -movflags +faststart earth.mp4
```
Then embed:
```html
<video autoplay loop muted playsinline src="earth.mp4"></video>
```

### As a live render (full interactivity)
Add an iframe to your page:
```html
<iframe src="https://live-xi-flame.vercel.app" style="width:100%;aspect-ratio:1;border:0;"></iframe>
```
Or import the JS directly into your project for tighter integration.

---

## Customization examples

- **Royal purple tree:** `?tintR=0.7&tintG=0.4&tintB=1.4&primaryColor=8a4cff`
- **Fast spin:** `?loop=15`
- **Calm ambient:** `?loop=90&motion=0.5&bloom=0.7`
- **Solar orb (orange center):** `?orbColor=ff7700&hotColor=ffeebb`
- **Static skeleton:** `?motion=0&bloom=0.2`
- **Hyper-bloom:** `?bloom=1.8&glow=1.4`

---

## Contributing

Open an issue or PR. Every change should preserve:
1. The seamless 45-second loop (integer cycle counts in shader motion)
2. The 6-fold rotational + Y-mirror symmetry (motion math depends only on `r`, `r3`, `|y|`)
3. Performance at 60fps on mid-range hardware

---

Built by Ricardo + Peace + Claude. 🌍
