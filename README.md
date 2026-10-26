# Gallery components

Four interactive image components, each a single self-contained HTML file (markup, CSS and JS in one page).

| File | Component | Interaction |
| --- | --- | --- |
| [`drift-gallery.html`](drift-gallery.html) | Drift Gallery | Cards cascade along a diagonal depth field. Autoplay drift, drag or wheel to scrub, cards near the cursor are pushed aside. |
| [`spiral-gallery.html`](spiral-gallery.html) | Spiral Gallery | Images on a 3D helix that rotates and climbs. Drag or wheel to spin, with inertia. |
| [`scale-slider.html`](scale-slider.html) | Scale Slider | Looping slider: the focused card is full size, neighbours shrink, dim and tuck in behind. Drag, wheel, ← → buttons, arrow keys. |
| [`infinite-draggable-grid.html`](infinite-draggable-grid.html) | Infinite Draggable Grid | Endless photo wall in every direction. Drag, throw, wheel; click a tile to expand it, click again or Esc to close. |

## Run

Open any of the four files in a browser. No build step, no server needed (file:// works; any static server works too). An internet connection is needed for GSAP (loaded from a CDN) and the placeholder photos.

```
drift-gallery.html
spiral-gallery.html
scale-slider.html
infinite-draggable-grid.html
README.md
```

## Dependencies

- **GSAP 3.13.0, core only.** No GSAP plugins (no ScrollTrigger, Draggable, etc.). APIs used: `gsap.ticker`, `gsap.set`, `gsap.to`, `gsap.from`, `gsap.utils.wrap`.
  - As shipped: loaded from jsDelivr, `https://cdn.jsdelivr.net/npm/gsap@3.13.0/dist/gsap.min.js`, pinned to that version with an `integrity` hash.
  - Offline: download that file next to the pages and point the `<script>` tag at it.
  - npm: `npm i gsap@3.13.0`, then `import { gsap } from "gsap";`
  - License: GSAP standard license, free for commercial use: https://gsap.com/standard-license
- **No other libraries, no fonts.** Type uses the system font stacks in `--sans` and `--mono`.
- **Photos:** placeholders from picsum.photos via `KIT.img(seed, w, h)` → `https://picsum.photos/seed/gallery<seed>/<w>/<h>`. Same seed, same picture, so the pages look the same on every load. Replace with real assets before shipping.

## File anatomy

Every file has the same four parts, in this order:

1. The GSAP `<script>` tag.
2. **Shared helpers** (`const KIT = {...}`), identical in all four files: `img`, `words` (the card labels in Scale Slider), `rand`, `clamp`, `lerp`, `$` (querySelector), `$$` (querySelectorAll as an array).
3. **Shared base CSS**, identical in all four files: colour tokens on `:root`, a small reset, and the text styles `.eyebrow`, `.h2`, `.hint`, plus the `.round` button (used by Scale Slider).
4. **Component** CSS, markup and script.

If you put several of these on one site, lift parts 2 and 3 into one shared stylesheet and one shared script.

Scale Slider's buttons are found by their `data-prev` and `data-next` attributes; everything else is selected by class.

## How each one works

All four run a per-frame loop on `gsap.ticker` and write transforms directly (`gsap.set` or `style.transform`). There are no timelines; the state lives in a few variables.

### Drift Gallery

- 22 cards (240 × 320) inside a `perspective: 900px` scene. Card `i` sits at distance `d = wrap(i * gap - offset)` along the cascade, so cards loop endlessly. Position: `x = d * slopeX`, `y = d * slopeY + jitter`, `z = -1.1 * d`, `rotateY(-22deg)` plus a small random `rotateZ`.
- Depth of field: past 12% of the half span, blur grows with distance (`dof`), opacity falls as `1 - depth³`, and a red `drop-shadow` offset grows with depth (`aberration`) for a chromatic fringe. `zIndex` follows distance.
- Cursor push: each card's screen position is projected; cards within `pushRadius` (220px) of the pointer are pushed away by up to `push` (120px), eased.
- Motion: autoplay drift of `autoplay` px per second (time based). Horizontal drag scrubs; on release the velocity decays by 0.94 per frame. The wheel adds velocity.
- Film grain: a fixed overlay with an SVG `feTurbulence` noise tile, jittered by a `steps(3)` keyframe.
- Intro: `gsap.from('.card', { opacity: 0, scale: .6, stagger: .03, duration: 1.2, ease: 'expo.out' })`.
- Knobs in `CFG`: `n, gap, slopeX, slopeY, rotY, jitterY, jitterRot, autoplay, push, pushRadius, dof, aberration`.

### Spiral Gallery

- 36 images (170 × 120) placed once on a helix: `rotateY(angle) translateZ(radius) translateY(height step)`, 3 turns, radius 360, height 1500. The `.helix` wrapper has `transform-style: preserve-3d` inside a `perspective: 1100px` scene.
- Each frame the helix rotates by `vel` degrees and moves vertically in step with the rotation (wrapped), so it reads as an endless spiral. It is tilted with `rotateX(-8deg)`.
- Each image's opacity goes from 0.25 to 1 depending on how much it faces the viewer (cosine of its angle).
- Drag sets the velocity (clamped to ±`inertiaMax`); on release it eases back to the `autoplay` speed. The wheel adds velocity.
- Intro: the scene scales in from 0.8 with a fade.
- Knobs in `CFG`: `count, turns, radius, height, inertiaDecay, inertiaMax, autoplay`.

### Scale Slider

- 9 cards, `min(420px, 64vw)` wide at 3:4, labelled from `KIT.words`. A continuous `pos` eases toward an integer `target` (lerp 0.09 per frame).
- For each card, `d` is its slot distance from the focus (wrapped, so the slider loops). Scale is `max(.55, 1 - .22|d|)`. Horizontal spacing follows a geometric series (0.62^|d|), which is what tucks the neighbours in behind the focused card. Cards drop 18px per slot, darken by 18% per slot (`brightness`), and hide beyond 3.2 slots. The photo inside each card shifts sideways for parallax (it is oversized: 124% × 112%).
- Drag moves `target` by the pointer distance over 0.6 card widths and snaps to the nearest card on release. Wheel (both axes) moves it and snaps after 150ms idle. Buttons and the ← → keys step one card.
- Starts with `pos = -3`, so the first frames slide the cards into place.
- Knobs in `CFG`: `gap, minScale, parallax`. (`focusY` and `autoplay` are declared but not used.)

### Infinite Draggable Grid

- A fixed pool of tiles (220 × 280, 24px gap): one more column and row than fits the viewport on each side. Every frame each tile's position is wrapped around the camera, and odd rows are offset half a column (brick layout). The DOM never grows, however far you drag.
- Each tile works out which world cell it is showing and hashes it to a seed, so a cell always shows the same photo. The `<img>` is only replaced when the seed changes. There are 300 distinct seeds, so photos repeat.
- Drag moves the camera; on release it keeps its velocity (throw) with 0.93 friction per frame. The wheel adds velocity in both axes. The page starts with a small drift.
- Click (pointer moved under 6px): a fixed `.detail` element starts at the tile's rectangle and animates to a centred frame 80% of the viewport height (`expo.inOut`, 0.9s), with a larger image (880 × 1120) and a dark scrim. Click the image or scrim, or press Esc, to animate it back to the tile.
- Hover: the photo scales to 1.06 with a CSS transition.
- Constants: `TW, TH, GAP` at the top of the script.

## Integration notes

- **They are full-viewport demos.** Each scene is `position: fixed; inset: 0` with `body { overflow: hidden }`, and the wheel and key listeners are on `window`. To place one inside a scrolling page, make the scene `position: relative` (or absolute) with a set height, attach the wheel and key listeners to the scene instead of `window`, and drop `body { overflow: hidden }`. Otherwise the wheel drives the component and the page cannot scroll.
- **The loops never stop.** Each `gsap.ticker` callback runs every frame, even off screen. On a real page, pause it when the component is out of view (IntersectionObserver plus `gsap.ticker.remove` / `add`).
- **Cleanup in a framework** (React, Vue, Svelte): the scripts keep no handles for teardown. On unmount, remove the ticker callback, the `window` listeners (`wheel`, `keydown`, `resize`) and kill running tweens; wrapping setup in `gsap.context()` and calling `ctx.revert()` handles the tweens.
- **Frame-rate dependence.** Spiral rotation, Scale Slider easing, Grid friction and Drift's drag inertia are per frame, so they run faster on 120 Hz screens. Drift's autoplay uses delta time and is unaffected. To normalise, scale those steps by `gsap.ticker.deltaRatio()`.
- **Performance.** Drift Gallery animates a CSS `filter` (blur + drop-shadow) on 22 cards every frame, which is the heaviest of the four on low-end GPUs. Infinite Grid rebuilds its tile pool on every `resize`, which reloads the photos.

## Known gaps

- No `prefers-reduced-motion` handling. Suggested: stop autoplay and drift, and jump instead of easing.
- Keyboard: only Scale Slider (arrow keys) and Infinite Grid (Esc to close) respond. Drift and Spiral have none, and the grid tiles are not focusable.
- All images have empty `alt` text; add real descriptions with real content.
- `pointercancel` is not handled, so an interrupted touch can leave a drag active until the next pointer up.
