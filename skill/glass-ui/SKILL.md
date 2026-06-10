---
name: glass-ui
description: Build glass / glassmorphism / liquid-glass interfaces in HTML, CSS, and JS. Two techniques, CSS backdrop-filter for frosted panels and a WebGL fragment-shader "liquid glass" lens for the premium Apple-style refraction look. Use whenever the user asks for glass UI, glassmorphism, frosted panels, translucent cards, liquid glass, an Apple/visionOS-style card, or shows a screenshot of a glassy interface they want reproduced. Also use when adding a glass card or panel on top of a photo or video background.
---

# Glass UI

Two distinct techniques produce "glass". Pick deliberately; they are not interchangeable.

| Technique | Look | Use when |
|---|---|---|
| CSS glassmorphism | Frosted, static blur | Panels, modals, navbars, inputs. Fast, works everywhere, no JS. |
| WebGL liquid glass | Lens refraction, bright rim, light gradient | One hero card or element that must feel like physical glass. Needs a fullscreen canvas behind it. |

A page can combine both (e.g. a CSS-frosted control panel plus one WebGL hero card). Glass only reads as glass when there is something visually rich behind it: a photo, gradient mesh, or colorful artwork. On a flat solid background both techniques look like a gray box, so always put a strong backdrop behind glass elements.

The backdrop is roughly 80% of the perceived quality. The same shader looks cheap over a simple gradient and premium over a rich photo. Default to a high-detail, colorful, CORS-enabled photo (Unsplash `images.unsplash.com` URLs send `access-control-allow-origin: *`; the template ships with a nebula + landscape fallback chain). Use a real brand/user image when one exists, and keep a procedural gradient only as the offline fallback. Also match typography to the look: a geometric sans like Plus Jakarta Sans at weight 600 to 800, never default system font at normal weight.

## Output contract (mandatory)

Every glass UI you generate is ONE self-contained `.html` file that follows the skeleton below. `assets/liquid-glass-template.html` is the canonical reference; the structure, section order, and boilerplate are fixed. Do not reorder sections, drop sections, split into multiple files, or pull in external CSS/JS. Only the parts marked "VARIES" change per request.

### Required skeleton, in this exact order

```
<!doctype html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>VARIES</title>
    <style>
      * { box-sizing: border-box; }                 /* 1. reset */
      html, body { ... }                             /* 2. apple system font stack, dark bg */
      /* 3. backdrop layer: #glcanvas (WebGL) OR .bg gradient (CSS) */
      /* 4. glass surface: #card / .glass-panel, edge border + inset highlight */
      /* 5. states: .dragging, :focus-visible        */
      /* 6. typography: headings + body, weight 600-800 */
      /* 7. inner elements: .tile / input / button via the ink recipe */
      @media (max-width: 720px) { ... }              /* 8. mobile: card in normal flow */
    </style>
  </head>
  <body>
    <!-- backdrop element(s) first, then glass content on top -->
    <!-- WebGL only: fragment-shader <script>, then main <script> -->
  </body>
</html>
```

WebGL files additionally carry the main script split into four banner-commented sections, in this order and with this comment style:

```
/* =================== WebGL setup =================== */
/* =================== Ink colour  =================== */
/* =================== Draggable card =============== */
/* =================== Render loop =================== */
```

### Fixed (copy verbatim, never rewrite)

- Apple system font stack on `html, body`; dark page background.
- Glass surface = translucent background + `~0.3` alpha edge border + `inset 0 1px 0` top highlight. Inner elements use the `--ink-rgb` translucent recipe, never nested `backdrop-filter`.
- `--ink` / `--ink-rgb` variable system driving all foreground color.
- A rich backdrop with a procedural-gradient fallback; `prefers-reduced-transparency` solid fallback.
- Accessibility/export invariants: `crossorigin="anonymous"` on backdrop img, `aria-hidden` on decorative canvas, `preserveDrawingBuffer: true`, `:focus-visible` outline.
- WebGL only: the shader, texture upload, `loadBackground` fallback chain, drag handlers, render loop, `visibilitychange` pause, and resize clamp are boilerplate. Adapt the template; do not author them from scratch.

