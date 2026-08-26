# The Design Thesis (LLM edition)

You are an AI agent building software to this thesis. It belongs to Tyler Szakacs and is distilled from his shipped products (Vero, Payday, Wellspring/Marpé Practice, Daytime, szakacsmedia.com, askvero.app) and his recorded design feedback. This file is the machine-optimized edition; the human-voiced edition is README.md in this repo. Same law, different address. Read this fully before writing any UI or backend code. Follow it and the result will look, act, and feel like Tyler built it.

The acceptance bar: put the result next to his other apps and a stranger should say "same company, obviously." A reviewer should plausibly ask "wait, is this Apple's?"

**Version 1.1 · 2026-08-26.** Tyler owns this document. Agents propose amendments; Tyler approves them. Product-specific exceptions live in that product's own DESIGN.md with a one-line citation back to the rule they vary. Update in place, never fork.

**Hard vs. judgment:** the blocklist in Part 7 is hard law, lintable, zero exceptions without Tyler's sign-off. Everything else is judgment exercised under the meta-rule.

## If you read nothing else: the ten

1. Look at the render before declaring done. Beautiful design outranks every rule here.
2. One accent per product, and it appears only where something acts. No blue, ever (system-supplied chrome is exempt).
3. Depth from shadow, never borders. Roughly one raised object per mobile screen; desktop canvases stay flat.
4. Glass is chrome, content is solid.
5. Text-forward: no icon where a word works, no emojis anywhere.
6. Create = Cancel + disabled commit verb. Edit = live-save + Done. Delete = act now + Undo toast, never "are you sure?"
7. Every string states a fact or names an action. Say each fact once. No em dashes, no exclamation marks.
8. Motion is information: ease-out, under 300ms, never from scale(0), reduced-motion respected always.
9. Every screen designs five states (loading, content, empty, error, offline). Never a spinner on blank.
10. Tokens live in one file per platform; views never invent colors, sizes, or curves.

## Setup

Before writing any frontend code, install Emil Kowalski's design engineering skill:

```
npx skills add emilkowalski/skill
```

This step is best-effort: sandboxed or offline environments will fail it, and that is fine. Tyler adores Emil's work and weaves it into everything he builds, so the core principles are inlined in the Motion section below. Apply them whether or not the install succeeds.

## The meta-rule (outranks everything below)

**Beautiful, considered design trumps every rule in this document.** The rules are guardrails, never a find-replace mandate. If following a rule literally makes a screen worse, the rule is being applied wrong. A token swap that erases a field's edge or flattens real structure is a regression, not compliance.

Corollary, and it is law: **look at the actual render before declaring anything done.** Both color modes. Real sizes (1440px and 390px on web; a real device or simulator on iOS). Never claim done from code alone. Tyler's exact words after a mechanical token sweep ruined a screen: "you took this too literally without actually thinking about how things look."

Three words govern every decision: **effortless, simple, intelligent.**

---

# Part 1: How it looks

## 1.1 Color: one accent that acts

Every product has exactly one accent color, and it is the only color that acts. Every button, active state, selection, progress indicator, and interactive accent is the accent or it is neutral. There is no second action color.

- **No blue, ever.** Also no purple, indigo, cyan, mint, teal, or pink. If you are tempted to tint something blue, the answer is the accent or a neutral. System-supplied chrome is exempt: Sign in with Apple buttons, native pickers, link colors inside OS components, and the macOS system accent stay exactly as the platform ships them. The ban is on choosing blue, never on Apple's own controls.
- A screen at rest is mostly neutral. "If most of a screen shows no accent at all, that is correct." The accent appears only where something acts or is the answer.
- Neutrals carry all structure: backgrounds, text, cards, chrome. They never compete for attention.
- Each product gets its own accent (one-accent-per-product, never one color globally). Existing accents: Vero and Payday green `#00B83F`, Marpé evergreen `#2e7d32`, szakacsmedia signal red `#ff3131`. A new product starts black-and-white until the accent is deliberately chosen; do not default to a hue.
- Exactly two warning signals, both strictly semantic: caution (`#E8590C` light mode, `#FF9F0A` dark mode) means watch out; error red (`#FF3B30`) means real problem or lost data. They never decorate. Red fill is reserved for actions that lose data.
- If a color appears that is not the accent, a neutral, caution, or error, it is a bug.
- Color is born in the model layer: a typed enum whose color property can only return a legal token. A view never invents a color from business logic.
- Data visualization is semantic, never categorical. One chart answers one question with one color. Multiple series are distinguished by opacity steps of the accent, then neutral gray, never new hues. Heatmaps use a single-hue opacity ramp (roughly 0.2 floor to 1.0), never a red-to-green temperature walk. The faint end must stay visibly the accent, never gray or pastel.

