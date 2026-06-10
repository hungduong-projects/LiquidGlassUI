# ◈ Liquid Glass UI

> Glass that actually refracts. Apple-style liquid glass for the web in one dependency-free HTML file.

[![Live Demo](https://img.shields.io/badge/demo-live-5eead4?style=flat-square)](https://hungduong-projects.github.io/LiquidGlassUI/)
[![License: MIT](https://img.shields.io/badge/license-MIT-blue?style=flat-square)](LICENSE)
[![Zero Dependencies](https://img.shields.io/badge/dependencies-0-success?style=flat-square)](#)

**[▶ Live demo](https://hungduong-projects.github.io/LiquidGlassUI/)** : drag the glass card, the shader tracks it in real time.

## What is this?

Two distinct glass techniques, both in plain HTML/CSS/JS. No build step, no framework, no npm.

| Technique | Look | Use when |
|---|---|---|
| **CSS glassmorphism** | Frosted, static blur | Panels, modals, navbars, inputs. Fast, works everywhere, no JS. |
| **WebGL liquid glass** | Lens refraction, bright rim, light gradient | One hero card that must feel like physical glass. |

## Quick start

```bash
git clone https://github.com/hungduong-projects/LiquidGlassUI.git
cd LiquidGlassUI
open index.html   # or just double-click it
```

Or grab [`skill/glass-ui/assets/liquid-glass-template.html`](skill/glass-ui/assets/liquid-glass-template.html) as a clean starting point and put your own content inside the `#card` div.

## Technique 1: CSS glassmorphism

Four properties carry the entire effect:

```css
.glass {
  background: rgba(255, 255, 255, 0.1);          /* 0.06 to 0.14 depending on backdrop */
  border: 1px solid rgba(255, 255, 255, 0.3);    /* the glass edge */
  box-shadow:
    0 24px 60px rgba(0, 0, 0, 0.18),             /* depth */
    inset 0 1px 0 rgba(255, 255, 255, 0.35);     /* top highlight = light on the rim */
  backdrop-filter: blur(22px) saturate(1.2);
  -webkit-backdrop-filter: blur(22px) saturate(1.2);  /* Safari */
}
```

- The `inset` top highlight is what sells it. Without it the panel is just a blurry rectangle.
- `saturate(1.2)` keeps the blurred backdrop vivid; plain blur goes muddy.
- Never nest `backdrop-filter`. Inner elements (tiles, pills) are flat translucent layers.

## Technique 2: WebGL liquid glass

The card div has **no background**. A fullscreen canvas renders the backdrop, and a fragment shader draws the glass slab behind the div:

1. JS reads the card's `getBoundingClientRect()` every frame and passes centre + half-size as uniforms, so dragging works for free.
2. The card shape is a superellipse: `pow(abs(d.x), 6.0) + pow(abs(d.y), 6.0)`.
3. **Refraction**: edge pixels sample the backdrop closer to the card centre, magnifying and warping it like thick glass.
4. **Frosting**: a 9x9 box blur of the lens sample.
5. **Lighting**: a soft interior mask times a vertical gradient, plus a thin bright rim at the boundary.

### Tuning knobs

All in the shader, commented inline:

| Knob | Default | Effect |
|---|---|---|
| Superellipse exponent | `6.0` | Higher = squarer corners |
| Lens strength | `0.22` | Higher = stronger refraction |
| Blur spread | `1.2` | Higher = frostier |
| Rim brightness | `0.3` | Higher = brighter edge |
| Whiteness `uWhite` | `0.0` | Fades toward white; keep ≤ 0.16 |

The demo's bottom-right **Tune the glass** panel drives all of these live, plus the CSS-frame blur, tint, edge and saturation, with a value readout beside each slider.

## Files

```
index.html                                        Demo page (what GitHub Pages serves)
bg.jpg                                            Backdrop image for the demo
skill/glass-ui/                                   Claude Code skill: teaches an AI agent both techniques
skill/glass-ui/assets/liquid-glass-template.html  Minimal template to copy into your project
```

## Tips

- Glass only reads as glass over a **visually rich backdrop** (photo, gradient mesh, artwork). On flat colors it looks like a gray box.
- Text on glass needs **weight, not opacity**: font-weight 600 to 800 with high-alpha color.
- Respect `prefers-reduced-transparency` and `prefers-reduced-motion` (the demo does).
- Cross-origin backdrop images taint canvas export; use `crossorigin="anonymous"` with a CORS-enabled host or a procedural backdrop.

## Contributing

Issues and PRs welcome. Keep it dependency-free: if a change needs npm, it probably belongs in a fork.

## License

[MIT](LICENSE)
