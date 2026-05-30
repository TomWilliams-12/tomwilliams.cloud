# Portfolio Visual Uplift Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Replace the generic dark dev-portfolio look on tomwilliams.cloud with a distinctive *editorial-technical* design — dual light/dark theme, Geist + Geist Mono, teal accent, a recurring "spec card" motif — that converts recruiter traffic into contact.

**Architecture:** Plain Hugo (extended) static site. A single hand-written stylesheet (`static/css/style.css`) holds all design tokens (CSS variables, themed via `data-theme` on `<html>`) and component styles. Templates in `layouts/` are restyled to a shared class contract. A tiny inline `<head>` script sets the theme before first paint (no FOUC); a nav toggle persists choice to `localStorage`.

**Tech Stack:** Hugo extended (dev v0.157.0), HTML templates, vanilla CSS + minimal vanilla JS, Google Fonts (Geist, Geist Mono), Credly badge images.

**Design source of truth:** [`docs/superpowers/specs/2026-05-30-portfolio-visual-uplift-design.md`](../specs/2026-05-30-portfolio-visual-uplift-design.md). Validated mockups live (gitignored) under `.superpowers/brainstorm/26053-1780128701/content/` — `final-hero-v2.html` and `sections-v2.html` are the visual targets.

---

## Class-name contract (used across all tasks — keep exact)

Layout: `.container`, `.site-nav .inner`, `.section-head`, `.section-kicker`, `.section-title`, `.section-link`.
Nav: `.site-nav`, `.brand`, `.brand .mono`, `.brand .dim`, `.nav-links`, `.theme-toggle`, `.theme-toggle button[data-theme-set]`, `.nav-cta`, `.nav-toggle`.
Hero: `.hero`, `.hero-main`, `.meta-line`, `.hero h1 .u`, `.hero h1 .u b`, `.lede`, `.cta-row`, `.btn`, `.btn-1`, `.btn-2`.
Spec card: `.spec`, `.spec-head`, `.spec-head .live`, `.spec .row`, `.spec .row .k`, `.spec .row .v`, `.spec .v .mono`, `.pill`.
Projects: `.proj-grid`, `.proj-card`, `.proj-top`, `.proj-idx`, `.proj-tag`, `.proj-foot`, `.stack-row`, `.proj-arrow`.
Certs: `.cert-grid`, `.cert-card`, `.cert-badge`, `.cert-name`, `.cert-level`.
Blog: `.blog-list`, `.blog-row`, `.blog-date`, `.blog-main`, `.blog-tag`.
Post pages: `.post-header`, `.post-date`, `.post-summary`, `.post-content`.
Contact: `.contact-section`, `.contact-form`, `.honeypot`, `.form-status`, `.form-success`, `.form-error`, `.contact-or`, `.contact-direct`.
Footer: `.site-footer`, `.social-links`.
Misc: `.page-index`.

---

## Verification model (this is a visual/markup project, not unit-tested)

Every task's "test" is two gates:
1. **Build green:** `hugo --gc --minify` exits 0 with no template errors.
2. **Visual check:** `hugo server` running; open the relevant page and confirm against the mockup, in **both themes** (toggle) and at desktop + mobile (≤780px) widths where relevant.

Keep `hugo server` running in a background terminal throughout (`hugo server -D`). Commit after each task.

---

## Task 1: Theme bootstrap + font swap in baseof

**Files:**
- Modify: `layouts/_default/baseof.html`

- [ ] **Step 1: Set default theme + add no-flash bootstrap script**

In `layouts/_default/baseof.html`, change the opening html tag from `<html lang="en">` to `<html lang="en" data-theme="dark">`.

Then immediately after `<head>` (before the `<title>`), add the bootstrap script so the stored/OS theme is applied before first paint:

```html
  <script>
    (function () {
      try {
        var t = localStorage.getItem('theme');
        if (t !== 'light' && t !== 'dark') {
          t = window.matchMedia('(prefers-color-scheme: light)').matches ? 'light' : 'dark';
        }
        document.documentElement.setAttribute('data-theme', t);
      } catch (e) {
        document.documentElement.setAttribute('data-theme', 'dark');
      }
    })();
  </script>
```

- [ ] **Step 2: Swap the stylesheet/font loading is handled in CSS**

No font `<link>` needed in baseof — fonts are `@import`ed at the top of `style.css` (Task 2). Leave the existing `<link rel="stylesheet" href="/css/style.css">` as-is. Confirm it still points there.

- [ ] **Step 3: Add the theme-toggle handler script**

At the bottom of `baseof.html`, inside the existing `DOMContentLoaded` handler `<script>` (the one with the mobile-nav toggle), add the theme-toggle wiring right after the mobile-nav block:

```javascript
      // Theme toggle
      const root = document.documentElement;
      const setTheme = (t) => {
        root.setAttribute('data-theme', t);
        try { localStorage.setItem('theme', t); } catch (e) {}
        document.querySelectorAll('.theme-toggle button').forEach((b) => {
          const on = b.dataset.themeSet === t;
          b.classList.toggle('on', on);
          b.setAttribute('aria-pressed', on ? 'true' : 'false');
        });
      };
      document.querySelectorAll('.theme-toggle button').forEach((b) => {
        b.addEventListener('click', () => setTheme(b.dataset.themeSet));
      });
      // sync initial button state with the theme set by the head bootstrap
      setTheme(root.getAttribute('data-theme') || 'dark');
```

- [ ] **Step 4: Build**

Run: `hugo --gc --minify`
Expected: exit 0, no errors. View `public/index.html` source — `<html ... data-theme="dark">` and the bootstrap script are present.

- [ ] **Step 5: Commit**

```bash
git add layouts/_default/baseof.html
git commit -m "feat(theme): no-flash theme bootstrap + toggle handler in baseof"
```

---

## Task 2: Replace the stylesheet (full design system)

This drops in the complete new `static/css/style.css`. After this task, components whose templates still use old markup will look unstyled until their template task lands (Tasks 3–9) — that is expected; the build stays green throughout.

**Files:**
- Overwrite: `static/css/style.css`

- [ ] **Step 1: Overwrite `static/css/style.css` with the complete stylesheet below**

