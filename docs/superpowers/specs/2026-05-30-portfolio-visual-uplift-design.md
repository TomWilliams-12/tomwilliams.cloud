# Portfolio Visual Uplift — Design Spec

**Date:** 2026-05-30
**Site:** tomwilliams.cloud (Hugo static site, extended ≥ v0.139.0; dev box runs v0.157.0)
**Goal:** Replace the generic "dark dev-portfolio" look with a distinctive, memorable *editorial-technical* design that stays credible for an enterprise infrastructure audience. The site's job is to convert recruiter / hiring-manager traffic into contact.

---

## 1. Design direction (locked via visual brainstorm)

**"Editorial-technical."** Refined and structured — reads like an engineering artifact made by someone with taste, *without* costume gimmicks (no crop-marks, part numbers, fake terminals, streaming logs, glow orbs, or gradient clip-text).

The identity is carried by a small set of recurring motifs, not by effects:

- A **profile "spec card"** — the signature component. A bordered panel with mono key / value rows, a live "Open to work" status, and cert pills. Recurs (in adapted form) on the hero, project pages, and certs.
- **Mono micro-labels** — section kickers (`// case studies`) and `KEY` labels in Geist Mono, uppercase, letter-spaced, teal.
- **Index numbers** — `01 / 02 / 03` on project cards, a subtle `01 — HOME` page index bottom-left.
- **Hairline structure** — 1px dividers between sections and list rows; bordered panels instead of floaty shadows.
- **Restrained motion** — section-link hover, card lift + teal edge-rule that scales in, hover arrow, status-dot pulse, theme cross-fade. One tasteful staggered hero reveal on load. No scattered micro-animations.

### Dual theme + toggle (locked)

Both a **light** (warm paper) and **dark** (warm-neutral near-black) theme, switchable via a toggle in the nav.

- **Default: dark.**
- Respect the visitor's OS preference (`prefers-color-scheme`) on first visit.
- Remember their explicit toggle choice in `localStorage` thereafter.
- Apply the theme **before first paint** via a tiny inline `<head>` script to avoid a flash of the wrong theme.

---

## 2. Design tokens

Replaces the current `:root` block in `static/css/style.css`. Themes are selected by `data-theme` on `<html>`.

### Light (`[data-theme="light"]`)
```
--bg:#ece7db; --bg-2:#e4dece; --panel:#f3efe6;
--ink:#16140f; --ink-2:#574f42;
--line:#d6cfbd; --line-2:#c3baa4;
--teal:#0d9488; --teal-2:#0b7d73; --teal-soft:rgba(13,148,136,.10);
--shadow:0 14px 38px rgba(60,50,30,.12);
```

### Dark (`[data-theme="dark"]`) — default
```
--bg:#101110; --bg-2:#171917; --panel:#181a18;
--ink:#e9e7df; --ink-2:#a4a195;
--line:#262825; --line-2:#33352f;
--teal:#2dd4bf; --teal-2:#5eead4; --teal-soft:rgba(45,212,191,.13);
--shadow:0 16px 44px rgba(0,0,0,.5);
```

The old mint `#0ff0b3` and the blue-black palette are **retired**. Accent is a single unified **teal** with a per-theme shade so the brand holds across the toggle. `theme-color` meta and `og`/favicon assets stay as-is (favicon/OG were done in the prior pass).

---

## 3. Typography

- **Display + body:** Geist
- **Mono / technical labels:** Geist Mono
- Loaded via Google Fonts (replacing the current Outfit + JetBrains Mono import). Self-hosting is a possible later perf optimisation, out of scope here.
- **Old fonts (Outfit, JetBrains Mono) retired.**

Scale (the larger scale validated in the mockups):
- `h1` hero: `clamp(2.7rem, 5.6vw, 4.4rem)`, weight 700, line-height .98, tracking -.04em
- Section title: `2rem`, weight 700, tracking -.03em
- Lede: `1.18rem`
- Body / card text: `.92rem`–`1rem`
- Spec table value: `~1.02rem`; key (mono): `.74rem`
- Section links / nav: `.8rem`–`.92rem` mono

---

## 4. Page-by-page application

> **Constraint:** Do **not** edit project case-study *body* content (`content/projects/*.md`) — several hold placeholder metrics (e.g. "X TB" in `fsx-migration.md`) the owner is filling in separately. Layout/styling of those pages is fair game. Blog body content also left untouched.

### 4.1 `layouts/_default/baseof.html`
- Add the **no-flash theme bootstrap** inline script in `<head>` (reads `localStorage` → else `prefers-color-scheme` → sets `data-theme` before paint; default dark).
- Swap font import to Geist + Geist Mono.
- Keep: `<title>` logic, `seo.html` partial, GA snippet, mobile-nav script, contact-form AJAX script.

