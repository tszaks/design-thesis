# How Apple Builds the Web

Apple's websites are not Swift. apple.com, icloud.com, music.apple.com, and developer.apple.com run on Svelte, React, Vue, Jekyll, and a bespoke in-house component system — and they still read unmistakably as Apple. This document explains both halves: what actually runs each property, and the concrete, copy-able techniques that manufacture the native feel in plain HTML and CSS.

The thesis in one line: **the native feel is a design-system property, not a language property.** Five ingredients do almost all of the work — the SF optical-size type scale, the `saturate(180%) blur(20px)` glass material with a designed fallback, iOS deceleration beziers, rAF-driven canvas scroll scrubbing that degrades to stills, and a fanatical progressive-enhancement floor.

**Method and honesty.** Compiled 2026-08-26 from live view-source probes of ten Apple properties (HTML, shipped CSS, and JS bundles), official open-source repos, license texts, job postings, and established teardowns. Claims are labeled **[CONFIRMED]** (verified directly), **[REPORTED]** (credible secondary sources), or **[SPECULATIVE]** (informed inference). Backwards compatibility is a first-class dimension: every technique carries its browser floor, its fallback, and Apple's own observed posture.

---

# Part 1 · The stacks (who runs what)

| Property | Stack | Evidence class |
|---|---|---|
| apple.com (marketing) | In-house "AC" component system, server-rendered HTML, versioned `.built.css/.built.js` bundles, declarative `data-component-list` bootstrapping | [CONFIRMED] live probe |
| icloud.com | **React** (+ TypeScript), single ~3.9 MB webpack bundle, with living SproutCore-era document engines under the iWork apps | [CONFIRMED] bundle inspection + job postings |
| music.apple.com, tv.apple.com, podcasts.apple.com, apps.apple.com | **Svelte, server-side rendered** (scoped-class hashes in the delivered HTML; Vite markers; served by Apple Media Products' `daiquiri` servers) | [CONFIRMED] live probes |
| developer.apple.com/documentation | **Vue 2.7** — the renderer is open source: `github.com/swiftlang/swift-docc-render` (Vue CLI 5, vue-router 3) | [CONFIRMED] repo + live site |
| account.apple.com (Apple Account) | **React + Redux** (`react-redux-kit-*.js` with subresource integrity) | [CONFIRMED] live probe |
| maps.apple.com | Thin shell over **MapKit JS** (Apple's public map framework); surrounding UI framework unidentified | [CONFIRMED] for MapKit; rest [SPECULATIVE] |
| swift.org | **Jekyll 4.3** static site (open repo). Even the Swift language's website is not Swift | [CONFIRMED] repo Gemfile |

**apple.com's in-house system, in detail [CONFIRMED]:** assets load from versioned component paths (`/ac/globalfooter/8/...`, `/ac/localnav/9/...`, `/ac/ac-films/7.5.1/...`); the 2025-refresh global nav ships as a shared UMD element service (`/api-www/global-elements/global-header/v1/`); page bodies are per-campaign bundles (`/v/iphone-17-pro/h/built/...`); behavior attaches declaratively (`data-component-list="ScrollGallery WillChange"`); analytics is in-house (`data-analytics-*`, 140 instances on the homepage). These are classes and data attributes on plain elements — progressive enhancement, not Custom Elements.

**icloud.com's lineage:** SproutCore 2008–2013 (Apple built the framework; 9to5Mac verified it in shipped code) → the Ember era (~2015–2019; SproutCore 2.0 literally became Ember) [REPORTED] → the React rewrite landing ~March 2022 [CONFIRMED by bundle contents + framework trackers + current job postings asking for "TypeScript and React... experience with SproutCore"].

**The org pattern [CONFIRMED via job postings]:** there is no single "Apple web framework." Framework choice is per-organization: media storefronts = Svelte, iCloud = React, developer docs = Vue, marketing = in-house AC, identity = React/Redux. Apple's web postings ask for TypeScript, "React, Svelte, Angular, VueJS", Vite/webpack, Playwright, WCAG — never Swift for web UI.

---

# Part 2 · The five ingredients (copy these)

## 2.1 Typography — the biggest single ingredient

**What Apple ships [CONFIRMED, extracted from live CSS]:** apple.com does not rely on the system font stack. It self-hosts **SF Pro as a variable WOFF2** through a font service (`/wss/fonts?families=SF+Pro,v4`), Referer-gated so the CSS returns empty off-domain. Declared stacks: `SF Pro Display, SF Pro Icons, Helvetica Neue, Helvetica, Arial, sans-serif` (display) and the `SF Pro Text` twin, with locale-aware regional cuts (PingFang, Hiragino) prepended per market.

**The licensing catch [CONFIRMED from the license text]:** SF Pro's license permits mock-ups for Apple-platform software only and forbids serving it as web content. **Only Apple may serve SF Pro on the web.** The legal reproduction for everyone else:

```css
font-family: system-ui, -apple-system, BlinkMacSystemFont,
             "Segoe UI", Roboto, Helvetica, Arial, sans-serif;
```
Floor: `-apple-system` Safari 9; `system-ui` Safari 11 / Chrome 56 / Firefox 92. You get real SF on Apple hardware and the best native face elsewhere. Need SF-everywhere? Use Inter (metrically close, legally clean).

**The type scale [CONFIRMED, measured from shipped CSS] — copy these exact triplets:**

| Role | Size / line-height / weight / tracking |
|---|---|
| Hero display | ~80px / 1.05 / 600–700 / −0.015em |
| Headline lg | 56px / 1.0714 / 600 / −0.005em |
| Headline | 48px / 1.0835 / 600 / −0.002em |
| Headline md | 40px / 1.1 / 600 / 0 |
| Callout | 32px / 1.125 / 600 / +0.002em |
| Callout sm | 28px / 1.1429 / 600 / +0.007em |
| Subhead | 24px / 1.1667 / 600 / +0.009em |
| Intro copy | 21px / 1.19 / 400–500 / +0.011em |
| Small copy | 19px / 1.21 / 400 / +0.012em |
| **Body** | **17px / 1.4706 / 400 / −0.022em** |

The signature habits: body is **17px** (the iOS default, not the web's 16); headings are **weight 600, never bold**; headline line-heights are extremely tight (1.05–1.125); and tracking follows SF's optical-size curve — negative at display sizes, slightly positive in the 19–32px mid-band, −0.022em on body. That tracking curve is most of why Apple type looks like Apple type.

## 2.2 The glass nav — exact material, exact fallback

**[CONFIRMED — extracted verbatim from apple.com's `globalheader.css`]:**

```css
/* Base = the FALLBACK: near-opaque solid in the same color */
#globalnav {
  --globalnav-background: rgba(250, 250, 252, .92);   /* dark: rgba(22, 22, 23, .88) */
  --globalnav-backdrop-filter: none;
  background: var(--globalnav-background);
  -webkit-backdrop-filter: var(--globalnav-backdrop-filter);
  backdrop-filter: var(--globalnav-backdrop-filter);
}

/* Enhancement, only where supported */
@supports (backdrop-filter: initial) {
  #globalnav.globalnav-scrim {
    --globalnav-backdrop-filter: saturate(180%) blur(20px);
    --globalnav-background: rgba(250, 250, 252, .8);   /* dark: rgba(22, 22, 23, .8) */
  }
}
```

The canonical numbers, unchanged for years: **`saturate(180%) blur(20px)` over `rgba(250,250,252,0.8)`** light / `rgba(22,22,23,0.8)` dark. The `saturate(180%)` is the secret — it makes colors behind the bar intensify (NSVisualEffectView vibrancy), which reads as material rather than mere blur. Separation is a 1px translucent hairline, never a heavy border. Nav height sits at the 44–48px iOS touch metric.

**The fallback lesson [CONFIRMED]:** the no-support base isn't "no background" — it's a MORE OPAQUE version of the same color (.8 → .92), so the design survives with zero layout shift. Floors: `backdrop-filter` Chrome 76, Firefox 103, Safari 9 with `-webkit-` prefix, unprefixed only Safari 18. Notably, Apple's current CSS has dropped the `-webkit-` prefix entirely — Safari ≤17 now gets the solid fallback. That's Apple's own floor speaking: enhance for evergreen, ship the designed solid to everyone else. If your audience includes older iOS, keep both prefixed and unprefixed.

## 2.3 Color and materials

**[CONFIRMED from shipped CSS]:** light surfaces are pure `#fff` and **`#f5f5f7`** (the Apple section gray); body text `#1d1d1f` (near-black, never #000); secondary `#6e6e73`; link blue `#0066cc`, button blue `#0071e3` (hover `#0077ed`); dark bands `#000` and `#161617`. Buttons are pills (the in-joke `border-radius: 980px`), 17px text — a direct iOS-button translation.

**Dark mode is split by property type [CONFIRMED]:** marketing pages are art-directed (dark bands are authored per section — the homepage CSS contains zero `prefers-color-scheme` queries), while the chrome and the app-like properties (nav, music, docs, iCloud) respond to the media query by flipping custom-property token sets. Copy the mechanism: express both palettes as CSS custom properties and let one query flip the token set. Floor: Safari 12.1 / Chrome 76 / Firefox 67.

## 2.4 Motion

**The beziers [CONFIRMED, extracted from shipped CSS]:**
```css
cubic-bezier(.4, 0, .6, 1)        /* the workhorse ease-in-out (78 occurrences in the nav alone) */
cubic-bezier(.25, .1, .3, 1)      /* soft ease-out, nav flyouts */
cubic-bezier(0, 0, 0.5, 1)        /* pure deceleration — the iOS scroll settle */
cubic-bezier(0.33, 1, 0.68, 1)    /* easeOutCubic, reveals */
cubic-bezier(0.3, 2, 0.5, 1)      /* springy overshoot (y > 1 ≈ SwiftUI spring) for pop-in accents */
```

**Scroll heroes: canvas image-sequence scrubbing, JS-driven [CONFIRMED].** The iPhone 17 Pro page's bundle has 56 `requestAnimationFrame` references, 14 `canvas`, and zero CSS scroll-timeline references. The mechanism: pre-rendered frame sequences (148 frames on the 2019 AirPods Pro hero) drawn to a sticky `<canvas>`, frame index = f(scroll progress), inside rAF. Refinements worth stealing: **checkpoint-priority preloading** (first frame, then last, then 50%, then 25/75%, densifying — a degraded scrub works early) and static-image replacement on mobile, slow connections, and noscript.

**Why Apple doesn't use CSS Scroll-Driven Animations [CONFIRMED absence]:** the floor is Chrome 115 / Safari 26 / Firefox flagged — Apple ships to a wider floor than its own newest Safari. For new work: author `animation-timeline: view()` inside `@supports (animation-timeline: view())` and keep an IntersectionObserver fallback (IO floor: Safari 12.2 / Chrome 51 — effectively universal; Apple's own docs app bundles the IO polyfill).

**Reduced motion is respected [CONFIRMED]:** 5 `prefers-reduced-motion` CSS blocks + 3 `matchMedia` checks on the iPhone page; sequences and autoplaying films replaced with stills. Copy the direction of the gate: enable effects under `(prefers-reduced-motion: no-preference)` so reduced motion is the default path, and pick a representative mid-sequence frame as the still. Floor: Safari 10.1 / Chrome 74 / Firefox 63.

**View Transitions: not used anywhere on Apple's properties [CONFIRMED absence].** If you want them (Chrome 111 / Safari 18 / Firefox 144), they're pure progressive enhancement — feature-detect, fall back to instant navigation.

## 2.5 Layout

- The classic fixed **980px** grid (2007–2014) survives as a max content measure and the pill-radius in-joke. Today [CONFIRMED]: fluid grids with the hard breakpoint trio at **~734px and ~1068px** (small / medium / large), section padding in the 80–150px band, one idea per viewport (headline, one sentence, media), copy measure capped near 600px.
- **The sticky local nav (`ac-localnav`) [CONFIRMED]:** every product page carries a 52px product-scoped bar — product name left, Buy pill right, the same glass recipe — `position: sticky; top: 0`. This one component is the web translation of iOS's large-title-to-inline collapse and contributes disproportionately to the native feel. Floor: universal.
- `100dvh` is adopted behind `@supports (height: 100dvh)` guards with `vh` math as the base [CONFIRMED]. Floor: Safari 15.4 / Chrome 108 / Firefox 101.
- Conservative elsewhere [CONFIRMED absence in probed files]: no container queries, no `:has()`, no `text-wrap: balance` in the probed CSS — fixed breakpoints and media queries instead.

## 2.6 Imagery and the progressive-enhancement floor

**[CONFIRMED by markup inspection]:** everything is 2x-first (`_small/_medium/_large` × `_2x` srcset naming), art-directed `<picture>` swaps at 734/1068, hero videos short, muted, `playsinline`, with poster frames and explicit pause buttons.

**The number that defines Apple's discipline: 49 `<noscript>` fallbacks on the homepage, 149 on the iPhone page.** Every JS-enhanced gallery has a static image inside noscript; apple.com marketing pages are fully readable with JavaScript disabled. The posture splits by property type [CONFIRMED]: marketing = full noscript floor; music.apple.com = SSR content readable without JS, playback requires it; icloud.com = app shell, JS hard-required, zero noscript. Match your posture to your property type: content pages degrade, apps may require.

---

# Part 3 · The backwards-compatibility doctrine

Apple's observed rule, everywhere in the shipped code: **enhance inside `@supports`, never punish the floor, and make the fallback a denser version of the same design.**

| Technique | Floor (Safari / Chrome / Firefox) | Apple's own fallback | Posture |
|---|---|---|---|
| `backdrop-filter` glass | 9 `-webkit-` → 18 unprefixed / 76 / 103 | `@supports` gate; solid `rgba(...,.92)` base | Progressive enhance [CONFIRMED] |
| `100dvh` | 15.4 / 108 / 101 | `@supports (height:100dvh)`; vh math base | Progressive enhance [CONFIRMED] |
| Scroll-linked heroes | n/a (JS) | rAF + canvas (not CSS timelines: 26/115/flag); checkpoint preloading; static image on mobile/noscript | Wide-floor JS, static degrade [CONFIRMED] |
| `prefers-reduced-motion` | 10.1 / 74 / 63 | stills instead of sequences | Respected [CONFIRMED] |
| `prefers-color-scheme` | 12.1 / 76 / 67 | marketing art-directed; chrome/apps token-flip | Mixed [CONFIRMED] |
| View Transitions | 18 / 111 / 144 | not adopted | [CONFIRMED absence] |
| Container queries / `:has()` / `text-wrap: balance` | 16 / 15.4 / 17.5 (Safari) | not found in probed CSS; breakpoints instead | Conservative [CONFIRMED in probed files] |
| JavaScript disabled | — | marketing: 49–149 noscript fallbacks per page; iCloud: hard-requires; Music: SSR reads, playback needs JS | Split by property type [CONFIRMED] |

---

# Part 4 · Why not Swift

**Confirmed: zero Swift in any probed Apple web front end.** The options that exist and why they don't fit: **SwiftWasm + Tokamak** (SwiftUI-like API over WebAssembly) is incomplete, maintainer-starved, multi-megabyte, and has no SSR story [CONFIRMED from repo]. **Ignite** (Paul Hudson's Swift static-site generator) is good for developer blogs, irrelevant at Apple's interactivity scale [CONFIRMED from repo]. Server-side Swift HTML (Vapor + Elementary) gives none of the client interactivity these properties need. Structurally [SPECULATIVE but well-supported]: no Swift toolchain for hydration/SSR/code-splitting, wasm payload cost vs. Svelte's compile-to-vanilla-JS, hiring pools, and web orgs that predate Swift itself (SproutCore, 2007). Even swift.org is Jekyll.

The takeaway for anyone building "Swift-looking" web apps: stop looking for a language bridge. Apple doesn't use one. Reimplement the design system — the type scale, the material, the beziers, the discipline — in whatever stack you already ship.

---

# Sources

**Live probes (2026-08-26):** apple.com (+ `/api-www/global-elements/global-header/v1/assets/globalheader.css` and `.umd.js`, `/v/home/a/styles/main.built.css`, the iPhone 17 Pro page CSS/JS, `/wss/fonts` service) · icloud.com (+ `main.js` bundle) · music.apple.com/us/browse · tv.apple.com · podcasts.apple.com · apps.apple.com · account.apple.com · developer.apple.com/documentation · maps.apple.com · swift.org

**Repos and official:** github.com/swiftlang/swift-docc-render (Vue 2.7, Vue CLI 5) · github.com/swiftlang/swift-org-website (Jekyll 4.3) · developer.apple.com/fonts (SF Pro license text) · Apple Newsroom, 2025-06-09 (Liquid Glass) · WWDC25 sessions 219/323/356 · github.com/TokamakUI/Tokamak · github.com/twostraws/Ignite

**History and teardowns:** 9to5Mac (iCloud = SproutCore, 2011) · Yehuda Katz, "Announcing Amber.js" (SproutCore 2.0 → Ember) · Hacker News 20891216 (2019 iCloud = Ember) · poweredby.keywordseverywhere (icloud.com framework, Mar 2022) · Ars Technica (Apple Music web GA, Apr 2020) · okupter.com and madewithsvelte (Svelte at Apple) · the Nov 2025 apps.apple.com source-map analyses (Svelte confirmed) · CSS-Tricks, "Let's Make One of Those Fancy Scrolling Animations Used on Apple Product Pages" · geyer.dev CSS image-sequence analysis · programmersought AirPods Pro teardown

**Stack evidence from Apple job postings:** iCloud front-end (TypeScript + "React, Svelte, Angular, VueJS", WebSockets/WebRTC/WebGL, XPC), iWork for iCloud (TypeScript + React + SproutCore).

**Browser floors:** caniuse.com and MDN entries for every floor cited.

## One more finding worth knowing

**Liquid Glass has not reached Apple's web [CONFIRMED as of 2026-08-26].** The 2025 nav refresh brought rounded flyouts, but the material is still classic `backdrop-filter` glass — no lensing, no refraction. The honest web approximation ceiling: blur + saturate, an inset top-edge highlight (`box-shadow: inset 0 1px 0 rgba(255,255,255,.35)`), a 1px translucent border, and large radii. SVG displacement-map refraction works in Chromium only — decorative at most, with plain glass as the real material.

---

*Companion documents in this repo: [README.md](README.md) (the design thesis), [LLM.md](LLM.md) (the thesis for machines), [APPLE_NATIVE.md](APPLE_NATIVE.md) (the SwiftUI component catalog). For web work, paste LLM.md and this file together; for iOS work, LLM.md and APPLE_NATIVE.md.*