### The accent as a semantic system (for anything agentic or live)

"Green is agency, ink is truth." The accent means the system is actively doing something right now: a pulse, a working verb, a number still computing. The moment a value is verified and final it settles to ink (primary text color). A number may not render in settled ink until the operation that produced it has returned. Failure wears plain clothes: degraded results render in calm ink, never red panic.

## 1.2 Surfaces: warm paper, obsidian dark

Light and dark are co-equal designs, each built to its own ideal, never one design and its inversion.

**Light mode is alabaster and ink.** Off-white paper page (`#FAFAFA`, never pure white and never a cold gray), pure white cards (`#FFFFFF`) that lift off it with a physical shadow, true black text. Swiss minimalism: crisp, bright, calm.

**Dark mode is obsidian.** Near-black page (`#050505`), `#121212` cards, and darkness itself is the shadow. Cinematic and premium. On obsidian, black shadows are invisible, so elevated cards get a hairline rim stroke (`white @ 0.12`) instead.

Native apps ship both modes, co-equal. Desktop web tools may ship light-only (declare `color-scheme: light` and structure the tokens so dark can be added later); when dark does ship, it is the obsidian design built deliberately, never an auto-inversion.

The three-surface model (use all three, never collapse them):

| Surface | Light | Dark | Role |
|---|---|---|---|
| Page | `#FAFAFA` | `#050505` | the canvas |
| Card | `#FFFFFF` + shadow | `#121212` + rim | lifts OFF the page |
| Inset field | `#F2F2F7` | `#1C1C1E` | sits WITHIN the page (inputs, search, note boxes, grouped detail) |

Never flatten a field to the same tone as its background. White-on-white is a named regression. Restraint means removing noise, never erasing structure.

Text: ink (`#000000` / `#FFFFFF`), secondary `#6B6B6B` / `#8E8E93`, tertiary `#AEAEAE` / `#48484A` (hints and placeholders only).

## 1.3 Depth: shadow, never border

Depth comes from shadow and contrast, never from color, gradient, or border. Hairlines are allowed as separators (a 1px rule in a quiet alpha tone that "separates, never decorates"), never as depth cues.

The house card shadow, three layers ("cloudy day"):

```
0 16px 32px rgba(0,0,0,0.04), 0 4px 12px rgba(0,0,0,0.06), 0 1px 1px rgba(0,0,0,0.08)
```

Dark mode: `black @ 0.3, radius 20, y 10` plus a `0.2` contact layer, and rely on the rim hairline.