```css
/* ========================================
   Tom Williams — Portfolio
   Editorial-technical, dual-theme
   ======================================== */
@import url('https://fonts.googleapis.com/css2?family=Geist:wght@300;400;500;600;700;800;900&family=Geist+Mono:wght@400;500;600&display=swap');

*, *::before, *::after { margin: 0; padding: 0; box-sizing: border-box; }

:root[data-theme="light"] {
  --bg:#ece7db; --bg-2:#e4dece; --panel:#f3efe6;
  --ink:#16140f; --ink-2:#574f42;
  --line:#d6cfbd; --line-2:#c3baa4;
  --teal:#0d9488; --teal-2:#0b7d73; --teal-soft:rgba(13,148,136,.10);
  --shadow:0 14px 38px rgba(60,50,30,.12);
  --on-teal:#08120f;
}
:root[data-theme="dark"] {
  --bg:#101110; --bg-2:#171917; --panel:#181a18;
  --ink:#e9e7df; --ink-2:#a4a195;
  --line:#262825; --line-2:#33352f;
  --teal:#2dd4bf; --teal-2:#5eead4; --teal-soft:rgba(45,212,191,.13);
  --shadow:0 16px 44px rgba(0,0,0,.5);
  --on-teal:#08120f;
}
:root {
  --font-sans:'Geist', system-ui, sans-serif;
  --font-mono:'Geist Mono', ui-monospace, monospace;
  --max-width:1120px; --radius:12px; --radius-sm:7px;
}

html { background:var(--bg); scroll-behavior:smooth; }
body {
  background:var(--bg); color:var(--ink);
  font-family:var(--font-sans); line-height:1.6; font-size:16px;
  overflow-x:hidden; -webkit-font-smoothing:antialiased;
  transition:background .4s ease, color .4s ease;
}

/* Paper grain — light theme only */
:root[data-theme="light"] body::before {
  content:''; position:fixed; inset:0; pointer-events:none; z-index:9999;
  opacity:.045; mix-blend-mode:multiply;
  background-image:url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 200 200'%3E%3Cfilter id='n'%3E%3CfeTurbulence type='fractalNoise' baseFrequency='.9' numOctaves='2' stitchTiles='stitch'/%3E%3C/filter%3E%3Crect width='100%25' height='100%25' filter='url(%23n)'/%3E%3C/svg%3E");
}

::-webkit-scrollbar { width:10px; }
::-webkit-scrollbar-track { background:var(--bg); }
::-webkit-scrollbar-thumb { background:var(--line-2); border-radius:6px; border:3px solid var(--bg); }
::-webkit-scrollbar-thumb:hover { background:var(--teal); }

a { color:var(--teal); text-decoration:none; transition:color .2s; }
img { max-width:100%; height:auto; }
.container { max-width:var(--max-width); margin:0 auto; padding:0 40px; }

/* ========================================
   Navigation
   ======================================== */
.site-nav {
  position:fixed; top:0; left:0; right:0; z-index:100;
  background:color-mix(in srgb, var(--bg) 85%, transparent);
  backdrop-filter:blur(20px);
  border-bottom:1px solid var(--line);
}
.site-nav .inner {
  max-width:var(--max-width); margin:0 auto; padding:0 40px;
  display:flex; align-items:center; justify-content:space-between; height:62px;
}
.brand { font-family:var(--font-mono); font-weight:600; font-size:.95rem; display:flex; align-items:center; gap:.55rem; color:var(--ink); }
.brand .mono { width:28px; height:28px; border:1.5px solid var(--ink); border-radius:7px; display:grid; place-items:center; font-size:.7rem; font-weight:700; }
.brand .dim { color:var(--ink-2); }
.nav-links { list-style:none; display:flex; align-items:center; gap:1.7rem; }
.nav-links a { font-family:var(--font-mono); font-size:.8rem; color:var(--ink-2); text-transform:uppercase; letter-spacing:.04em; transition:color .2s; }
.nav-links a:hover { color:var(--teal); }
.nav-cta { color:var(--teal) !important; border:1px solid var(--line-2); border-radius:var(--radius-sm); padding:.4rem .8rem; text-transform:none !important; letter-spacing:0 !important; }
.nav-cta:hover { border-color:var(--teal); }
.theme-toggle { display:inline-flex; border:1px solid var(--line-2); border-radius:100px; padding:3px; gap:2px; background:var(--bg-2); font-family:var(--font-mono); }
.theme-toggle button { border:none; background:none; cursor:pointer; font:inherit; font-size:.66rem; letter-spacing:.06em; text-transform:uppercase; color:var(--ink-2); padding:.34rem .72rem; border-radius:100px; transition:all .25s; }
.theme-toggle button.on { background:var(--teal); color:var(--on-teal); }
.nav-toggle { display:none; background:none; border:none; color:var(--ink); cursor:pointer; font-size:1.2rem; padding:.5rem; }

/* ========================================
   Section heads
   ======================================== */
section { padding:62px 0; }
.section-head { display:flex; align-items:flex-end; justify-content:space-between; gap:1rem; margin-bottom:34px; }
.section-kicker { font-family:var(--font-mono); font-size:.78rem; letter-spacing:.16em; text-transform:uppercase; color:var(--teal); margin-bottom:.6rem; display:flex; align-items:center; gap:.6rem; }
.section-kicker::before { content:''; width:24px; height:1.5px; background:var(--teal); }
.section-title { font-size:2rem; font-weight:700; letter-spacing:-.03em; }
.section-link { font-family:var(--font-mono); font-size:.92rem; font-weight:500; color:var(--ink-2); white-space:nowrap; }
.section-link:hover { color:var(--teal); }
.section-desc { color:var(--ink-2); max-width:600px; margin-top:.4rem; }

/* ========================================
   Hero
   ======================================== */
.hero { padding:130px 40px 56px; }
.hero .inner { max-width:var(--max-width); margin:0 auto; display:grid; grid-template-columns:1.1fr .9fr; gap:56px; align-items:center; }
.meta-line { font-family:var(--font-mono); font-size:.8rem; letter-spacing:.14em; text-transform:uppercase; color:var(--teal); margin-bottom:1.5rem; display:flex; align-items:center; gap:.7rem; }
.meta-line::before { content:''; width:30px; height:1.5px; background:var(--teal); }
.hero h1 { font-size:clamp(2.7rem,5.6vw,4.4rem); font-weight:700; line-height:.98; letter-spacing:-.04em; margin-bottom:1.4rem; }
.hero h1 .u { position:relative; white-space:nowrap; }
.hero h1 .u b { color:var(--teal); font-weight:700; }
.hero h1 .u::after { content:''; position:absolute; left:-.02em; right:-.02em; bottom:.08em; height:.16em; background:var(--teal-soft); z-index:-1; }
.lede { font-size:1.18rem; color:var(--ink-2); max-width:440px; margin-bottom:2rem; line-height:1.62; }
.cta-row { display:flex; gap:1rem; align-items:center; flex-wrap:wrap; }

/* Buttons */
.btn { font-family:var(--font-mono); font-size:.84rem; font-weight:500; padding:.85rem 1.4rem; border-radius:var(--radius-sm); transition:all .2s; display:inline-flex; align-items:center; gap:.5rem; border:1px solid transparent; cursor:pointer; }
.btn-1 { background:var(--teal); color:var(--on-teal); border-color:var(--teal); }
.btn-1:hover { background:var(--teal-2); border-color:var(--teal-2); }
.btn-2 { color:var(--ink); border-color:var(--line-2); background:transparent; }
.btn-2:hover { border-color:var(--teal); color:var(--teal); }

/* ========================================
   Spec card
   ======================================== */
.spec { background:var(--panel); border:1px solid var(--line-2); border-radius:13px; box-shadow:var(--shadow); overflow:hidden; }
.spec-head { display:flex; align-items:center; justify-content:space-between; padding:1rem 1.25rem; border-bottom:1px solid var(--line); font-family:var(--font-mono); font-size:.72rem; letter-spacing:.1em; text-transform:uppercase; color:var(--ink-2); }
.spec-head .live { display:flex; align-items:center; gap:.5rem; color:var(--teal); }
.spec-head .live i { width:8px; height:8px; border-radius:50%; background:var(--teal); animation:pulse 2.4s infinite; }
@keyframes pulse { 0% { box-shadow:0 0 0 0 var(--teal-soft); } 70% { box-shadow:0 0 0 8px transparent; } 100% { box-shadow:0 0 0 0 transparent; } }
.spec .row { display:grid; grid-template-columns:30% 70%; border-bottom:1px solid var(--line); }
.spec .row:last-child { border-bottom:none; }
.spec .row .k { font-family:var(--font-mono); font-size:.74rem; letter-spacing:.06em; text-transform:uppercase; color:var(--ink-2); padding:1.05rem 1.25rem; display:flex; align-items:center; }
.spec .row .v { font-size:1.02rem; font-weight:600; padding:1.05rem 1.25rem; display:flex; align-items:center; gap:.4rem; line-height:1.35; flex-wrap:wrap; }
.spec .v .mono { font-family:var(--font-mono); font-size:.92rem; font-weight:500; }
.pill { font-family:var(--font-mono); font-size:.72rem; border:1px solid var(--line-2); border-radius:5px; padding:.12rem .38rem; color:var(--ink-2); }

/* ========================================
   Projects
   ======================================== */
.proj-grid { display:grid; grid-template-columns:repeat(3,1fr); gap:18px; align-items:stretch; }
.proj-card { position:relative; display:flex; flex-direction:column; background:var(--panel); border:1px solid var(--line-2); border-radius:var(--radius); padding:26px 24px 22px; overflow:hidden; transition:transform .2s, border-color .2s, box-shadow .2s; }
.proj-card::before { content:''; position:absolute; left:0; top:0; bottom:0; width:2px; background:var(--teal); transform:scaleY(0); transform-origin:top; transition:transform .25s; }
.proj-card:hover { transform:translateY(-4px); border-color:var(--teal); box-shadow:var(--shadow); }
.proj-card:hover::before { transform:scaleY(1); }
.proj-top { display:flex; justify-content:space-between; align-items:center; margin-bottom:1.1rem; }
.proj-idx { font-family:var(--font-mono); font-size:.74rem; color:var(--ink-2); }
.proj-tag { font-family:var(--font-mono); font-size:.66rem; text-transform:uppercase; letter-spacing:.08em; color:var(--teal); background:var(--teal-soft); border:1px solid var(--teal-soft); padding:.18rem .5rem; border-radius:5px; }
.proj-card h3 { font-size:1.18rem; font-weight:600; letter-spacing:-.02em; margin-bottom:.6rem; line-height:1.25; }
.proj-card h3 a { color:var(--ink); }
.proj-card h3 a:hover { color:var(--teal); }
.proj-card p { color:var(--ink-2); font-size:.92rem; line-height:1.55; }
.proj-foot { margin-top:auto; padding-top:1.4rem; display:flex; align-items:center; justify-content:space-between; gap:1rem; }
.stack-row { display:flex; flex-wrap:wrap; gap:.35rem; }
.stack-row span { font-family:var(--font-mono); font-size:.68rem; color:var(--ink-2); border:1px solid var(--line-2); border-radius:5px; padding:.15rem .45rem; }
.proj-arrow { color:var(--teal); opacity:0; transform:translateX(-4px); transition:.25s; font-family:var(--font-mono); flex-shrink:0; }
.proj-card:hover .proj-arrow { opacity:1; transform:translateX(0); }

/* ========================================
   Certifications
   ======================================== */
.cert-grid { display:grid; grid-template-columns:repeat(5,1fr); gap:14px; }
.cert-card { display:flex; flex-direction:column; align-items:center; text-align:center; background:var(--panel); border:1px solid var(--line-2); border-radius:var(--radius); padding:22px 14px 18px; transition:transform .2s, border-color .2s; }
.cert-card:hover { transform:translateY(-3px); border-color:var(--teal); }
.cert-badge { width:92px; height:92px; object-fit:contain; margin-bottom:14px; }
.cert-name { font-size:.88rem; font-weight:600; line-height:1.3; margin-bottom:.5rem; color:var(--ink); }
.cert-level { font-family:var(--font-mono); font-size:.64rem; letter-spacing:.08em; text-transform:uppercase; color:var(--ink-2); border:1px solid var(--line-2); border-radius:4px; padding:.12rem .4rem; }

/* ========================================
   Blog list (editorial rows)
   ======================================== */
.blog-list { border-top:1px solid var(--line); }
.blog-row { display:grid; grid-template-columns:130px 1fr; gap:28px; align-items:start; padding:24px 8px; border-bottom:1px solid var(--line); transition:background .2s, padding .2s; }
.blog-row:hover { background:var(--panel); padding-left:18px; }
.blog-date { font-family:var(--font-mono); font-size:.76rem; color:var(--ink-2); letter-spacing:.04em; padding-top:.2rem; }
.blog-main h3 { font-size:1.22rem; font-weight:600; letter-spacing:-.015em; margin-bottom:.45rem; }
.blog-main h3 a { color:var(--ink); }
.blog-row:hover h3 a { color:var(--teal); }
.blog-main p { color:var(--ink-2); font-size:.92rem; line-height:1.55; max-width:640px; margin-bottom:.7rem; }
.blog-tag { font-family:var(--font-mono); font-size:.68rem; color:var(--teal); background:var(--teal-soft); border-radius:5px; padding:.16rem .5rem; margin-right:.35rem; }

/* ========================================
   Post pages (project + blog single)
   ======================================== */
.post-header { padding:140px 0 1.5rem; }
.post-header .post-date { font-family:var(--font-mono); font-size:.78rem; color:var(--teal); letter-spacing:.1em; text-transform:uppercase; margin-bottom:1rem; display:block; }
.post-header h1 { font-size:clamp(2rem,4.2vw,3rem); font-weight:700; letter-spacing:-.035em; max-width:780px; margin-bottom:1rem; line-height:1.05; }
.post-header .post-summary { color:var(--ink-2); max-width:620px; font-size:1.1rem; line-height:1.6; }
.post-content { max-width:760px; margin:0 auto; padding:2.5rem 40px 5rem; }
.post-content h2 { font-size:1.6rem; font-weight:700; margin:2.6rem 0 .9rem; letter-spacing:-.02em; }
.post-content h3 { font-size:1.22rem; font-weight:600; margin:2rem 0 .6rem; }
.post-content p { margin-bottom:1.3rem; color:var(--ink); font-size:1.02rem; line-height:1.75; }
.post-content a { color:var(--teal); text-decoration:underline; text-underline-offset:3px; }
.post-content ul, .post-content ol { margin:0 0 1.3rem 1.4rem; color:var(--ink); font-size:1.02rem; line-height:1.7; }
.post-content li { margin-bottom:.45rem; }
.post-content code { font-family:var(--font-mono); font-size:.85em; background:var(--bg-2); padding:.15rem .4rem; border-radius:4px; color:var(--teal); }
.post-content pre { background:var(--bg-2); border:1px solid var(--line); border-radius:var(--radius-sm); padding:1.2rem 1.5rem; overflow-x:auto; margin-bottom:1.5rem; }
.post-content pre code { background:none; padding:0; font-size:.82rem; color:var(--ink); }
.post-content blockquote { border-left:3px solid var(--teal); padding-left:1.2rem; margin:1.5rem 0; color:var(--ink-2); font-style:italic; }

/* ========================================
   Contact
   ======================================== */
.contact-section { text-align:center; }
.contact-form { max-width:520px; margin:2rem auto 0; display:flex; flex-direction:column; gap:.8rem; text-align:left; }
.contact-form input, .contact-form textarea { background:var(--panel); border:1px solid var(--line-2); border-radius:var(--radius-sm); padding:.9rem 1.1rem; color:var(--ink); font-family:var(--font-sans); font-size:.95rem; transition:border-color .2s; outline:none; width:100%; }
.contact-form input:focus, .contact-form textarea:focus { border-color:var(--teal); }
.contact-form textarea { resize:vertical; min-height:130px; }
.contact-form .btn { align-self:center; margin-top:.5rem; }
.form-status { max-width:520px; margin:1rem auto 0; padding:1rem 1.4rem; border-radius:var(--radius-sm); font-size:.95rem; }
.form-success { background:var(--teal-soft); color:var(--teal); border:1px solid var(--teal); }
.form-error { background:rgba(220,60,40,.1); color:#d83a24; border:1px solid rgba(220,60,40,.3); }
.honeypot { position:absolute !important; left:-9999px !important; width:1px; height:1px; opacity:0; pointer-events:none; }
.contact-or { font-family:var(--font-mono); font-size:.74rem; color:var(--ink-2); letter-spacing:.12em; text-transform:uppercase; margin:1.8rem 0 1rem; }
.contact-direct { display:flex; gap:1rem; justify-content:center; flex-wrap:wrap; }

/* ========================================
   Footer
   ======================================== */
.site-footer { border-top:1px solid var(--line); padding:2.4rem 40px; text-align:center; color:var(--ink-2); font-size:.8rem; }
.social-links { display:flex; justify-content:center; gap:1.5rem; margin-bottom:.9rem; }
.social-links a { font-family:var(--font-mono); color:var(--ink-2); font-size:.8rem; text-transform:uppercase; letter-spacing:.04em; transition:color .2s; }
.social-links a:hover { color:var(--teal); }

/* Page index marker */
.page-index { position:fixed; left:14px; bottom:12px; z-index:50; font-family:var(--font-mono); font-size:.62rem; letter-spacing:.12em; color:var(--ink-2); opacity:.5; pointer-events:none; }

/* ========================================
   Animations
   ======================================== */
@keyframes fadeUp { from { opacity:0; transform:translateY(16px); } to { opacity:1; transform:translateY(0); } }
.hero .inner > * { animation:fadeUp .6s ease-out both; }
.hero .hero-main > * { animation:fadeUp .6s ease-out both; }
.hero .hero-main > *:nth-child(1) { animation-delay:.05s; }
.hero .hero-main > *:nth-child(2) { animation-delay:.13s; }
.hero .hero-main > *:nth-child(3) { animation-delay:.22s; }
.hero .hero-main > *:nth-child(4) { animation-delay:.32s; }
.hero .spec { animation-delay:.34s; }

@media (prefers-reduced-motion: reduce) {
  * { animation:none !important; transition:none !important; scroll-behavior:auto !important; }
}

/* ========================================
   Responsive
   ======================================== */
@media (max-width:880px) {
  .proj-grid { grid-template-columns:repeat(2,1fr); }
  .cert-grid { grid-template-columns:repeat(3,1fr); }
}
@media (max-width:780px) {
  .hero { padding:110px 24px 44px; }
  .hero .inner { grid-template-columns:1fr; gap:34px; }
  .container { padding:0 24px; }
  .site-nav .inner { padding:0 24px; }
  section { padding:44px 0; }
  .nav-links { display:none; }
  .nav-links.open { display:flex; flex-direction:column; align-items:flex-start; position:absolute; top:62px; left:0; right:0; background:var(--bg); padding:1.2rem 24px; gap:1rem; border-bottom:1px solid var(--line); }
  .nav-toggle { display:block; }
  .section-head { flex-direction:column; align-items:flex-start; }
  .proj-grid, .cert-grid { grid-template-columns:1fr; }
  .blog-row { grid-template-columns:1fr; gap:.5rem; }
  .post-content { padding:2rem 24px 4rem; }
}
```