### 4.2 `layouts/partials/nav.html`
- `TW` monogram tile + `tomwilliams.cloud` wordmark (mono).
- Mono uppercase nav links (existing menu).
- **Theme toggle** (☀/☾) added to the nav; wired to a small JS handler (sets `data-theme`, writes `localStorage`, updates `aria-pressed`).
- Preserve the `resumeReady`-gated "Download CV" CTA.
- Mobile: toggle remains reachable; nav collapses as today.

### 4.3 `layouts/page/home.html`
- **Hero:** meta-line kicker (`Senior Cloud Engineer · AWS 5×`), headline with a teal underline-accent word, lede, two CTAs (`View Projects`, `Get in touch`; plus `Download CV` when `resumeReady`), and the **profile spec card** on the right. Card pulls from existing params: `availability` → live status row, `location` → "Based", plus Role / Stack / Scale / Certs. Preserve availability + location conditionals.
- **About teaser, Featured Projects, Certifications, Latest Posts, Contact** restyled to the system below.
- **Contact** keeps the Formspree form, honeypot, AJAX status, and direct mailto/LinkedIn fallbacks exactly (functionality untouched; only styling).
- One staggered fade-up reveal on hero children on load (CSS, respecting `prefers-reduced-motion`).

### 4.4 Projects — `layouts/projects/list.html`, `single.html`, home featured block
- **Cards:** flex-column panel cards with mono index, teal tag, title, summary, tech pills pinned to a bottom footer row (bottom-aligned across the grid), hover lift + teal edge-rule + arrow.
- **Single (case study):** restyle header (tag, title, summary, tech pills) and `.post-content` typography (headings, code blocks, lists, blockquotes) to the new tokens/fonts. **Body content unchanged.**

### 4.5 Certifications — home section + `layouts/page/about.html`
- Credential **rack**: panel cards framing the **real Credly badge image**, with name + level pill caption, hover lift.
- **Decision (needs confirm):** move from Credly's `<iframe>` embed to rendering the **badge image directly**, linking out to the Credly verification page. Faster, themeable, sits cleanly in the card. Requires per-cert `image` + `verify_url` (or `badge_url`) added to `data/certs.yaml` (and the badge PNGs in `static/images/certs/` if we self-host the images). **Fallback:** if asset wrangling isn't wanted now, keep the existing iframe embed inside the restyled card. *(See Open Questions.)*

### 4.6 Blog — `layouts/blog/list.html`, `single.html`, home latest block
- **List:** editorial rows (mono date column + title + 1–2 line summary + tag pill), hairline dividers, hover background + title→teal. Replaces the card grid.
- **Single:** restyle post header + `.post-content` typography. **Body content unchanged.**

### 4.7 `layouts/partials/footer.html`
- Restyle to tokens (hairline top border, mono social links, teal hover). Content unchanged.

### 4.8 `static/css/style.css`
- Rewrite `:root`/theme tokens, base, nav, hero, spec card, sections, project cards, certs rack, blog list, post typography, contact, footer, animations, responsive — all on the new system.
- Keep: smooth scroll, custom scrollbar (recoloured), reduced-motion handling. **Drop** the heavy noise overlay (replace with subtle/no texture; the structure now carries the look).

---

## 5. What is explicitly preserved (prior-pass work — do not regress)

- `layouts/partials/seo.html` (OG/Twitter/JSON-LD) — untouched.
- Favicon / apple-touch-icon / OG image — untouched.
- Hero availability badge + location (re-expressed in the spec card, same params).
- Contact: mailto/LinkedIn fallbacks, Formspree, honeypot, AJAX status.
- `resumeReady`-gated Download CV CTA (nav + hero).
- GA4 snippet, RSS outputs, menu config, `data/certs.yaml`, tags taxonomy.

---

## 6. Constraints & verification

- **Build must stay green:** `hugo --gc --minify`.
- **Visual verification before done:** run `hugo server`, check **homepage + a project page + a blog page**, both themes, at desktop and **mobile** widths (≤ 780px).
- **Accessibility:** maintain colour contrast in both themes; `prefers-reduced-motion` respected; toggle has an accessible label / `aria-pressed`.
- No new build tooling — stays a plain Hugo + single CSS file site.

---

## 7. Out of scope

- Editing project/blog case-study body copy (owner is filling placeholder metrics separately).
- New content/sections, CMS changes, analytics changes.
- Self-hosting fonts; image optimisation pipeline.

---

## 8. Open questions for review

1. **Certs rendering** — switch to direct badge images + verification links (preferred; needs `image`/`verify_url` in `data/certs.yaml` and/or PNGs in `static/`), or keep the existing Credly iframe inside the restyled card for now?
2. **Light-theme texture** — fully flat, or a *very* subtle paper grain on the light theme only? (Default plan: flat/minimal.)
3. Anything in the validated mockups you want tuned before the implementation plan (spacing, the "holds up under load" headline copy, etc.)?