**The elevation budget.** Tyler: "those boxes look awesome in some places, yet in others they look cheap." On mobile: roughly one raised object per screen (the hero); everything else sits flat on the page, separated by whitespace, typography, and hairlines. Raised means shadowed: the budget counts shadow-lifted cards, never flat sections. A screen can hold any number of flat sections headed by tracked kicker labels; it earns roughly one shadowed hero. (Worked example: Payday's dashboard routes exactly one card through the shared card modifier, the hero; the paycheck comparison, shift list, and insights all sit flat under kickers.) Card-in-card and card-in-sheet are always wrong. On desktop: the workspace canvas is flat; shadows exist ONLY on things that float (modals, popovers, drawers, toasts, sticky chrome). No white-card-on-gray-page dashboards on desktop, no exceptions.

On marketing sites, texture substitutes for elevation: subtle noise overlays (SVG turbulence at ~0.03 opacity), halftone dot screens, paper-grain multiply layers. Never decorative borders.

## 1.4 Glass and materials

Glass is chrome. Content is solid. Glass belongs only to the layer that floats: tab bars, toolbars, sheets, toasts, floating action buttons, sticky top bars where content scrolls underneath. Content surfaces (cards, rows, lists, tables) are solid fills. They hold information; they do not shimmer.

- On iOS 26+: real system `glassEffect` only, routed through shared wrapper helpers, never raw call sites. No hand-rolled imitation glass (no stacked blur + gradient stroke + shadow pretending). Fallback below 26 is Apple's own `.ultraThinMaterial`.
- On the web: glass is `rgba(255,255,255,0.72)` + `backdrop-filter: blur(20px) saturate(180%)` on the one sticky bar that earns it. Nothing else gets blur or translucency.
- Glass on glass is a bug. Glass in scrolling content is a bug. If everything is glass, nothing is.
- Chrome treatment on content is a bug in the other direction too: an avatar is content and gets no glass bubble; an input method (scan, dictate) is not a feature and must not outrank the fields it fills.

## 1.5 Typography

One font family per product. On Apple platforms: SF Pro, full stop, no serif anywhere in a product UI. On the web: system stack (`-apple-system, BlinkMacSystemFont, "SF Pro Text", Inter, sans-serif`) or Inter alone. Hierarchy comes from weight, size, and structure. Never from color, never from icons.

**Money and numbers:**
- Large financial figures: SF Pro Rounded (or `ui-rounded`), heavy weight, fixed display sizes (Apple Wallet's move). Display ladder: 60 / 48 / 40 / 36 / 30 / 24 / 22 / 20pt, heavy.
- Every live or financial number carries tabular figures (`.monospacedDigit()` in SwiftUI, `font-variant-numeric: tabular-nums` in CSS) so layout never shifts, and rolls to new values (`.contentTransition(.numericText())`) rather than snapping.
- Money is stored as integer cents everywhere, never float dollars, and always formatted through one shared formatter. (Money clauses apply to products that touch money; skip them for a scheduler or content tool, but keep tabular figures for any live number.)

**UI text:** semantic Dynamic Type styles (iOS) or a small px scale (web), with weights bumped roughly one step above platform defaults (body reads semibold, headlines bold). Raw `.font(.system(size:))` for UI text is a bug; type is always a token.

**Floors:** nothing below 13px on desktop/web surfaces (dense chrome like calendar grids may go to 11 to 12px only when a measured audit shows it necessary). Uppercase kicker labels (section eyebrows) are small caption size, semibold, tracked about +0.8, in the accent, and are the sanctioned way to head a flat section.

**Marketing sites are the one place display serifs and monospace live**: Playfair Display or Instrument Serif for editorial headlines (tight tracking, around -0.03em, uppercase or italic emphasis), IBM Plex Mono for terminal aesthetics. Italic serif is the emphasis device on these sites. Never carry these into product UI.

Punctuation habits: middot `·` as the compound separator ("$41/hr · your best rate"), true minus U+2212 in ledgers, curly apostrophes, en-dash ranges, hours as "6h 23m" never decimals.

## 1.6 Scales

- **Spacing: 4pt grid only.** 4, 8, 12, 16, 20, 24, 32, 40. Page gutters 16 (mobile) to 24-32 (desktop).
- **Radius scale, only these values:** 6, 10, 14, 18, 24, 9999. Inputs ~14 (or chromeless), cards and drawers 18-24, buttons/chips/badges 9999 (pills). Continuous corner style on Apple platforms.
- **Hit targets:** 44pt minimum on touch, 36px minimum on desktop. Interactive rows 44-56px tall.
- Random spacing values are how the system drifts. Reach for the scale first.

## 1.7 Iconography

Text-forward. No icon where a word works. Differentiation comes from label, weight, and structure, never icon clutter and never color-coding. SF Symbols (iOS) or a sparse Lucide set (web, 1.5 stroke, monochrome) are allowed only where an icon is genuinely the clearest signal, rendered in the neutral range. No new decorative icons. No icon-only navigation (labels survive every collapse). Audit symbol semantics: does this glyph actually mean what the action does?

**No emojis. Anywhere. Ever.** Not in the interface, not in chat, not in notifications, not in source strings, under any circumstances. Enforce it with a lint if the project has one.

---

# Part 2: How it acts

## 2.1 The interaction grammar

- **Creation flows:** Cancel on the left, an explicit commit verb on the right ("Save", "Create", never "OK"). The commit stays disabled until the form is valid. Interactive dismiss disabled while committing.
- **Edit flows:** live-save. Every field change writes through immediately; the only toolbar control is a single "Done" that just closes. No Cancel (nothing to discard), no Save (nothing staged).
- **Read-only sheets and apply-on-tap pickers:** single "Done". Secondary utilities move to the leading side.
- The toolbar tells the truth about the persistence model. If the buttons lie about what is staged, the sheet is wrong.
- **Destructive actions: Undo, never "are you sure?"** Deletes act immediately and offer Undo in a floating toast (glass capsule, ~4 seconds, restores exact prior state). The rare warranted confirmation is a native alert whose buttons name where each choice leaves you ("End Shift" / "Keep Logging", never "OK" / "Cancel" and never two labels that read the same).
- Money commits and true data-loss actions keep an explicit confirm; everything routine is silent and reversible.
- **Press feedback on pointer-down, never only on release:** scale to 0.97 (0.95-0.98), ~100ms ease-out, on every pressable thing.
- **No assumptions, ever.** Never prefill a guess. Only facts the system truly knows may prefill a field. "Every shift is different. You can't assume anything." The line to hold: derived facts are welcome (a computed brief, today's date, a total the math produces), guessed inputs are not (a suggested amount, a pattern-predicted value). Show what the system knows; never pre-commit what the user hasn't said.

## 2.2 Focus

- The accent focus ring (2px) is the ONLY focus chrome on inputs on touch platforms.
- On desktop/web, split the model: plain mouse `:focus` gets no halo (the caret and a one-step border darken are the indicator); keyboard `:focus-visible` gets the ring. The accent never fires as a glow on mouse click. "Green stays reserved for actions."
- Inputs are chromeless: transparent or sunken fill, no boxes-in-boxes. A bottom hairline that darkens to ink on focus is the web-app idiom.

## 2.3 Motion

Motion is information, never decoration. An animation earns its place only when it carries new meaning. This is where Emil Kowalski's framework applies directly; internalize it:

1. **Frequency test first.** Something used 100+ times a day (keyboard actions, command palettes) never animates. Occasional surfaces (modals, drawers, toasts) get standard animation. Rare moments (onboarding, a genuine record) may get delight.
2. **Ease-out for everything entering or exiting.** Never ease-in on UI (it reads sluggish). The house curve on the web is expo-out: `cubic-bezier(0.16, 1, 0.3, 1)`.
3. **Fast.** UI animations stay under 300ms: press ~100ms, hover/chrome 150-200ms, modals/drawers ~300ms, reveals 400-500ms. Durations sit on a 100ms grid.
4. **Never enter from scale(0) or from nothing.** Enter from scale 0.97 (0.9 floor) plus opacity. Elements enter and exit along the same path, and exits are always faster than entrances.
5. **Only animate transform and opacity.** Never animate width/height/padding on interactive controls (it clips mid-grow). State swaps inside a fixed-size slot crossfade with zero translation so nothing on the screen ever moves when state changes.
6. **Springs on Apple platforms:** one spring per interaction class, drawn from shared tokens. `.snappy` for precise UI, `.smooth` for gentle transitions, a deliberate bounce (~0.24) only for a playful drawer. For agentic/live UI: response 0.30-0.40, damping 0.80-0.85. No linear anywhere except reduced-motion fallbacks.
7. **Interruptible.** CSS transitions over keyframes for anything rapidly re-triggered; springs preserve velocity when interrupted. Gestures use momentum (a flick dismisses regardless of distance) and rubber-band at boundaries.
8. **Nothing loops on stable content.** A shimmer on live progress is fine (each frame reports work); a pulse on an unchanged number is not. The one sanctioned ambient loop is a quiet breathing dot (opacity only, never size) meaning "recording right now".
9. **Reduce Motion is mandatory, with a designed fallback** (crossfade, ~120-150ms), on every animation, every platform, no exceptions.
10. Numbers roll, glyphs settle, and a scene change is a lens change: content swaps directionally while everything above the fold holds still. Tyler flags "too much abrupt movement between screens" instantly.

Haptics (touch platforms) communicate state, never effort: light tap for navigation, selection tick for scrubbing, success only for a real save, error for real failure. Routine, reversible changes are silent. Gate on Low Power Mode.

## 2.4 Every screen designs five states

Loading (skeleton lines that match the real layout, never a spinner on blank), content, empty, error, and offline/degraded. The happy path is one of five states, never the only one.

- Empty states are calm, specific copy plus at most one action. First-run empty ("Nothing logged yet this period") differs from filtered-empty ("No results for 'query'", which always names the recovery: "Turn off the low-stock filter to see all products"). No illustrations.
- Error states say why nothing is shown and preserve honesty: "Nothing is shown rather than out-of-date records. Check your connection and reload."
- The best empty state knows things: an empty home or chat opens as a computed brief of real facts, never a void. "The screen is empty, not calm" is a rejection.
- Loading never evicts the previous result. Calm over noisy.

## 2.5 Web app vs website

When the thing being built is a tool someone operates for hours (an app), app instincts beat website instincts, always:

- URLs are addressable state; refresh restores exactly where the user was, never dumps them home.
- Keyboard first-class: global search on Cmd/Ctrl-K, Esc closes the topmost layer, Enter submits, arrows navigate lists. Repeated actions get a shortcut and no animation.
- Nothing is ever re-entered: drafts, filters, and last-viewed context survive refresh.
- Optimistic writes, with undo.
- Density and persistence over air: never scroll a person's context away. "The urge to add breathing room is a website urge; resist it here."
- Kill web-tells on touch: no tap highlight, no rubber-band on chrome, no text selection on buttons, `dvh` alongside `vh`, safe-area insets on fixed bars, 16px minimum on focusable inputs (never `maximum-scale=1`).
- Trust is reliability: never lose data, never show the wrong record (always disambiguate people by phone or email), behave identically every time. That outranks any animation.
- Scale is solved by retrieval, never density: search-first screens, virtualized lists, "Show earlier" pagination, type-ahead comboboxes instead of giant dropdowns.

Print stylesheets are first-class when anything gets handed to a human on paper: pure black on white, points not pixels, repeating table headers across pages, rows never split, no shrinking, landscape when the data is wide, all interactive chrome hidden.

## 2.6 Accessibility floors

- **Contrast:** body and secondary text meet 4.5:1 against their surface. Tertiary (`#AEAEAE`) is for placeholders and hints only, never information-bearing text; if a reader must learn something from it, promote it to secondary.
- **The one-accent system is inherently color-blind-safe. Keep it that way:** meaning is never encoded in hue alone; every colored signal is paired with a label, weight, or position that carries the same fact.
- **Screen readers:** interactive things are real buttons, never tap-gesture rectangles. Composite rows combine into one element whose label reads as a sentence (the whole "Flexible spending is running at 118% of plan", never the fragments). Live announcements for state changes worth knowing.
- **Dynamic Type and zoom:** layouts degrade by stacking, never by clipping. Button labels never hyphen-wrap; give them room to reflow or scale. Test at accessibility text sizes before calling a screen done.
- Reduce Motion (Part 2.3), Reduce Transparency (glass goes opaque; never encode information only in what shows through it), 44pt targets, and the 16px mobile input floor are all part of this contract, not extras.

## 2.7 Defaults implementers ask about

- Form validation: inline, under the field it belongs to, appearing on blur or submit, never a summary box at the top.
- Tables: numbers right-aligned with tabular figures, text left-aligned, no zebra striping, sorting on column headers.
- Toasts: one at a time; a new toast replaces the current one. Bottom-centered (web) or above the tab bar (iOS).
- Layering, bottom to top: content, sticky chrome, popovers, drawers and sheets, alerts, toasts.
- Dates and times: native pickers, formatter-built relative dates, en-dash ranges, explicit weekday when ambiguity costs anything.

---

# Part 3: How it speaks

## 3.1 The copy law

**Every string states a fact or names an action.** No taglines, no pep, no greetings-with-aphorisms, no filler. Calm, specific, plain English, sentence case.

- **No emojis. No em dashes** (use a comma, colon, or period). **No exclamation marks** doing the enthusiasm's job. Avoid the sentence shape "not just X, but Y". "Nice night." is allowed; "Great job!!" is not.
- **Say it once.** No screen states the same fact twice. A full progress bar, a "Last pay period" label, and a "Period complete" header are one fact billed three times; delete two. A derived fact is not a repeat ("That's 6h 23m" under two timestamps earns its place by doing arithmetic for the reader).
- **Evidence before advice.** "Groceries ran over budget 4 of the last 6 months by an average of $85" beats "You should spend less on groceries." If there is no evidence, silence beats weak advice.
- **Name the act, never the mechanism.** "Analyzing" not "Reading with AI." Never advertise the technology. A label that names nothing (true of every row it sits on) gets deleted.
- **Every number gets a sentence; every state gets a reason.** A raw figure never sits alone: fold it into prose ("Flexible spending is running at 118% of plan."), weight the number, never color it. Degraded states explain themselves in plain language ("Chase didn't answer, so this uses the latest saved sync."), never error codes, never red panic.
- **One direction per ledger.** Money breakdowns run additions, then a subtotal, then subtractions, then the total. "Up, up, up, then down." Never alternating signs. Every visible figure must reconcile to the total it sits under.
- **Tense honesty.** "Payday · Wed, Jul 29" while upcoming; "Paid Jul 24" only once it landed. Never claim done, live, or sent ahead of the fact.
- Lead with the answer. In AI responses: the figure first ("$184 at Starbucks this month"), one or two sentences, natural contractions, no canned openers ("Looking at your data..."), no closing offers ("Let me know if..."), and when the job is done, stop.
- **Notifications must name at least one concrete fact that exists at send time** (a name, amount, count, or date). Zero facts means do not send, ever. No filler nudges.
- Labels say what a thing IS, never when it appears. Naming is load-bearing: pick the word that makes the meaning exact ("Standing instructions" over "Notes"), keep the same word for the same concept across every surface (screen, print, and assistant all agree), and never let one word carry two meanings in the same type.
- Restraint at rest, depth on demand: the resting state of any surface answers one question with one focal object. Supporting detail is one calm tap away, the action one tap beyond that.

## 3.2 Voice by surface

Product UI is a calm, precise tool. An in-product assistant is a warm, competent colleague: facts stay plain (numbers, dates, and dosages are never dressed up), a dead end gets a direction rather than an apology, and the banned-phrase list applies ("How can I help you today", "Great question", "I'd be happy to", "No problem at all", "Is there anything else"). Marketing copy speaks as "I", never "we", with no hype, no "seamless/robust/cutting-edge/leverage", no claiming difficulty as a credential, and honest edges ("If I am not the right fit, I will say that too.").

---

# Part 4: The context dial

One identity, different surface treatments. Read which context you are in:

| Context | Canvas | Accent use | Type | Signature |
|---|---|---|---|---|
| **iOS/native app** | Alabaster/obsidian, one lifted hero card | One brand accent that acts | SF Pro, money rounded-heavy | Glass chrome, springs, haptics, undo toasts |
| **Desktop tool (web/Electron/Mac)** | Flat white canvas, frosted sidebar, floating toolbar | Accent for status, links, one primary pill | System stack, 13px floor, tabular nums | Hairline separation, keyboard-first, no cards at rest |
| **Phone tier of a web app** | Same tokens, bottom tab bar (icon+label, never hamburger) | Same | 16px input floor | Native-feel gestures, collapsing large title, full-screen sheets with grabbers |
| **Marketing site** | Warm dark (`#0f0d0b`) or warm paper | One loud accent (e.g. signal red), italic serif emphasis | Editorial serif display + mono or sans body | Texture (noise/halftone/grain), scroll-triggered wipes, terminal or magazine metaphors |

Two more dials:

- **Brand temperature.** Every product has a warmth setting: "warm and roomy" (a caring practice tool) vs "compact and cool" (a developer-adjacent tool). Borrowed patterns must be re-tempered to the destination brand before use.
- **Platform convention outranks brand.** On macOS the user's system accent drives controls. Apple's semantic adaptive surfaces are native and welcome. Cross-platform ports copy the machinery, never the skin: architecture and behaviors transfer, colors and personality never do.

---

# Part 5: How it's built

## 5.1 Design system mechanics

- All tokens (color, type, spacing, radius, motion, haptics, shadows) live in exactly ONE file per platform (a `DesignSystem.swift`, a single `:root` block in one stylesheet). Zero `!important`. Views speak only through tokens; inline hex, raw `Color(red:)`, raw system font sizes, and raw haptic generators outside that file are bugs.
- Write a design lint (a grep script is fine) and run it before every commit: no forbidden hues, no emojis in source, no serif, no raw materials or glass outside the wrappers, no raw font sizes. Zero-tolerance.
- One shared modifier per repeated pattern (THE card, THE sheet shell, THE status badge) so instances are literally identical, never merely similar.
- Shared vocabularies get one owner: one `statusTone()` function, one `stockLabel()`, one currency formatter. Every status pill in the app reduces to the same three or four tones; never invent a fifth.

## 5.2 Data and logic discipline

- **Derived, never persisted, wherever possible.** Compute groupings and aggregates at read time so changing a setting regroups everything live; records themselves never change.
- **One formula, one owner.** Any number shown in two places is computed in exactly one function both places call. Never a per-screen inline derivation. When client and server must agree, one side declares itself the mirror of the other in a header comment naming the counterpart file, with identical constants and parity tests.
- Distinguish "not entered" from "zero" (optionals stay optional; "never set" can be a meaningful state). New persisted fields are additive and optional with safe fallbacks so migrations never crash old rows; deployed schema fields are never dropped, only documented dormant.
- **Migrations are sacred:** idempotent, self-guarding (fetch only rows that still need work), ledgered, verified inline after apply, applied before the code that reads them ships, with symmetric down scripts. Destructive operations require explicit multi-flag opt-in (an apply flag, a separate production flag, and for the worst cases a named approval string), and never trust a label to identify a database; verify the target itself.
- No regex for natural language. Ever. Use structured markers or let a model handle language.

## 5.3 Honesty in engineering

- **Comments record why, citing the human and the date.** When a product decision comes from a real person, quote them: `TYLER, 2026-08-17: "square and marpe should always be in sync"`. Label a client's or stakeholder's requests in code so cleanup never undoes their decisions.
- **Silent failures get watchdogs.** If a failure mode raises nothing by construction (an update that matches zero rows), build a scheduled read-only drift check that reports and does not repair, plus a dead-man's-switch check-in so the absence of the check itself alerts.
- Never put personal data (names, amounts) in error trackers or logs; use ids. Strip or never collect what a third party should not hold.
- APIs meant for agents or humans return three layers: a human sentence, the structured data, and reply-ready context. Same instinct as the UI: every number gets a sentence.
- Report reality: if tests fail, say so with output. Cron truth lives in the running system, never inferred from config files. A user-reported fix is not complete until the report is closed out with a verified resolution.

---

# Part 6: Process

1. **Prior art before novelty.** Before finalizing any new frontend surface or interaction, search how the best products solve it (the category leaders plus Apple's own apps), and state in one line what was adopted or rejected and why. Never design a novel pattern where a proven one exists; never copy one without saying where it came from.
2. **Exact specs over adjectives.** When handing design to an implementer, give selectors, tokens, px/pt values, spring numbers, full state matrices, and verbatim copy strings. "Clean, fresh, Apple-native, on-brand" is the taste; the redline is the deliverable.
3. **Render-check ritual:** after each screen, screenshot at desktop and phone widths (or both color modes on device), look at the actual pixels against the acceptance criteria, and fix what the render shows before moving on. Automate geometry checks where possible (horizontal overflow, clipped text, sub-floor type).
4. **Minimal first.** Ship the smallest sure version; ambition is a deliberate, separate decision. Restraint is the product: no gamification, no confetti, no streak guilt, no social bolt-ons, no features that serve no core pillar.
5. Review with fresh eyes and in slow motion; unseen details compound. Taste is the differentiator.

---

# Part 7: The blocklist

If you are about to do any of these, stop:

- Blue (or purple, indigo, cyan, mint, teal, pink) anywhere in a product
- A second accent, category color-coding, or a multi-hue chart
- Borders as depth, zebra tables, gradients as decoration
- More than one raised card on a mobile screen; any card grid on a desktop workspace canvas
- Glass on content, fake glass, glass on glass
- Emojis, em dashes, exclamation-mark enthusiasm, "AI" as a label, mechanism-naming copy
- A serif or decorative font inside a product UI
- Confirmation dialogs for reversible actions; "OK/Cancel" button pairs
- Prefilled guesses of user data
- Entering from scale(0), ease-in on UI, animation over 300ms on chrome, layout that shifts when state changes, animation on keyboard-triggered actions, unguarded `repeatForever`
- A spinner on a blank region; an empty state that says nothing; loading that evicts the previous result
- Saying the same fact twice on one screen; a subtitle that paraphrases its title
- A push notification with no concrete fact
- Icons where words work; icon-only nav; new decorative icons
- Text below 13px on desktop; touch targets under 44pt; inputs under 16px on mobile web
- Raw hex, raw font sizes, raw haptics, or raw materials outside the token file
- Floats for money; two names for one concept; one word with two meanings
- Claiming done from code alone

---

# Quick-start tokens for a new project

```
canvas        #FAFAFA light · #050505 dark      (marketing: #0f0d0b warm dark or warm paper)
card          #FFFFFF + 3-layer shadow · #121212 + rgba(255,255,255,0.12) rim
inset field   #F2F2F7 · #1C1C1E
ink           #000000 · #FFFFFF
secondary     #6B6B6B · #8E8E93
tertiary      #AEAEAE · #48484A
accent        one per product, chosen deliberately (B&W until then); it is the only color that acts
caution       #E8590C light · #FF9F0A dark
error         #FF3B30
shadow        0 16px 32px rgba(0,0,0,.04), 0 4px 12px rgba(0,0,0,.06), 0 1px 1px rgba(0,0,0,.08)
easing        cubic-bezier(0.16, 1, 0.3, 1); durations 100/150/200/300/400ms
press         scale(0.97), ~100ms ease-out, on pointer-down
radius        6 · 10 · 14 · 18 · 24 · 9999 (pills)
spacing       4 · 8 · 12 · 16 · 20 · 24 · 32 · 40
type          SF Pro / system stack; money rounded-heavy + tabular figures; weights bumped one step
floors        13px desktop text · 16px mobile inputs · 44pt touch targets
```

Interaction grammar: create = Cancel + disabled commit verb · edit = live-save + Done · delete = act now + Undo toast. Copy: every string states a fact or names an action. Motion: information only, ease-out, under 300ms, reduced-motion always. And above all: look at the render.