- [ ] **Step 2: Build**

Run: `hugo --gc --minify`
Expected: exit 0. (CSS-only change cannot break the Hugo build; this just confirms nothing references a now-removed asset.)

- [ ] **Step 3: Commit**

```bash
git add static/css/style.css
git commit -m "feat(css): replace stylesheet with editorial-technical dual-theme system"
```

---

## Task 3: Nav partial + theme toggle + mobile menu

**Files:**
- Overwrite: `layouts/partials/nav.html`

- [ ] **Step 1: Overwrite `layouts/partials/nav.html`**

```html
<nav class="site-nav">
  <div class="inner">
    <a href="/" class="brand"><span class="mono">TW</span>tomwilliams<span class="dim">.cloud</span></a>
    <button class="nav-toggle" aria-label="Toggle navigation" aria-expanded="false">☰</button>
    <ul class="nav-links">
      {{ range .Site.Menus.main }}
      <li><a href="{{ .URL }}">{{ .Name }}</a></li>
      {{ end }}
      {{ if .Site.Params.resumeReady }}
      <li><a href="{{ .Site.Params.resume }}" class="nav-cta" download>Download CV ↓</a></li>
      {{ end }}
      <li>
        <div class="theme-toggle" role="group" aria-label="Colour theme">
          <button type="button" data-theme-set="light" aria-pressed="false">☀ Light</button>
          <button type="button" data-theme-set="dark" aria-pressed="true">☾ Dark</button>
        </div>
      </li>
    </ul>
  </div>
</nav>
```