### Varies per request

- `<title>` and all card content (headings, copy, tiles/inputs/buttons/media).
- Card sizing: width `clamp(...)`, padding, `border-radius`.
- Backdrop image URLs (a real user/brand image first; keep a fallback chain).
- Inner-element kind (stat tiles vs form inputs vs media controls), always built with the ink-translucent recipe.
- `inkRGB` (light-on-dark vs dark-on-light, chosen for the backdrop).
- WebGL tuning constants only: superellipse exponent `6.0`, lens strength `0.22`, blur spread `1.2`, rim brightness `0.3`, `whiteness` at most `0.16`.

The boundary: structure, section order, and the invariants above stay identical run to run; content, sizing, backdrop, and tuning constants adapt to the brief. Output should be recognizably the same template, never byte-identical.

### Worked example A, CSS glassmorphism (input to on-template output)

Input: *"glassmorphism login page, email + password + sign-in button, animated gradient backdrop, one HTML file."*

On-template output: single file following the skeleton. Backdrop layer is a `.bg` animated gradient (no canvas). Section 4 is `.glass-panel` with backdrop-filter blur + `0.3` border + inset highlight. Section 7 inner elements are the email/password inputs and sign-in button, each styled with the `rgba(var(--ink-rgb),…)` recipe. No WebGL sections. `prefers-reduced-transparency` solid fallback present.

```html
<style>
  * { box-sizing: border-box; }
  html, body { margin:0; min-height:100%; font-family:-apple-system,"SF Pro Display",system-ui,sans-serif; }
  .bg { position:fixed; inset:0; background:linear-gradient(120deg,#1b2a6b,#7048a8,#d4527e); background-size:200% 200%; animation:drift 18s ease infinite; }
  @keyframes drift { 0%,100%{background-position:0 50%} 50%{background-position:100% 50%} }
  .glass-panel {                                   /* the glass surface */
    background:rgba(255,255,255,.1);
    border:1px solid rgba(255,255,255,.3);
    box-shadow:0 24px 60px rgba(0,0,0,.18), inset 0 1px 0 rgba(255,255,255,.35);
    backdrop-filter:blur(22px) saturate(1.2); -webkit-backdrop-filter:blur(22px) saturate(1.2);
    border-radius:24px; color:var(--ink,#fff);
  }
  .field { border:1px solid rgba(var(--ink-rgb,255,255,255),.45); background:rgba(var(--ink-rgb,255,255,255),.07); border-radius:12px; }
  @media (prefers-reduced-transparency: reduce) { .glass-panel { background:#222; backdrop-filter:none; } }
  @media (max-width:720px) { .glass-panel { width:min(88vw,340px); } }
</style>
<!-- body: <div class="bg"></div> then the glass-panel with inputs + button -->
```

### Worked example B, WebGL liquid glass (input to on-template output)

Input: *"visionOS-style liquid glass profile card, refracts the backdrop at its edges, draggable, name + role + 3 stat tiles."*

On-template output: copy `assets/liquid-glass-template.html` verbatim. VARIES only: set `<title>`, replace the `#card` h1/p with the name + role, fill the three `.tile`s with the requested stats, point `loadBackground([...])` at a fitting photo chain. Everything else (shader, the four banner-commented script sections, drag/render/resize/visibility plumbing, ink system) stays byte-for-byte. Tuning constants kept at defaults unless the brief asks for squarer corners (raise `6.0`) or stronger refraction (raise `0.22`).

## Technique 1: CSS glassmorphism

Four properties carry the entire effect:

```css
.glass-panel {
  background: rgba(255, 255, 255, 0.1);          /* 0.06 to 0.14 depending on backdrop brightness */
  border: 1px solid rgba(255, 255, 255, 0.3);    /* the glass edge */
  box-shadow:
    0 24px 60px rgba(0, 0, 0, 0.18),             /* depth */
    inset 0 1px 0 rgba(255, 255, 255, 0.35);     /* top inner highlight = light hitting the rim */
  backdrop-filter: blur(22px) saturate(1.2);
  -webkit-backdrop-filter: blur(22px) saturate(1.2);  /* Safari still needs the prefix */
  border-radius: 24px;
}
```

