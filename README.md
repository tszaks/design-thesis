# My Design Thesis

I've been building software for a while now. Vero, Payday, Wellspring, a scheduling app, a handful of sites. And somewhere along the way I noticed something: every fight I had with a screen ended the same few ways. The same rules kept winning. Different products, different platforms, same rules.

So I wrote them down.

This is that document. It's not a style guide I aspire to. It's a record of decisions I already made, usually after getting something wrong first. The hex codes are here, but the hex codes are the least of it. What actually transfers is the reasoning.

Three documents live in this repo. This README is the one I wrote for people. [`LLM.md`](LLM.md) is the same law rewritten for machines: paste it into an AI agent at the start of a session and it will build like I do. And [`APPLE_NATIVE.md`](APPLE_NATIVE.md) is the field guide: a catalog of Apple-native SwiftUI components (the drawers, the collapsing bars, the glass, the haptics, the hundred quirks you have to know exist before you can use them), every entry answering EXISTS, USE WHEN, and BUILD, with verified availability floors from iOS 13 to 26, fallbacks for older systems, and the gotchas that only show up in production. For iOS work, paste LLM.md and the catalog together. Steal any of it. That's why it's public.

One honest caveat before anything else. This thesis was distilled primarily from Payday and Vero (they're the proof), from Wellspring's translation of their law to a desktop tool, and from years of my own feedback. It's the standard I'm consolidating toward. It is not a claim that every repo I own already honors it. Some don't. Some never should.

Which is why, if you're an agent or a person about to touch one of my repos, you check jurisdiction first:

- **Payday and Vero iOS** get the full law. These are the reference implementations.
- **Wellspring** gets the full law plus its own documented exceptions (its own green, light-only, a flat desktop canvas), each recorded with a reason.
- **New products** get the full law from day one.
- **Older repos not yet under the law** get migrated deliberately, as real commissioned work. Never drive-by "fixed."
- **Marketing and client sites** inherit the copy law and the restraint. That's it. Not the palette, not the elevation budget, not the Apple bar. Restyling a client's site into alabaster and one green isn't compliance. It's vandalism.

My acceptance bar for my own products is simple. Put one next to the others and a stranger should say "same company, obviously." A reviewer should plausibly ask "wait, is this Apple's?" That's a taste target, not a checklist item. Shipping a screen's five states beats polishing its chrome every single time.

Version 1.2, August 2026. This document changes when my taste does, in place, never forked. The blocklist near the bottom is hard law with zero exceptions. Everything else is judgment.

## The rule that outranks everything

Let me start with the one I learned the hard way.

I once had an agent sweep a codebase over to new color tokens. It did exactly what I asked. It also flattened a set of input fields to white-on-white, erasing their edges entirely, because the letter of the rule said so. My feedback at the time: "you took this too literally without actually thinking about how things look."

So here's the meta-rule, and it sits above every rule below it: **beautiful, considered design trumps the rules.** They're guardrails, not a find-replace mandate. If following a rule literally makes a screen worse, you're applying it wrong. A token swap that erases real structure is a regression, not compliance.

The corollary is just as binding: **look at the actual render before you call anything done.** Both color modes. Real sizes (1440px and 390px on the web, a real device on iOS). Never claim done from code alone. I have never once regretted looking at the render. I have frequently regretted not looking.

Three words govern everything: **effortless, simple, intelligent.**

## If you read nothing else

1. Look at the render before declaring done. Beautiful design outranks every rule here.
2. One accent per product, and it appears only where something acts. No blue, ever (system chrome is exempt).
3. Depth from shadow, never borders. Roughly one raised object per mobile screen; desktop canvases stay flat.
4. Glass is chrome, content is solid.
5. Text-forward: no icon where a word works, no emojis anywhere.
6. Create = Cancel + disabled commit verb. Edit = live-save + Done. Delete = act now + Undo toast, never "are you sure?"
7. Every string states a fact or names an action. Say each fact once. No em dashes, no exclamation marks.
8. Motion is information: ease-out, under 300ms, never from scale(0), reduced-motion respected always.
9. Every screen designs five states: loading, content, empty, error, offline. Never a spinner on blank.
10. Tokens live in one file per platform. Views never invent colors, sizes, or curves.

## One habit worth stealing

Before any frontend work, my agents install Emil Kowalski's design engineering skill:

```
npx skills add emilkowalski/skill
```

I adore Emil's work and weave it into everything I build. His animation framework is inlined in the motion section below, so the thesis stands on its own either way.

---

# Part 1: How it looks

## One color that acts

Every product I build has exactly one color that's allowed to do anything. Every button, every active state, every selection, every interactive accent is that color or it's neutral. There is no second action color.

Why? Because color is meaning, and meaning dilutes. The moment a screen has three colors competing, none of them are saying anything. When most of a screen shows no accent at all, that's not a screen waiting for polish. That's the design working.

I don't use blue. Or purple, indigo, cyan, mint, teal, or pink. If you're tempted to tint something blue, the answer is the accent or a neutral. (System chrome is exempt: Sign in with Apple, native pickers, and the macOS system accent stay exactly as the platform ships them. The ban is on choosing blue, never on Apple's own controls.)

Each product picks its own accent, once, deliberately. Vero and Payday share a green (`#00B83F`). Wellspring runs a deeper evergreen (`#2e7d32`) because it's a sibling brand, not a clone. My own site runs a signal red. A new product starts black and white until the accent decision is made on purpose. Don't default to a hue.

Two warning signals exist, and only two: caution (`#E8590C` light, `#FF9F0A` dark) means watch out, and error red (`#FF3B30`) means real trouble or lost data. They never decorate. Red fill is reserved for the actions that lose data. If a color shows up that isn't the accent, a neutral, caution, or error, it's a bug.

Two more things about color, because they're where everyone drifts:

Color is born in the model layer. A typed enum returns a legal token; a view never invents a color from business logic. If you're writing `.purple` in a view, the model is missing an abstraction, not the palette missing a hue.

Charts get one hue. One chart answers one question with one color. Multiple series get opacity steps of the accent, then gray. Heatmaps are a single-hue opacity ramp (about 0.2 at the floor up to 1.0), never a red-to-green temperature walk. I tried the temperature walk once. It rendered as muddy browns and I reversed it on sight.

And for anything live or agentic, the accent becomes a sentence: **green is agency, ink is truth.** The accent means the system is doing something right now. The moment a value is verified and final, it settles to ink. A number may not render in settled ink until the operation that produced it has returned. And when something fails, failure wears plain clothes: calm ink, plain words, never red panic.

## Warm paper, obsidian dark

Light and dark are two designs, each built to its own ideal. Never one design and its inversion.

Light mode is alabaster and ink: an off-white paper page (`#FAFAFA`, never pure white, never a cold gray), pure white cards that lift off it with a real shadow, true black text. Swiss minimalism. Crisp and calm.

Dark mode is obsidian: a near-black page (`#050505`), `#121212` cards, and darkness itself is the shadow. On obsidian, black shadows are invisible, so elevated cards get a hairline rim (`white @ 0.12`) instead.

Native apps ship both, co-equal. Desktop web tools may ship light-only (declare `color-scheme: light`, structure the tokens so dark can come later; that's Wellspring's answer). When dark ships, it gets designed. Never auto-inverted.

There are three surfaces, and you need all three:

| Surface | Light | Dark | Role |
|---|---|---|---|
| Page | `#FAFAFA` | `#050505` | the canvas |
| Card | `#FFFFFF` + shadow | `#121212` + rim | lifts OFF the page |
| Inset field | `#F2F2F7` | `#1C1C1E` | sits WITHIN the page (inputs, search, note boxes) |

Never flatten a field to the tone of its background. White-on-white is the mistake I reverted an entire commit over ("nasty" was my exact word). Restraint means removing noise, never erasing structure.

Text runs ink (`#000000` / `#FFFFFF`), secondary (`#6B6B6B` / `#8E8E93`), tertiary (`#AEAEAE` / `#48484A`). Tertiary is for hints and placeholders only. If a reader has to learn something from it, promote it.

## Shadow, never border

Depth comes from shadow and contrast. Never from color, gradient, or border. Hairlines are allowed as separators (a quiet 1px alpha rule that separates, never decorates), but never as depth.

My house shadow, three layers:

```
0 16px 32px rgba(0,0,0,0.04), 0 4px 12px rgba(0,0,0,0.06), 0 1px 1px rgba(0,0,0,0.08)
```

Dark mode leans on `black @ 0.3, radius 20, y 10` plus a thin contact layer and the rim hairline.

These numbers are a starting point, not scripture. Payday had to strengthen them because they vanished on its real surfaces, and it cited the meta-rule when it did. That's the meta-rule doing exactly its job. Tune the shadow on the render.

Now the part people get wrong: **the elevation budget.** I said it plainly once about my own app: "those boxes look awesome in some places, yet in others they look cheap." Shadow marks the one thing a screen is about. On mobile that means roughly one raised object per screen, the hero, with everything else flat under whitespace, typography, and hairlines. Raised means shadowed; flat sections with small tracked labels are unlimited. (Payday's dashboard is the worked example: exactly one card is lifted. The paycheck comparison, the shift list, the insights all sit flat.) Card-in-card and card-in-sheet are always wrong.

On desktop, go further. The workspace canvas is flat. Shadows exist only on things that actually float: modals, popovers, drawers, toasts, sticky chrome. A desktop tool is not a card dashboard.

On marketing sites, texture takes elevation's job: subtle noise at 3% opacity, halftone dot screens, paper grain. Still never decorative borders.

## Glass is chrome

Glass belongs to the layer that floats: tab bars, toolbars, sheets, toasts, the one sticky bar that content scrolls underneath. Content is solid. Cards, rows, lists, and tables hold information; they don't shimmer.

On iOS that means the real system glass, routed through shared wrappers, never raw call sites, and never a hand-rolled imitation (no blur-plus-gradient-plus-shadow pretending). On the web it means `rgba(255,255,255,0.72)` with `backdrop-filter: blur(20px) saturate(180%)` on the one bar that earns it, and nothing else translucent anywhere.

Glass creates depth by contrast. If everything is glass, nothing is. And the mistake runs both directions: an avatar is content, so it gets no glass bubble. A scan button is an input method, not a feature, so it must not outrank the fields it fills.

## Typography

One font family per product. On Apple platforms that's SF Pro, full stop, no serif in a product UI. On the web it's the system stack or Inter. Hierarchy comes from weight, size, and structure. Never from color. Never from icons.

Money gets special treatment because money is the product: SF Pro Rounded, heavy, at fixed display sizes (60 down through 20pt), the same move Apple Wallet makes. Every live number carries tabular figures so layout never shifts, and rolls to new values instead of snapping. Money is stored as integer cents, never float dollars, and formatted through exactly one shared formatter. (The money clauses apply to products that touch money. A scheduler can skip them. The tabular figures stay regardless.)

UI text uses semantic type styles with weights bumped one step over platform defaults: body reads semibold, headlines bold. A raw font size in a feature view is a bug; type is always a token.

Floors: nothing below 13px on desktop surfaces (dense chrome like calendar grids and chart axis labels can go to 10-12px, but only when a measured audit shows it's needed). Section eyebrows are small caps-style kickers: caption size, semibold, tracked wide, in the accent. That's the sanctioned way to head a flat section.

Marketing sites are the one place display serifs and monospace get to live. Playfair or Instrument Serif for editorial headlines, IBM Plex Mono for terminal moods, italic serif as the emphasis device. It never crosses into product UI.

Small habits that add up: the middot as separator ("$41/hr · your best rate"), a true minus sign in ledgers, curly apostrophes, en-dash ranges, and hours as "6h 23m," never decimals.

## Spacing, radius, icons

Spacing sits on a 4pt grid: 4, 8, 12, 16, 20, 24, 32, 40 (a product can add a step through its own DESIGN.md; Payday ships a 30). Radii come from one scale: 6, 10, 14, 18, 24, 9999. Buttons, chips, and badges are pills. Touch targets are 44pt minimum. Random values are how a system drifts, so reach for the scale first.

Icons: no icon where a word works. Labels, weight, and structure do the differentiating. SF Symbols or a sparse Lucide set are allowed where an icon is genuinely the clearest signal, rendered neutral, and that's it. No decorative icons. No icon-only navigation. And ask the annoying question every time: does this glyph actually mean what the action does?

**No emojis. Anywhere. Ever.** Not in the interface, not in chat, not in notifications, not in source. I lint for it.

---

# Part 2: How it acts

## The interaction grammar

Here's a test I run on every sheet: do the toolbar buttons tell the truth about the persistence model?

Creating something? Cancel on the left, an explicit verb on the right ("Save," "Create," never "OK"), and the verb stays disabled until the form is valid. There's staged work, so both buttons are honest.

Editing something? Live-save. Every change writes through the moment it happens, so the only control is a single "Done" that closes the sheet. A Cancel button on a live-save sheet is a lie: there's nothing to discard.

Deleting something? Do it immediately and offer Undo in a toast for a few seconds. Never "are you sure?" Confirmation dialogs make the user pay attention so the software doesn't have to. Undo is the grammar of forgiveness. (The rare confirm that's actually warranted, real data loss or money moving, is a native alert whose buttons name where each choice leaves you. "End Shift" next to "Keep Logging." Never "OK" and "Cancel.")

Press feedback happens on pointer-down, never only on release: everything pressable scales to 0.97 for about 100ms.

And one rule I hold absolutely: **never prefill a guess.** Only facts the system truly knows may prefill a field. My words, from the app that taught me this: "Every shift is different. You can't assume anything." Derived facts are welcome (today's date, a computed total, a brief built from real data). Guessed inputs are not. Show what you know. Never pre-commit what the user hasn't said.

Focus follows the same restraint. On touch, a 2px accent ring is the only focus chrome. On desktop, split it: mouse focus gets no halo (the caret and a slightly darker border are the signal), keyboard focus gets the ring. I once rejected an entire build because the accent glowed every time I clicked a search field with the mouse. The accent is for actions.

## Motion

Motion is information. An animation earns its place only when it carries new meaning. This is where Emil's framework applies directly, and I hold every piece of it:

Ask how often the user will see it. Something used a hundred times a day never animates. Modals and drawers get standard treatment. A first-run moment or a genuine record can have delight.

Ease out, always. Ease-in reads sluggish because it delays the exact moment the user is watching. My house curve on the web is `cubic-bezier(0.16, 1, 0.3, 1)`.

Stay under 300ms for UI. Press feedback around 100, chrome 150-200, modals 300, reveals up to 500. Durations sit on a 100ms grid.

Never enter from nothing. Scale from 0.97 with opacity, never from zero. Exits are faster than entrances, along the same path they came in.

Only animate transform and opacity. Never animate a control's width or height (it clips mid-grow). When state changes inside a slot, crossfade in place with zero translation, so nothing on the screen ever moves when state changes.

Nothing loops on stable content. A shimmer over live progress is fine; each frame reports work. A pulse on a number that hasn't changed is not. The one ambient loop I allow is a quiet breathing dot (opacity only, never size) that means "recording right now."

Reduced motion is mandatory, with a designed crossfade fallback, on every animation, every platform. No exceptions, including the screen shown at every launch.

On Apple platforms, one spring per interaction class, from shared tokens. And a scene change should feel like a lens change: content swaps directionally while everything above it holds still. I flag "too much abrupt movement between these screens" instantly, because I've written that exact sentence before.

Haptics communicate state, not effort: a light tap for navigation, a tick for scrubbing, success only for a real save. Routine, reversible changes are silent. A haptic that fires constantly stops being felt.

## Five states, every screen

The happy path is one of five states, never the only one. Every data surface designs loading (skeletons that match the real layout, never a spinner on blank), content, empty, error, and offline.

Empty states are calm, specific, and useful. First-run empty ("Nothing logged yet this period") is different from filtered-empty, which always names the way out ("Turn off the low-stock filter to see all products"). Error states say why nothing is shown: "Nothing is shown rather than out-of-date records. Check your connection and reload." That sentence does more for trust than any illustration, which is why there are no illustrations.

The best empty state knows things. An empty home should open as a short brief computed from real data, never a void. I rejected a chat screen once with the line "the screen is empty, not calm." There's a difference. Learn to see it.

And loading never evicts the previous result. Calm over noisy.

## Apps are not websites

When the thing you're building is a tool someone operates for hours, app instincts beat website instincts every time they conflict.

URLs are addressable state: refresh restores exactly where the user was. The keyboard is first-class: Cmd-K search, Esc closes the top layer, Enter submits, and anything done dozens of times a day gets a shortcut and no animation. Nothing is ever re-entered; drafts, filters, and context survive refresh. Writes are optimistic, with undo. Density beats air (the urge to add breathing room is a website urge; resist it in a tool). On touch, kill the web-tells: no tap highlight, no rubber-banding chrome, no selectable button labels, real safe-area handling, 16px minimum on inputs.

Trust is reliability, not decoration. Never lose data. Never show the wrong record (disambiguate people by phone or email; two clients named Maria is a safety issue, not a cosmetic one). Behave identically every time. That outranks any animation.

Scale is solved by retrieval, never density: search-first screens, virtualized lists, "Show earlier" pagination, type-ahead pickers instead of hundred-row dropdowns.

And when something gets handed to a human on paper, the print stylesheet is a first-class surface: pure black on white, repeating table headers, rows never split across pages, no shrinking. The person reading it at home doesn't care that it was a webpage.

## Accessibility floors

Body and secondary text meet 4.5:1 against their surface. Tertiary gray is placeholders and hints only. The one-accent system is inherently color-blind-safe; keep it that way by never encoding meaning in hue alone. Interactive things are real buttons, never tap-gesture rectangles, and composite rows read to a screen reader as one full sentence. Layouts degrade by stacking, never clipping, and button labels never hyphen-wrap. Test at accessibility text sizes before calling a screen done. Reduce Motion, Reduce Transparency, 44pt targets, and the 16px input floor are all part of the same contract.

## Defaults, so nobody has to ask

Validation is inline, under the field, on blur or submit. Table numbers right-align with tabular figures; no zebra striping; sorting lives on the column headers. One toast at a time; a new one replaces the old. Layers stack content, sticky chrome, popovers, sheets, alerts, toasts, in that order. Dates use native pickers and honest relative formatting.

---

# Part 3: How it speaks

This is the part I care about most, and the part most software gets wrong.

**Every string states a fact or names an action.** No taglines, no pep, no greetings with aphorisms attached. Calm, specific, plain English, sentence case. No emojis, no em dashes (use a comma, a colon, or a period), no exclamation marks doing the enthusiasm's job. "Nice night." is allowed. "Great job!!" is not.

**Say it once.** No screen states the same fact twice. A full progress bar, a "Last pay period" label, and a "Period complete" header are one fact billed three times; delete two of them. A derived fact is not a repeat: "That's 6h 23m" under two timestamps earns its place because it does arithmetic the reader would otherwise do.

**Evidence before advice.** "Groceries ran over budget 4 of the last 6 months by an average of $85" beats "You should spend less on groceries." And when there's no evidence, say nothing. Silence beats weak advice.

**Name the act, never the mechanism.** "Analyzing," not "Reading with AI." Saying "AI" in a label is tacky, and I've deleted it every time it's appeared. A label that's true of every row it sits on names nothing, and gets deleted too.

**Every number gets a sentence. Every state gets a reason.** A raw figure never sits alone; fold it into prose ("Flexible spending is running at 118% of plan.") and let weight, not color, make it land. When something degrades, it explains itself like a person would: "Chase didn't answer, so this uses the latest saved sync." Never an error code. Never red panic.

**Ledgers run one direction.** Additions, then a subtotal, then subtractions, then the total. Up, up, up, then down. Never alternating signs down a column. And every figure on the face must reconcile to the total it sits under; a card that argues with itself is broken.

**Tense honesty.** "Payday · Wed, Jul 29" while it's upcoming. "Paid Jul 24" only once it landed. Never claim done, sent, or live ahead of the fact.

**Notifications name a concrete fact or don't send.** A merchant, an amount, a count, a date, something real that exists at send time. Zero facts, zero notification. No filler nudges, ever.

Lead with the answer everywhere. An AI response starts with the figure ("$184 at Starbucks this month"), runs one or two sentences, uses contractions, skips the canned openers ("Looking at your data...") and the canned closers ("Let me know if..."), and when the job is done, stops.

Names are load-bearing. Pick the word that makes the meaning exact ("Standing instructions" instead of a fourth field called "Notes"), keep the same word for the same concept on every surface, and never let one word carry two meanings in the same place.

The feeling all of this adds up to: restraint at rest, depth on demand. Any surface at rest answers one question with one focal object. The detail is one calm tap away. The action is one tap past that.

---

# Part 4: The context dial

One identity, four surface treatments. Read which one you're in before you write a line:

| Context | Canvas | Accent use | Type | Signature |
|---|---|---|---|---|
| **iOS/native app** | Alabaster/obsidian, one lifted hero card | One brand accent that acts | SF Pro, money rounded-heavy | Glass chrome, springs, haptics, undo toasts |
| **Desktop tool** | Flat white canvas, frosted sidebar, floating toolbar | Accent for status, links, one primary pill | System stack, 13px floor, tabular nums | Hairlines, keyboard-first, no cards at rest |
| **Phone tier of a web app** | Same tokens, bottom tab bar (icon+label, never hamburger) | Same | 16px input floor | Native-feel gestures, collapsing title, full-screen sheets |
| **Marketing site** | Warm dark or warm paper | One loud accent, italic serif emphasis | Editorial serif + mono or sans | Texture, scroll wipes, terminal or magazine moods |

Two dials on top of that. Every product has a temperature: warm and roomy, or compact and cool. When you borrow a pattern from one product, re-temper it before it lands in another. And platform convention outranks brand: on macOS the user's system accent drives controls, Apple's semantic surfaces are welcome, and cross-platform ports copy the machinery, never the skin.

---

# Part 5: How it's built

The design system is enforced, not aspirational. All tokens live in exactly one file per platform, and a lint script (a grep is fine) runs before every commit: no forbidden hues, no emojis in source, no serif, no raw font sizes, no raw glass. Zero tolerance. Repeated patterns get one shared implementation (THE card, THE sheet, THE badge) so instances are literally identical. Shared vocabularies get one owner: one status function, one currency formatter, and never a fifth status color.

Data follows the same discipline. Derive instead of persisting wherever possible, so changing a setting regroups everything live. One formula, one owner: a number shown in two places is computed in one function both places call, and when client and server must agree, one file declares itself the mirror of the other by name, with parity tests. "Not entered" and "zero" stay different facts. New fields are additive and optional so old rows never crash. Migrations are idempotent, ledgered, verified after apply, shipped before the code that reads them, and destructive operations take deliberate multi-step opt-in. And no regex for natural language. Ever.

Then there's the part I'd call engineering honesty. Comments record why, citing the human and the date ("TYLER, 2026-08-17: ..."), so a cleanup pass never undoes a real person's decision. Silent failures get watchdogs: if a bug can happen without raising anything, build a scheduled read-only check that reports and doesn't repair, plus a dead-man's switch so the absence of the check itself alerts. No personal data in error trackers; use ids. And report reality: failing tests get reported as failing, and a user-reported bug isn't fixed until the person who reported it would agree.

---

# Part 6: Process

Prior art before novelty. Before finalizing any new surface, look at how the best products solve it, and state in one line what you adopted or rejected and why. Never invent a pattern where a proven one exists; never copy one without saying where it came from.

Exact specs over adjectives. When design gets handed to an implementer, it goes as selectors, tokens, pixel values, spring numbers, state matrices, and verbatim copy. "Clean, fresh, Apple-native" is the taste. The redline is the deliverable.

Render-check after every screen. Screenshot at both widths or both modes, look at the actual pixels, fix what the render shows, then move on. Automate the geometry checks where you can.

Minimal first. Ship the smallest sure version; ambition is a separate, deliberate decision. And restraint is the product: no gamification, no confetti, no streaks, no social bolt-ons.

Review with fresh eyes, and in slow motion. Unseen details compound. Taste is the differentiator.

---

# Part 7: The blocklist

Hard law. If you're about to do any of these, stop:

- Blue (or purple, indigo, cyan, mint, teal, pink) anywhere in a product
- A second accent, category color-coding, or a multi-hue chart
- Borders as depth, zebra tables, gradients as decoration
- More than one raised card on a mobile screen; any card grid on a desktop workspace canvas
- Glass on content, fake glass, glass on glass
- Emojis, em dashes, exclamation-mark enthusiasm, "AI" as a label, mechanism-naming copy
- A serif or decorative font inside a product UI
- Confirmation dialogs for reversible actions; "OK/Cancel" button pairs
- Prefilled guesses of user data
- Entering from scale(0), ease-in on UI, animation over 300ms on chrome, layout that shifts when state changes, animation on keyboard-triggered actions, unguarded looping animation
- A spinner on a blank region; an empty state that says nothing; loading that evicts the previous result
- Saying the same fact twice on one screen; a subtitle that paraphrases its title
- A push notification with no concrete fact
- Icons where words work; icon-only nav; new decorative icons
- Text below 13px on desktop; touch targets under 44pt; inputs under 16px on mobile web
- Raw hex, raw font sizes, raw haptics, or raw materials outside the token file
- Floats for money; two names for one concept; one word with two meanings
- Claiming done from code alone

---

# Quick-start tokens

```
canvas        #FAFAFA light · #050505 dark      (marketing: warm dark or warm paper)
card          #FFFFFF + 3-layer shadow · #121212 + rgba(255,255,255,0.12) rim
inset field   #F2F2F7 · #1C1C1E
ink           #000000 · #FFFFFF
secondary     #6B6B6B · #8E8E93
tertiary      #AEAEAE · #48484A
accent        one per product, chosen deliberately (B&W until then); the only color that acts
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

The shadow row is a starting point; strengthen it if it vanishes on your real surfaces (mine did).

Create = Cancel + disabled commit verb. Edit = live-save + Done. Delete = act now + Undo toast. Every string states a fact or names an action. Motion is information, ease-out, under 300ms.

And above all: look at the render.