- [ ] **Step 2: Confirm the mobile-nav JS targets the new class**

In `layouts/_default/baseof.html`, the mobile-nav script selects `.site-nav ul`. Confirm it still works — `.nav-links` is a `ul`, so `document.querySelector('.site-nav ul')` matches. If you prefer precision, change that selector to `.site-nav .nav-links`. Either works.

- [ ] **Step 3: Build + visual check**

Run: `hugo --gc --minify` → exit 0.
With `hugo server` running, open `/`: confirm the brand monogram, uppercase mono nav links, and the ☀/☾ toggle render. Click the toggle — the whole page should cross-fade between themes and the active segment highlights teal. Reload — the chosen theme persists. Narrow to ≤780px — links collapse behind ☰ and the toggle is reachable in the open menu.

- [ ] **Step 4: Commit**

```bash
git add layouts/partials/nav.html layouts/_default/baseof.html
git commit -m "feat(nav): brand monogram, mono links, working theme toggle"
```

---

## Task 4: Home hero + profile spec card

**Files:**
- Modify: `layouts/page/home.html` (hero `<section>` only, lines ~3–20)

- [ ] **Step 1: Replace the hero `<section>` in `layouts/page/home.html`**

Replace the existing `<!-- Hero -->` section (from `<section class="hero">` through its closing `</section>`) with:

```html
<!-- Hero -->
<section class="hero">
  <div class="inner">
    <div class="hero-main">
      <div class="meta-line">Senior Cloud Engineer · AWS 5×</div>
      <h1>Infrastructure that <span class="u"><b>holds up</b></span> under load.</h1>
      <p class="lede">
        Multi-account AWS, infrastructure-as-code, and legacy migrations —
        engineered so the graphs stay flat and no one gets paged at 3am.
      </p>
      <div class="cta-row">
        {{ if .Site.Params.resumeReady }}<a href="{{ .Site.Params.resume }}" class="btn btn-1" download>Download CV ↓</a>{{ end }}
        <a href="/projects/" class="btn {{ if .Site.Params.resumeReady }}btn-2{{ else }}btn-1{{ end }}">View Projects →</a>
        <a href="#contact" class="btn btn-2">Get in touch</a>
      </div>
    </div>
    <div class="spec">
      <div class="spec-head">
        <span>// profile</span>
        {{ with .Site.Params.availability }}<span class="live"><i></i>{{ . }}</span>{{ end }}
      </div>
      <div class="row"><div class="k">Role</div><div class="v">Senior Cloud Engineer</div></div>
      <div class="row"><div class="k">Stack</div><div class="v"><span class="mono">AWS · Terraform · Lambda</span></div></div>
      <div class="row"><div class="k">Scale</div><div class="v">Multi-account · Landing Zone</div></div>
      <div class="row"><div class="k">Certs</div><div class="v"><span class="pill">PRO×2</span><span class="pill">SPEC</span><span class="pill">ASSOC×2</span></div></div>
      {{ with .Site.Params.location }}<div class="row"><div class="k">Based</div><div class="v">{{ . }}</div></div>{{ end }}
    </div>
  </div>
</section>
```

- [ ] **Step 2: Build + visual check**

Run: `hugo --gc --minify` → exit 0.
On `/`: hero is left headline + lede + CTAs, right profile spec card with a pulsing teal "Open to opportunities" status. The `holds up` underline accent shows. Staggered fade-up plays on load. Toggle themes — both read well. At ≤780px the spec card stacks under the hero text.

- [ ] **Step 3: Commit**

```bash
git add layouts/page/home.html
git commit -m "feat(home): editorial hero + profile spec card"
```

---

## Task 5: Home about teaser + Featured Projects + Projects list page

**Files:**
- Modify: `layouts/page/home.html` (about section + featured-projects section)
- Overwrite: `layouts/projects/list.html`

- [ ] **Step 1: Replace the About + Featured Projects sections in `layouts/page/home.html`**

Replace the existing `<!-- About -->` section and `<!-- Featured Projects -->` section with:

```html
<!-- About -->
<section>
  <div class="container">
    <div class="section-head">
      <div>
        <div class="section-kicker">about</div>
        <h2 class="section-title">Who I am</h2>
      </div>
      <a href="/about/" class="section-link">full bio →</a>
    </div>
    <p class="lede" style="max-width:760px;">
      Senior Cloud Engineer based in the UK with 5+ years building and running AWS infrastructure at scale.
      I came up through IT support and sysadmin work before going all-in on cloud — and I hold five active
      AWS certifications across the associate, professional, and specialty tracks. I like turning brittle,
      manual infrastructure into automated systems teams can trust.
    </p>
  </div>
</section>

<!-- Featured Projects -->
<section>
  <div class="container">
    <div class="section-head">
      <div>
        <div class="section-kicker">case studies</div>
        <h2 class="section-title">Featured Projects</h2>
      </div>
      <a href="/projects/" class="section-link">all projects →</a>
    </div>
    <div class="proj-grid">
      {{ range $i, $p := first 3 (where .Site.RegularPages "Section" "projects") }}
      <article class="proj-card">
        <div class="proj-top">
          <span class="proj-idx">{{ printf "%02d" (add $i 1) }}</span>
          {{ with $p.Params.tag }}<span class="proj-tag">{{ . }}</span>{{ end }}
        </div>
        <h3><a href="{{ $p.Permalink }}">{{ $p.Title }}</a></h3>
        <p>{{ $p.Summary }}</p>
        <div class="proj-foot">
          <div class="stack-row">{{ range first 3 $p.Params.tech }}<span>{{ . }}</span>{{ end }}</div>
          <a class="proj-arrow" href="{{ $p.Permalink }}" aria-label="Read {{ $p.Title }}">→</a>
        </div>
      </article>
      {{ end }}
    </div>
  </div>
</section>
```