Why each piece matters:
- The `inset` top highlight is what sells it. Without it the panel is just a blurry rectangle.
- `saturate(1.2)` keeps the blurred backdrop vivid; plain blur goes muddy.
- Border at ~0.3 alpha reads as a catch-light on the glass edge.

### Inner elements (tiles, pills, buttons inside a glass panel)

Do NOT nest `backdrop-filter` (stacked blurs are expensive and often render wrong). Inner elements are flat translucent layers:

```css
.glass-inner {
  border: 1px solid rgba(255, 255, 255, 0.45);
  background: rgba(255, 255, 255, 0.07);
  border-radius: 15px;        /* or 999px for pills */
}
.glass-inner:hover { background: rgba(255, 255, 255, 0.16); }
```

### Ink color variable

Drive all foreground color through CSS variables so the whole card can flip between light-on-dark and dark-on-light:

```css
.card { color: var(--ink, #fff); }
.card .tile {
  border: 1px solid rgba(var(--ink-rgb, 255, 255, 255), 0.45);
  background: rgba(var(--ink-rgb, 255, 255, 255), 0.07);
}
```

Set `--ink` / `--ink-rgb` from JS if the backdrop changes (e.g. user uploads a light photo).

## Technique 2: WebGL liquid glass

Start from the bundled template: read `assets/liquid-glass-template.html`, copy it, and put the user's content inside the `#card` div. It is a complete working page (shader, draggable card, image/gradient backdrop). Do not rewrite the shader from scratch; adapt the template.

How it works, so you can modify it confidently:

1. The card div itself has NO background. A fullscreen `<canvas>` sits behind everything and renders the backdrop image every frame.
2. JS reads the card's `getBoundingClientRect()` each frame and passes center + half-size to the shader as uniforms. The glass region follows the div automatically, so dragging works for free.
3. Inside the shader, the card shape is a superellipse: `pow(abs(d.x), 6.0) + pow(abs(d.y), 6.0)` where `d` is the pixel offset normalized by the card half-size. Value < 1 means inside the rounded rect.
4. Refraction: sample the backdrop at `lens = center + (uv - center) * (1.0 - roundedBox * 0.22)`. Pixels near the edge sample closer to the card center, which magnifies and warps the backdrop exactly like thick glass. The `0.22` is lens strength.
5. Frosting: a 9x9 box blur of the lens sample (the `-4..4` double loop).
6. Lighting: `rb1` (soft interior mask) times a vertical gradient adds the top-light; `rb2` (thin band at the boundary) adds the bright rim.
7. "Liquidness/whiteness": `mix(lighting, vec4(1.0), uWhite)` fades the glass toward white. Keep `uWhite` at or below 0.16; above that it looks like paper.

Tuning knobs (all in the template, commented):
- Corner roundness: the superellipse exponent (6.0). Higher = squarer corners.
- Lens strength: the 0.22 factor.
- Blur amount: the loop offset multiplier (1.2) or loop range.
- Rim brightness: the `* 0.3` on `rb2`.

HTML content on top of the card (text, stats, avatars) is regular DOM, styled with the inner-element recipe from Technique 1. Only the glass slab itself is shader-drawn.

## Pitfalls

- `backdrop-filter` ignores elements without a stacking context below them; if blur does nothing, check the backdrop actually renders behind the panel (not as a sibling painted later).
- Text on glass needs weight, not opacity: use font-weight 600 to 800 with high-alpha color rather than thin text at low alpha.
- WebGL canvas must use `preserveDrawingBuffer: true` if you ever export it to PNG.
- Cross-origin backdrop images taint canvas export; load with `crossorigin="anonymous"` and a CORS-enabled host, or fall back to a procedural/gradient backdrop (the template includes one).
- On mobile, drop draggability and let the card sit in normal flow; keep the canvas `position: fixed` behind it.
- Respect `prefers-reduced-transparency`: provide a solid fallback background when set.
