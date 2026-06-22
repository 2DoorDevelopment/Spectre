# Spectre — Design System

**Last updated:** 2026-06-22

The aesthetic is a **HUD/cockpit interface**: near-black panels, thin glowing borders, monospaced-feeling display type, restrained accent colors. Derived directly from the supplied mockup.

---

## 1. Color Palette

Pulled from the mockup's style guide, with semantic roles assigned.

| Token | Hex | Role |
|-------|-----|------|
| `--bg-base` | `#070B12` | Page background (deepest near-black). |
| `--bg-panel` | `#0D1420` | Panel / card background. |
| `--bg-elevated` | `#1F3147` | Raised surfaces, hover states, borders-as-fill. |
| `--accent-cyan` | `#00C6FF` | Primary accent: active nav, primary buttons, links, focus. |
| `--accent-green` | `#30DC84` | Success / online status / positive values. |
| `--accent-amber` | `#FFAE00` | Warning / highlight / attention (use sparingly). |
| `--text-primary` | `#E6EDF5` | Primary text (off-white, not pure white). |
| `--text-muted` | `#7E8B9E` | Secondary text, labels, captions. |
| `--border-line` | `#1F3147` | Default thin panel borders. |
| `--border-glow` | `#00C6FF` @ low alpha | Accent border glow on focus/active. |

### Usage rules
- **Cyan is the signature.** It carries identity; use it for the most important interactive element on a screen, not everything.
- **Amber is rare.** Reserve for genuine warnings/attention. Overuse kills the HUD feel.
- **Green = status/positive only.** Online indicators, profit, success toasts.
- **Never pure white or pure black** for text/bg; use the off-tones above to reduce eye strain on dark UI.
- Danger/error red is intentionally not in the mockup palette; add `--accent-red: #FF4D5E` only for destructive actions and errors.

---

## 2. Typography

| Role | Font | Weights | Notes |
|------|------|---------|-------|
| Display / headings / stats | **Orbitron** | 500, 700 | Uppercase, wide letter-spacing for the HUD look. |
| UI / body | **Rajdhani** | 400, 500, 600 | Condensed, technical; sharper than Raleway for cockpit feel. |
| Data tables / dense numbers | **Inter** (fallback) | 400, 500 | Where pure readability beats flavor. |

Loaded via `next/font` with only the listed weights, subsetted, to keep payload small.

### Scale (Tailwind-aligned)
| Use | Size | Font |
|-----|------|------|
| Hero title | 48–64px | Orbitron 700 |
| Page title | 28–32px | Orbitron 700 |
| Stat number | 24–32px | Orbitron 500 |
| Section label | 12px, uppercase, letter-spaced | Orbitron 500 / Rajdhani 600 |
| Body | 14–16px | Rajdhani 400/500 |
| Caption / muted | 12px | Rajdhani 400 |

---

## 3. Layout & Spacing

- **Grid:** 8px base spacing unit. Tailwind spacing scale (multiples of 4/8).
- **Panels:** rounded corners (`6–8px`), 1px border in `--border-line`, subtle inner padding (16–24px).
- **App shell:** left vertical icon sidebar (collapsible) + top nav bar + content area, matching the mockup dashboard.
- **Max content width:** ~1440px centered on large screens; full-bleed dark background.
- **Corner accents:** optional thin L-shaped bracket details on key panels for the HUD frame look (the mockup's outer frame corners).

---

## 4. Core Components (design-system primitives)

These live in `src/components/ui/` and are built during Phase 0.

| Component | Description |
|-----------|-------------|
| `Panel` | The base card: dark bg, thin border, optional corner brackets, optional title bar. |
| `Stat` | Big Orbitron number + small uppercase label + optional sub-link (the "Quick Stats" cells). |
| `Button` | Primary (cyan fill/outline), Secondary (muted outline). Sharp corners, uppercase Orbitron label. |
| `StatusDot` | Small colored dot + label (green online / amber degraded / red offline). |
| `NavItem` | Sidebar/topnav item with active cyan underline/glow. |
| `Badge` | Small pill for tags (e.g., ship roles, org ranks). |
| `DataTable` | Dense, sortable table styled for the dark HUD (zebra via subtle elevation, not heavy lines). |
| `EmptyState` | Centered icon + in-universe flavor text ("No ships in your hangar yet."). |
| `Skeleton` | Loading placeholder with a faint cyan shimmer. |

### Interaction states
- **Hover:** slight elevation (`--bg-elevated`) + border brightens toward cyan.
- **Active/selected:** cyan underline or left-border, faint glow.
- **Focus:** visible cyan focus ring (accessibility — never remove outlines without replacement).
- **Disabled:** reduced opacity, no glow.

---

## 5. The "Unique" Factor

To avoid looking templated, Spectre leans on:
- **Corner bracket framing** on hero and key panels (HUD reticle feel).
- **Thin animated scan-line or sweep** on the hero only (subtle, performance-cheap CSS, respects `prefers-reduced-motion`).
- **Orbitron wide-tracked labels** as a consistent signature.
- **Restraint**: lots of negative space, few colors, which reads as intentional and premium rather than busy.

Animations must respect `prefers-reduced-motion` and never block interaction or hurt performance.

---

## 6. Accessibility

- Maintain WCAG AA contrast: `--text-primary` on `--bg-panel` passes; `--text-muted` is for non-essential text only.
- All interactive elements keyboard-reachable with visible focus.
- Status never communicated by color alone (pair dots with text labels).
- Respect `prefers-reduced-motion`.

---

## 7. Tailwind Token Mapping (implementation note)

The palette and type scale will be encoded in `tailwind.config.ts` as custom theme tokens (`colors.bg.base`, `colors.accent.cyan`, etc.) and CSS variables in `globals.css`, so the whole system is one source of truth and theme tweaks propagate everywhere.