- [ ] **Step 2: Overwrite `layouts/projects/list.html`**

```html
{{ define "main" }}
<section style="padding-top:130px;">
  <div class="container">
    <div class="section-head">
      <div>
        <div class="section-kicker">case studies</div>
        <h2 class="section-title">Projects</h2>
      </div>
    </div>
    <p class="section-desc" style="margin-bottom:34px;">Real-world infrastructure challenges I've solved. Details sanitised for confidentiality.</p>
    <div class="proj-grid">
      {{ range $i, $p := .Pages.ByDate.Reverse }}
      <article class="proj-card">
        <div class="proj-top">
          <span class="proj-idx">{{ printf "%02d" (add $i 1) }}</span>
          {{ with $p.Params.tag }}<span class="proj-tag">{{ . }}</span>{{ end }}
        </div>
        <h3><a href="{{ $p.Permalink }}">{{ $p.Title }}</a></h3>
        <p>{{ $p.Summary }}</p>
        <div class="proj-foot">
          <div class="stack-row">{{ range first 3 $p.Params.tech }}<span>{{ . }}</span>{{ end }}</div>
          <a class="proj-arrow" href="{{ $p.Permalink }}" aria-label="Read {{ $p.Title }}">→</a>
        </div>
      </article>
      {{ end }}
    </div>
  </div>
</section>
{{ end }}
```

- [ ] **Step 3: Build + visual check**

Run: `hugo --gc --minify` → exit 0.
On `/`: about teaser reads as a confident lede; the three project cards bottom-align their tech pills and show the teal edge-rule + arrow on hover. On `/projects/`: same card system, full list. Both themes; ≤780px collapses to one column.

- [ ] **Step 4: Commit**

```bash
git add layouts/page/home.html layouts/projects/list.html
git commit -m "feat(projects): editorial cards on home + projects list"
```

---

## Task 6: Certifications — resolve real Credly images + render rack

The current site uses Credly's `<iframe>` embed. Switch to the badge image (faster, themeable), linking to the verification page, with the iframe kept as a per-cert fallback.

**Files:**
- Modify: `data/certs.yaml` (add `image` + `verify_url` per cert)
- Create: `static/images/certs/*.png` (resolved badge images)
- Modify: `layouts/page/home.html` (certifications section)
- Modify: `layouts/page/about.html` (verified-badges block)

- [ ] **Step 1: Resolve each badge's image + verification URL**

For each `badge_id` in `data/certs.yaml`, the public badge page is `https://www.credly.com/badges/<badge_id>/public_url`. Fetch it and read the `og:image` meta. Run this helper (it downloads each badge PNG and prints the YAML lines to paste):

```bash
mkdir -p static/images/certs
for id in $(grep badge_id data/certs.yaml | sed -E 's/.*badge_id: *"?([a-f0-9-]+)"?.*/\1/'); do
  page="https://www.credly.com/badges/${id}/public_url"
  img=$(curl -fsSL -A "Mozilla/5.0" "$page" | grep -oE 'property="og:image" content="[^"]+"' | head -1 | sed -E 's/.*content="([^"]+)".*/\1/')
  if [ -n "$img" ]; then
    curl -fsSL -A "Mozilla/5.0" "$img" -o "static/images/certs/${id}.png" && echo "OK  ${id}  -> static/images/certs/${id}.png"
  else
    echo "MISS ${id} (keep iframe fallback)"
  fi
done
```

Expected: 5 `OK` lines and 5 PNGs in `static/images/certs/`. If any `MISS`, that cert will use the iframe fallback automatically (Step 3 handles it).

- [ ] **Step 2: Add `image` + `verify_url` to each entry in `data/certs.yaml`**

