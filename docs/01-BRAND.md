# Spectre — Brand

**Last updated:** 2026-06-22

---

## 1. Name

**Spectre.**

A spectre is something present but unseen, a watchful presence, fitting for a tool that quietly tracks your fleet, your trades, and your org in the background. It is short, sharp, sci-fi, and reads well as a minimal logo mark.

### Pronunciation & spelling
- Spelled **Spectre** (British spelling), not "Specter."
- Always capitalized as a proper noun: "Spectre," never "spectre" mid-sentence when referring to the product.

### Domain note
`spectre.com` is almost certainly unavailable (the name is common in software, including the CPU vulnerability). Likely domain candidates:
- `spectre.app`
- `playspectre.com` / `playspectre.app`
- `spectreverse.com` (nods to "the 'Verse")
- `getspectre.app`

Domain is **not yet locked.** Build the brand around the word "Spectre" and resolve the domain before launch.

---

## 2. Trademark & Legal Constraints (IMPORTANT)

Spectre is an unofficial, fan-made tool. To stay clear of Cloud Imperium Games (CIG) and Roberts Space Industries (RSI) intellectual property:

**Do NOT use in the product name, logo, or branding:**
- "Star Citizen," "Squadron 42," "RSI," "CIG," "UEE," or any official faction/manufacturer names.
- Ship names (Carrack, Hornet, etc.) as product/brand identifiers.
- Official CIG logos, fonts, or art assets.

**Do:**
- Reference Star Citizen descriptively in body copy ("a companion tool for Star Citizen") — descriptive use is fine.
- Display a clear disclaimer in the footer of every page:
  > *Spectre is an unofficial, community-made tool and is not affiliated with Cloud Imperium Games or Roberts Space Industries. All game content and materials are property of their respective owners.*
- Credit UEX as the data source where UEX data is shown, per their API badge guidance.

---

## 3. Logo

### Concept
A minimal **HUD-style mark** that reads instantly at small sizes (favicon, Discord avatar) and large (landing hero). The placeholder in the mockup is a downward chevron/triangle inside a bracket frame — Spectre's mark should be in that family but ownable.

### Direction (to be designed)
The wordmark is set in **Orbitron** (the primary display font), uppercase, with generous letter-spacing: `S P E C T R E`. The icon mark is a geometric glyph evoking a stealthed/ghosted ship reticle or a radar sweep, rendered as a thin-stroke outline in the cyan accent (`#00C6FF`) on a near-black field.

Two lockups:
1. **Horizontal:** mark + wordmark side by side (for the top nav).
2. **Stacked / mark-only:** for favicon, Discord avatar, mobile.

A first SVG draft of the mark accompanies this documentation (`spectre-logo.svg`). It is a starting point, not final.

### Logo rules
- Minimum clear space around the mark = the height of the "S" in the wordmark.
- Never stretch, recolor outside the palette, or add drop shadows/gradients beyond the subtle glow used in the design system.
- On light backgrounds (rare; the app is dark-mode-first), use the mark in `#070B12` (near-black).

---

## 4. Voice & Tone

Spectre speaks like a **competent flight computer**: precise, terse, no fluff. It respects that the user is an experienced pilot.

- **Concise.** Short labels, no hand-holding. "Add ship," not "Click here to add a ship to your fleet!"
- **Confident, not cute.** Avoid jokey copy and excessive personality in core UI.
- **Sci-fi flavor, sparingly.** Occasional in-universe flavor in empty states or loading text is welcome ("Scanning the 'Verse…"), but never at the expense of clarity.
- **Honest.** When data is community-sourced and may be stale, say so.

### Microcopy examples
| Context | Copy |
|---------|------|
| Empty fleet | "No ships in your hangar yet." |
| Loading | "Pulling live data…" |
| Stale data warning | "Prices last synced 4h ago. Community-reported." |
| Error | "Couldn't reach UEX. Showing cached data." |

---

## 5. Typography (brand level)

| Role | Typeface | Use |
|------|----------|-----|
| Display / headings | **Orbitron** | Logo, page titles, stat numbers, nav. |
| Body / UI | **Rajdhani** or **Inter** | Paragraphs, tables, form labels. |

> Note: the mockup labels the secondary font "Raleway." Raleway works but is soft for a HUD. **Rajdhani** (condensed, technical) is recommended as the secondary for a sharper, more cockpit-like feel. Final call documented in `04-DESIGN-SYSTEM.md`. Inter is the safe fallback for dense data tables where readability beats flavor.