For every cert where Step 1 produced a PNG, add two keys (use that cert's own `badge_id`). Example for the first entry:

```yaml
- name: "Solutions Architect — Professional"
  level: "PRO"
  year: "2025"
  badge_id: "1912cfbe-dd94-43bb-8834-2237745e36da"
  image: "/images/certs/1912cfbe-dd94-43bb-8834-2237745e36da.png"
  verify_url: "https://www.credly.com/badges/1912cfbe-dd94-43bb-8834-2237745e36da/public_url"
```

Repeat for all five entries (DevOps `77233e3f-…`, Security `8f229d2a-…`, SA-Associate `13866ad2-…`, SysOps `2f52db24-…`), each using its own id in both `image` and `verify_url`.

- [ ] **Step 3: Replace the Certifications section in `layouts/page/home.html`**

```html
<!-- Certifications -->
<section>
  <div class="container">
    <div class="section-head">
      <div>
        <div class="section-kicker">credentials</div>
        <h2 class="section-title">AWS Certifications</h2>
      </div>
      <span class="section-link">5 / 5 active ✓</span>
    </div>
    <div class="cert-grid">
      {{ $levels := dict "PRO" "Professional" "SPE" "Specialty" "ASC" "Associate" }}
      {{ range .Site.Data.certs }}
      <a class="cert-card" href="{{ .verify_url | default (printf "https://www.credly.com/badges/%s/public_url" .badge_id) }}" target="_blank" rel="noopener">
        {{ if .image }}
        <img class="cert-badge" src="{{ .image }}" alt="{{ .name }} — AWS certification badge" loading="lazy" width="92" height="92">
        {{ else }}
        <div data-iframe-width="100" data-iframe-height="100" data-share-badge-id="{{ .badge_id }}" data-share-badge-host="https://www.credly.com"></div>
        {{ end }}
        <div class="cert-name">{{ .name }}</div>
        <div class="cert-level">{{ index $levels .level | default .level }}</div>
      </a>
      {{ end }}
    </div>
  </div>
  {{ if not (where .Site.Data.certs "image" "!=" nil) }}<script type="text/javascript" async src="//cdn.credly.com/assets/utilities/embed.js"></script>{{ end }}
</section>
```

> Note: the Credly embed script only loads if at least one cert lacks an `image` (i.e. a fallback is in play). If all five resolved, the page ships zero third-party JS for certs.

- [ ] **Step 4: Update the verified-badges block in `layouts/page/about.html`**

Replace the `<h2>Verified Badges</h2>` block + its `certs-grid` loop with the same image-first pattern:

```html
    <h2>Verified Badges</h2>
    <div class="cert-grid" style="margin-top:1.5rem;">
      {{ $levels := dict "PRO" "Professional" "SPE" "Specialty" "ASC" "Associate" }}
      {{ range .Site.Data.certs }}
      <a class="cert-card" href="{{ .verify_url | default (printf "https://www.credly.com/badges/%s/public_url" .badge_id) }}" target="_blank" rel="noopener">
        {{ if .image }}
        <img class="cert-badge" src="{{ .image }}" alt="{{ .name }} — AWS certification badge" loading="lazy" width="92" height="92">
        {{ else }}
        <div data-iframe-width="100" data-iframe-height="100" data-share-badge-id="{{ .badge_id }}" data-share-badge-host="https://www.credly.com"></div>
        {{ end }}
        <div class="cert-name">{{ .name }}</div>
        <div class="cert-level">{{ index $levels .level | default .level }}</div>
      </a>
      {{ end }}
    </div>
```

Keep the existing `embed.js` `<script>` at the bottom of `about.html` only if any cert may use the fallback; otherwise it's harmless to leave. (Leave it.)

- [ ] **Step 5: Build + visual check**

Run: `hugo --gc --minify` → exit 0.
On `/` and `/about/`: five cert cards show the real AWS badge images, hover-lift, with name + level caption, each linking to its Credly verification page. Both themes; the badges sit centred and evenly. At ≤880px the grid reflows.

- [ ] **Step 6: Commit**

```bash
git add data/certs.yaml static/images/certs layouts/page/home.html layouts/page/about.html
git commit -m "feat(certs): real Credly badge images in themed rack (iframe fallback)"
```

---

## Task 7: Blog — editorial list (home latest + blog list page)

**Files:**
- Modify: `layouts/page/home.html` (latest-posts section)
- Overwrite: `layouts/blog/list.html`

- [ ] **Step 1: Replace the Latest Posts section in `layouts/page/home.html`**

```html
<!-- Latest Posts -->
<section>
  <div class="container">
    <div class="section-head">
      <div>
        <div class="section-kicker">writing</div>
        <h2 class="section-title">Latest Posts</h2>
      </div>
      <a href="/blog/" class="section-link">all posts →</a>
    </div>
    <div class="blog-list">
      {{ range first 3 (where .Site.RegularPages "Section" "blog") }}
      <div class="blog-row">
        <span class="blog-date">{{ .Date.Format "2 Jan 2006" }}</span>
        <div class="blog-main">
          <h3><a href="{{ .Permalink }}">{{ .Title }}</a></h3>
          <p>{{ .Summary }}</p>
          {{ with .Params.tags }}<div>{{ range . }}<a class="blog-tag" href="/tags/{{ . | urlize }}/">{{ . }}</a>{{ end }}</div>{{ end }}
        </div>
      </div>
      {{ end }}
    </div>
  </div>
</section>
```

- [ ] **Step 2: Overwrite `layouts/blog/list.html`**

```html
{{ define "main" }}
<section style="padding-top:130px;">
  <div class="container">
    <div class="section-head">
      <div>
        <div class="section-kicker">writing</div>
        <h2 class="section-title">Blog</h2>
      </div>
    </div>
    <p class="section-desc" style="margin-bottom:18px;">Thoughts on cloud architecture, automation, and lessons learned in production.</p>
    <div class="blog-list">
      {{ range .Pages.ByDate.Reverse }}
      <div class="blog-row">
        <span class="blog-date">{{ .Date.Format "2 Jan 2006" }}</span>
        <div class="blog-main">
          <h3><a href="{{ .Permalink }}">{{ .Title }}</a></h3>
          <p>{{ .Summary }}</p>
          {{ with .Params.tags }}<div>{{ range . }}<a class="blog-tag" href="/tags/{{ . | urlize }}/">{{ . }}</a>{{ end }}</div>{{ end }}
        </div>
      </div>
      {{ end }}
    </div>
  </div>
</section>
{{ end }}
```

- [ ] **Step 3: Build + visual check**

Run: `hugo --gc --minify` → exit 0.
On `/` and `/blog/`: posts render as editorial rows — mono date column, title (teal on hover), 1–2 line summary, tag pills, hairline dividers, row background + indent on hover. Both themes; ≤780px stacks the date above the title.

- [ ] **Step 4: Commit**

```bash
git add layouts/page/home.html layouts/blog/list.html
git commit -m "feat(blog): editorial list rows on home + blog page"
```

---

## Task 8: Post pages — project single + blog single

**Files:**
- Overwrite: `layouts/projects/single.html`
- Overwrite: `layouts/blog/single.html`

> **Constraint:** Do NOT touch `.Content` / the markdown body — only the surrounding header markup and (via Task 2 CSS) the `.post-content` typography.

- [ ] **Step 1: Overwrite `layouts/projects/single.html`**

```html
{{ define "main" }}
<article>
  <header class="post-header">
    <div class="container">
      {{ with .Params.tag }}<span class="post-date">{{ . }}</span>{{ end }}
      <h1>{{ .Title }}</h1>
      {{ with .Params.summary }}<p class="post-summary">{{ . }}</p>{{ end }}
      {{ with .Params.tech }}
      <div class="stack-row" style="margin-top:1.4rem;">
        {{ range . }}<span>{{ . }}</span>{{ end }}
      </div>
      {{ end }}
    </div>
  </header>
  <div class="post-content">
    {{ .Content }}
  </div>
</article>
{{ end }}
```

- [ ] **Step 2: Overwrite `layouts/blog/single.html`**

```html
{{ define "main" }}
<article>
  <header class="post-header">
    <div class="container">
      <span class="post-date">{{ .Date.Format "2 January 2006" }}</span>
      <h1>{{ .Title }}</h1>
      {{ with .Params.summary }}<p class="post-summary">{{ . }}</p>{{ end }}
      {{ with .Params.tags }}
      <div style="margin-top:1.2rem;">{{ range . }}<a class="blog-tag" href="/tags/{{ . | urlize }}/">{{ . }}</a>{{ end }}</div>
      {{ end }}
    </div>
  </header>
  <div class="post-content">
    {{ .Content }}
  </div>
</article>
{{ end }}
```

- [ ] **Step 3: Build + visual check**

Run: `hugo --gc --minify` → exit 0.
Open a project page (e.g. `/projects/fsx-migration/`) and a blog post: header has a teal mono label, large tight headline, summary, and tech/tag chips; the body content is comfortably readable at 1.02rem with styled headings, code blocks, lists, and blockquotes in both themes. Confirm the case-study body text (including the "X TB" placeholders) is unchanged. Check ≤780px padding.

- [ ] **Step 4: Commit**

```bash
git add layouts/projects/single.html layouts/blog/single.html
git commit -m "feat(posts): restyle project + blog single headers and typography"
```

---

## Task 9: Contact, footer, page index, about header

**Files:**
- Modify: `layouts/page/home.html` (contact section + add `.page-index`)
- Overwrite: `layouts/partials/footer.html`
- Modify: `layouts/page/about.html` (header only)

- [ ] **Step 1: Replace the Contact section in `layouts/page/home.html`**

```html
<!-- Contact -->
<section id="contact" class="contact-section">
  <div class="container">
    <div class="section-kicker" style="justify-content:center;">say hello</div>
    <h2 class="section-title">Get in touch</h2>
    <p class="section-desc" style="margin:.6rem auto 0;">
      Open to new opportunities and happy to chat about cloud architecture,
      infrastructure automation, or anything AWS.
    </p>
    <form class="contact-form" id="contact-form" action="{{ .Site.Params.contact.formspree }}" method="POST">
      <input type="text" name="name" placeholder="Your name" required>
      <input type="email" name="email" placeholder="Your email" required>
      <textarea name="message" placeholder="Your message" required></textarea>
      <input type="text" name="_gotcha" class="honeypot" tabindex="-1" autocomplete="off" aria-hidden="true">
      <button type="submit" class="btn btn-1">Send message →</button>
    </form>
    <div id="form-status" class="form-status" style="display:none;"></div>
    <p class="contact-or">or reach me directly</p>
    <div class="contact-direct">
      {{ with .Site.Params.email }}<a href="mailto:{{ . }}" class="btn btn-2">✉ {{ . }}</a>{{ end }}
      {{ with .Site.Params.linkedin }}<a href="{{ . }}" class="btn btn-2" target="_blank" rel="noopener">in LinkedIn</a>{{ end }}
    </div>
  </div>
</section>
<div class="page-index">01 — HOME</div>
```

> The `.section-kicker` centring inline-style overrides the default left tick layout for this one centred section. The `::before` tick still renders; that's intentional.

- [ ] **Step 2: Overwrite `layouts/partials/footer.html`**

```html
<footer class="site-footer">
  <div class="social-links">
    {{ with .Site.Params.github }}<a href="{{ . }}" target="_blank" rel="noopener">GitHub</a>{{ end }}
    {{ with .Site.Params.linkedin }}<a href="{{ . }}" target="_blank" rel="noopener">LinkedIn</a>{{ end }}
    {{ with .Site.Params.email }}<a href="mailto:{{ . }}">Email</a>{{ end }}
  </div>
  <p>&copy; {{ now.Year }} Tom Williams · Built with Hugo</p>
</footer>
```

- [ ] **Step 3: Restyle the about-page header in `layouts/page/about.html`**

Wrap the title in the post-header pattern so the About page matches. Change the opening of the article from:

```html
<article>
  <header class="post-header">
    <div class="container">
      <h1>{{ .Title }}</h1>
    </div>
  </header>
  <div class="post-content">
```

to:

```html
<article>
  <header class="post-header">
    <div class="container">
      <span class="post-date">about me</span>
      <h1>{{ .Title }}</h1>
      <p class="post-summary">Senior Cloud Engineer · AWS 5× certified · United Kingdom</p>
    </div>
  </header>
  <div class="post-content">
```

(Leave the rest of `about.html` — the `.Content` and the Task 6 cert grid — intact.)

- [ ] **Step 4: Build + visual check**

Run: `hugo --gc --minify` → exit 0.
On `/#contact`: centred form with themed inputs (teal focus ring), `btn-1` submit, the "or reach me directly" mailto + LinkedIn fallbacks as `btn-2`. Submit a test message — the AJAX success/error status still shows (form JS untouched). Footer reads as mono social links + copyright. `01 — HOME` index sits bottom-left. `/about/` header matches the post style. Both themes.

- [ ] **Step 5: Commit**

```bash
git add layouts/page/home.html layouts/partials/footer.html layouts/page/about.html
git commit -m "feat(contact/footer/about): restyle to design system + page index"
```

---

## Task 10: Full verification + cleanup

**Files:**
- Modify: `.gitignore` (commit the `.superpowers/` ignore added during brainstorm, if not already committed)

- [ ] **Step 1: Commit the gitignore change if pending**

Run: `git status --short`
If `.gitignore` shows as modified (it gained a `.superpowers/` ignore line during brainstorming):

```bash
git add .gitignore
git commit -m "chore: gitignore .superpowers brainstorm dir"
```

- [ ] **Step 2: Clean production build**

Run: `hugo --gc --minify`
Expected: exit 0, no WARN/ERROR about templates or missing layouts. Note the page count is unchanged from before the redesign (40 pages).

- [ ] **Step 3: Grep for regressions of retired tokens/fonts**

Run:
```bash
grep -rn "Outfit\|JetBrains\|0ff0b3\|--bg-deep\|--accent\b" layouts static/css || echo "clean"
```
Expected: `clean` (no references to the old fonts, old mint accent, or old variable names remain).

- [ ] **Step 4: Manual pass with `hugo server`**

Run `hugo server` and verify each, in BOTH themes and at desktop + 375px mobile:
- `/` — hero, spec card, about, projects, certs (real badges), blog, contact, footer
- `/projects/` and one `/projects/<slug>/`
- `/blog/` and one `/blog/<slug>/`
- `/about/`
- Toggle persists across navigation and reload; first-visit honours OS theme (test via devtools "Emulate prefers-color-scheme").
- No layout shift / FOUC of the wrong theme on load.

- [ ] **Step 5: Confirm preserved functionality (prior-pass work)**

View source of `/` and confirm still present: `seo.html` OG/Twitter/JSON-LD tags, GA4 gtag snippet, favicon links, availability + location, Formspree action + honeypot, `resumeReady` gating (CV button hidden while `resumeReady = false`).

- [ ] **Step 6: Final commit (if any stray changes)**

```bash
git status --short
# if anything outstanding:
git add -A && git commit -m "chore: portfolio visual uplift final tidy"
```

---

## Self-review (completed by plan author)

**Spec coverage:** direction & motifs → Tasks 2–9; dual theme + no-flash + persistence → Tasks 1, 3; tokens → Task 2; typography → Task 2; hero + spec card → Task 4; projects (home + list + single) → Tasks 5, 8; certs as images → Task 6; blog (home + list + single) → Tasks 7, 8; contact/footer/about → Task 9; drop noise / light grain → Task 2; preserved prior-pass work → Tasks 1, 6, 9 + verified in Task 10. All §4 subsections mapped.

**Placeholder scan:** none — every code step contains complete copy-pasteable code; cert image URLs are generated by the Step-1 script, not hand-waved.

**Type/class consistency:** class names verified against the contract block and the Task 2 stylesheet (`.proj-card`, `.spec`, `.cert-card`, `.blog-row`, `.meta-line`, `.hero .inner/.hero-main`, `theme-toggle button[data-theme-set]`). The toggle JS in Task 1 reads `data-theme-set` which matches the Task 3 markup.
