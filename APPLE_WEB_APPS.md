# How Apple Builds Web Apps

The sibling of [APPLE_WEB.md](APPLE_WEB.md), one level deeper. That document covers Apple's web *sites*; this one covers the web *applications* — icloud.com/calendar, Mail, Notes, Photos, the iWork suite — desktop-class software running in a browser tab.

Read this with one framing in mind: **steal the architecture, beat the chrome.** The structural layer of iCloud's web apps (the shell, the sync model, the realtime channel, the optimistic-write grammar, the keyboard registry) is genuinely excellent and largely invisible. The visual layer is the weakest part of Apple's web estate — dated controls, heavy loads, no offline — and a disciplined small team can out-execute it. The architecture is the prize.

**Method and honesty.** Compiled 2026-08-26 from live probes: the icloud.com shell HTML and its 3.9 MB main bundle, the per-app bundles (Calendar 3.4 MB, Mail 5 MB, Notes 8 MB, Photos, Find My, Drive), Calendar's 398 KB CSS, music.apple.com and tv.apple.com with their manifests — plus Apple docs, WWDC sessions, job postings, and reverse-engineering projects. Claims labeled **[CONFIRMED]** / **[REPORTED]** / **[SPECULATIVE]**. Sources at the end.

---

# Part 1 · The shell: "CloudOS"

## One shell, many iframed apps [CONFIRMED]

icloud.com is a parent React shell that hosts every app in a **single full-viewport iframe** (`id="early-child"`). The inline bootstrap in the shell HTML contains the literal routing table:

```js
["calendar","invites","contacts2","contacts","iclouddrive","keynote",
 "mail2","notes3","numbers","pages","photos3","reminders2"]
```

When the URL matches an app, the bootstrap creates the iframe **before the shell's own React loads** — the app starts downloading in parallel with the shell (that's why it's named early-child; a straightforward perceived-performance win):

```js
var E = n.createElement("iframe");
E.id = "early-child";
E.src = "https://" + host + "/applications/" + app + "/current/" + locale +
        "/index.html?rootDomain=..." + "#launchRoute=" + encodeURIComponent(deepLink);
```

Note **`#launchRoute=`**: the original deep link (`icloud.com/calendar/view/week/...`) is handed to the child in the URL fragment, so the app restores exact state. The iframe stays `visibility:hidden` and "unclaimed" until the app claims it — no flash of loading chrome. The internal codename is still **CloudOS** (it dates to the 2011 SproutCore era), and child↔parent communication is a postMessage bus carrying `{isCloudOSMessage: true, methodName, appName, buildNumber, ...}`.

## Versioned, immutable builds [CONFIRMED]

Two-level versioning: the shell at `/system/icloud.com/2630Build35/en-us/`, and **each app independently versioned** — on the same probed day, Calendar was `2630Build17`, Mail `2630Hotfix39`, Find My still `2536Project29`. The `current` path segment is a server-side alias, so the shell never knows app versions; a release is publishing a new immutable build directory (cacheable forever) and flipping the alias. A shared "mastering number" (`2630`) aligns the independently-shipped fleet to one release train. Apps are **localized at build time** — locale is a path segment, not a runtime lookup.

## Navigation: SPA inside, MPA between [CONFIRMED]

Switching apps is a **real page load** of the shell at the new path; navigation *within* an app is fully client-side, with the child reflecting state to the address bar through the postMessage bus. Deliberate: each app is its own browsing context with its own strict CSP — memory isolation and a security boundary in one move.

## The home screen [CONFIRMED core, REPORTED details]

The 2022 redesign replaced the icon grid with customizable widget tiles, including an iOS-style **jiggle mode** for rearranging. The shell carries a **badge bus** — child apps report unread counts up, the shell renders them on tiles and the favicon. Tile chrome is shell-native React; at least the Photos tile renders content directly in the shell (its fonts and CSS are compiled into the shell bundle), while interactive tiles hand off to app frames [REPORTED for the full split].

## Auth: idmsa, the widget iframe, token handoff [CONFIRMED]

Sign-in is Apple's **AppleID auth widget in its own cross-origin iframe** (`idmsa.apple.com`), spoken to over a postMessage-RPC bridge (pmrpc), with the OAuth grant delivered back via `response_mode: web_message`. Session bootstrap (`setup.icloud.com/setup/ws/1/accountLogin`) returns the per-user **webservices map** (see Part 2). Tokens persist in **IndexedDB** (`icloud-ids-db` → one `ids-data` store) and reach same-origin app frames via a hidden `/bridge` iframe handshake with a 5s timeout.

---

# Part 2 · The data layer

## CloudKit JS vs. the partitioned private services [CONFIRMED]

CloudKit JS is Apple's public SDK (`cdn.apple-cloudkit.com/ck/2/cloudkit.js`): containers → public/private/shared databases, record zones, and the core primitive — **sync tokens**. `fetchDatabaseChanges`/`fetchRecordZoneChanges` return a token plus `moreComing`; you cache the token and pull only deltas forever after.

But most first-party apps talk to **partitioned private REST services**, where `pNN` is the user's data shard:

| App | Endpoint |
|---|---|
| Calendar | `pXX-calendarws.icloud.com` (JSON `/ca/events`, `/ca/startup` — NOT CalDAV; CalDAV exists separately for third-party clients) |
| Contacts / Reminders / Mail / Notes | `pXX-contactsws` / `-remindersws` / `-mailws` / `-notesws` |
| **Photos** | `pXX-ckdatabasews.icloud.com` — the actual CloudKit database web API |
| Drive | `pXX-drivews` + `pXX-docws` (CloudDocs) |
| Find My | `pXX-fmipweb` |

## Realtime: the "webcourier" WebSocket [CONFIRMED]

icloud.com holds one persistent WebSocket to **`wss://websocket.push.apple.com`** — APNs web push, internally "webcourier" — streaming binary push frames, with a long-poll fallback and an idle **connection-parking** mode to save battery. The pattern: subscriptions fire APNs → webcourier delivers a "something changed" ping → the app pulls the delta with its cached sync token. Pushes carry no data; they carry *permission to fetch*.

## Offline: the honest truth [CONFIRMED]

A service worker is registered — **for push only**. There is no offline app-shell caching and no offline data mode. You cannot read your calendar offline on icloud.com. (This is one of the beatable gaps.)

## Optimistic writes with explicit rollback [CONFIRMED]

Calendar's Redux store keeps a `draft` per event and ships reducers named `undoUpdateEvent`, `eventSaveFailed`, `eventDetailsFetchFailed` — optimistic local mutation, server reconciliation, and *named* rollback actions rather than generic error states. iWork gets conflict-free partial saves from its document format (Part 3).

---

# Part 3 · The app UI layer (the copy-able half)

## iCloud Calendar internals [CONFIRMED from the shipped bundle]

- **Stack:** React + Redux. State tree: `event.records{}`, `draft{}`, `selectedEventIds`, `inspectedEventId`, `inspectorIsInCreateMode`.
- **The grid is CSS Grid + absolute overlays, not canvas.** Events place with computed `gridArea: "${top} / ${colStart+1} / ${top+1} / ${colEnd+2}"`; drag previews are `position:absolute` layers with a named `zIndex:"dropTargetPreview"`; all-day events get a separate lane.
- **Drag-to-create and drag-to-move are custom pointer handlers**, not HTML5 drag-and-drop — mouse position tracked in a ref, Redux actions `showCreatingEventOnCanvas`/`createDraft` backing the gesture. And a fossil worth knowing: `e._unsafe_allowPointerPropagationForSproutCore || t.stopPropagation()` — a SproutCore-era interaction layer still lives under the React layer, and pointer events are selectively let through to it.
- **Editing happens in a floating popover inspector** (`eventInspected`, `inspectorIsInCreateMode`), not a page or a full-height sheet.
- **View switcher** is a real segmented control — `<ui-segmented-control role="radiogroup">` — shipped in *two* implementations (a SproutCore `Segmented` view and a React `UISegmentedControl`), mid-migration.
- **Keyboard registry:** `registerShortcutKeyCombo(combo, action)` — throws on duplicate registration; cross-platform modifier normalization (Cmd→metaKey, Win→metaKey on Windows, Option→altKey); and a Map of held meta-key events to work around the macOS "keyup doesn't fire while Cmd is held" bug. 55 `metaKey` references in one app.

## The AppKit-for-web design system [CONFIRMED]

Calendar's CSS ships a full port of UIKit/AppKit semantics as custom properties, with colors decomposed into **HSL triplets** so tint and dark mode recompute live instead of swapping stylesheets:

```css
--theme-color-appTint-h / -s / -l          /* accent as hue/sat/light components */
--theme-color-labelPrimary … labelQuaternary
--theme-color-fillPrimary … fillQuaternary
--theme-color-backgroundPrimary
--theme-color-systemRed-h/s/l
--theme-darken-background-hover / -active
```

Dark mode is `@media (prefers-color-scheme: dark)` flipping token sets; even the favicon flips to a dark variant. The component registry is the AppKit vocabulary: `SplitView`, `ResizeHandleView`, `SearchField`, `Segmented`, `UISwitch`, `UISlider`, toolbars, sidebars, sheets — all first-class primitives. Context menus are custom Apple-style menus on `onContextMenu` (browser menu suppressed) [custom menus CONFIRMED; universal suppression REPORTED]. And **SF Pro is served to every browser** on icloud.com (self-hosted WOFF2 plus the apple.com font service, with per-script variants swapped by locale) — non-Mac users get SF too.

## iWork: the canvas + WASM document engine [CONFIRMED]

Pages/Numbers/Keynote render documents to **`<canvas>`**, driven by the native **C++/Objective-C engine compiled to WebAssembly** — Apple built a custom LLVM-based toolchain with its own linker, an ObjC runtime, a Foundation layer, and a TypeScript bridge (shipped in iWork 10, 2020; current job postings still require SproutCore + WASM + Canvas/WebGL). Text editing is a custom hit-tested editor drawing selection and carets onto the canvas, not contenteditable [REPORTED]. The document format matches: `.iwa` files are **Snappy-compressed Protocol Buffers** with a `should_merge` flag enabling incremental partial saves that map cleanly onto CloudKit record deltas.

## Notes: a protobuf text engine, no third-party editor [CONFIRMED]

Notes is the heaviest client (8 MB) and the most surprising: its rich text is a **custom protobuf document model called TopoText** (186 references), synced via CloudKit and rendered by a bespoke engine (35 `getContext("2d")` sites, OffscreenCanvas, a WebGL path), with contenteditable used as an *input surface*, not the source of truth. Explicitly checked and absent: Slate, Lexical, ProseMirror, Quill, TipTap. **Apple uses no third-party rich-text framework anywhere; the editors are bespoke.** Mail's composer is the lighter contenteditable-plus-execCommand approach.

## Photos, Find My, and the fossil record [CONFIRMED]

Photos rides the real CloudKit web API with a **derivative pipeline** — the server pre-renders thumbnails at multiple resolutions, served from signed expiring `icloud-content.com` URLs, lazy-loaded via IntersectionObserver (grid virtualization [REPORTED]). Find My is the last **SproutCore holdout** — its HTML still loads `javascript-packed.js` from a sproutcore directory — with the map layer on **MapKit JS**. The suite is a 2011→2026 migration still in flight, visible in the shipped code.

## System-integration workarounds [CONFIRMED]

No native menus → custom menu components + contextmenu suppression. Tab badging → the shell rewrites `<link rel="icon">` per app, per dark mode, per unread count. `beforeunload` → clean RPC teardown of the app frame. Clipboard → custom copy/cut/paste handling (iWork uses raw protobuf as its web pasteboard payload). File transfer → a three-step upload saga (reserve → PUT → commit) and signed CDN download URLs.

---

# Part 4 · The doctrine, and the honest scorecard

What separates these from websites — the same doctrine as a native app:

1. **Statefulness over documents.** Long-lived state-machine clients; the server is a sync endpoint, not a page generator; the UI reacts to server changes without user action.
2. **URLs are app state.** Deep links captured at boot (`#launchRoute`) and restored exactly. No page navigation inside an app, ever.
3. **Keyboard-first, platform-aware**, down to OS-specific input bugs.
4. **Perceived performance is engineered:** the early-child iframe races the shell, immutable builds cache forever, preload chains, idle connection parking.
5. **Skeletons over spinners:** app frames stay hidden until claimed; list states drive skeletons.

**Where Apple's web apps fall short [CONFIRMED/REPORTED]** — the beatable gaps: severe load weight (3.9 MB shell + 3-8 MB per app, full shell reload on every app switch); no offline at all; dated visual chrome mid-migration (two segmented-control implementations, a SproutCore app in production, `_unsafe_` escape hatches); file UX and collaboration behind Google's. A small, disciplined web app with a flat canvas, hairline separation, real keyboard support, and honest states can beat this chrome outright. The architecture underneath it — the shell, sync tokens, webcourier, optimistic rollback — is what deserves stealing.

---

# Part 5 · The PWA stance

- **icloud.com is not a PWA** [CONFIRMED]: no manifest (404), no install path; the service worker is push-only. Apple treats its productivity suite as a classic multi-context web app.
- **music.apple.com and tv.apple.com ARE installable PWAs** [CONFIRMED]: SSR Svelte with real manifests (maskable icons, standalone scope) and dual modern/legacy Vite builds.
- **Apple's platform rules for everyone else's web apps** [CONFIRMED, WWDC23 + WebKit blog]: Web Push works on iOS only for Home-Screen-installed apps (a Safari tab cannot subscribe); no silent push, ever — every push must show a notification or the origin loses permission; VAPID against `web.push.apple.com` with a ≤24h JWT; `navigator.setAppBadge()` only in installed contexts.

---

# The copy-able recipes (distilled)

1. **Shell-of-iframes:** versioned immutable shell; on route match, synchronously create a full-bleed iframe at `/<app>/current/<locale>/index.html#launchRoute=<encoded-url>` *before* your own framework boots; postMessage-RPC bus; unread counts hoisted to a shell badge bus that redraws favicon and tiles.
2. **Auth handoff:** identity provider in its own cross-origin iframe, `response_mode=web_message` over postMessage, tokens in one IndexedDB store, exposed to app frames via a hidden bridge iframe with a timeout.
3. **Realtime:** one WebSocket per session to a push relay, long-poll fallback, idle parking; pushes are pings, clients pull deltas by cached **sync token** with a `moreComing` loop.
4. **Desktop grid:** CSS Grid + `gridArea` for placed items, absolute layers for drag previews, custom pointer handlers (not HTML5 DnD), optimistic draft state with *named* rollback actions (`undoUpdateEvent`, `saveFailed`).
5. **Semantic tokens as HSL components** (`--x-h/-s/-l`) so accent tinting and dark mode recompute live; port the UIKit names (labelPrimary…Quaternary, fillPrimary…, systemRed).
6. **Heavy document engines:** native code → WASM with a TS bridge, render to canvas, store as compressed protobuf with partial-save merge flags mapped onto record deltas.

---

# Sources

**Live probes (2026-08-26):** icloud.com shell HTML + `/system/icloud.com/2630Build35/en-us/main.js` (3,937,383 bytes) · `/applications/{calendar,mail2,notes3,photos3,find,iclouddrive,pages}/current/en-us/` bundles · Calendar main.css (398 KB) · music.apple.com + `/manifest.json` · tv.apple.com

**Apple:** developer.apple.com/documentation/cloudkitjs + CloudKit Web Services Reference + WWDC15 session 710 · WWDC23 session 10120 (Web Push for web apps) · webkit.org/blog/13878 · developer.apple.com/documentation/usernotifications/sending-web-push-notifications · jobs.apple.com postings (iCloud Web; Realtime Web Experiences; iWork for iCloud — SproutCore + WASM + Canvas/WebGL required)

**Reverse engineering:** the pyicloud project on GitHub (auth + calendar services) · github.com/GitRegret/apple-mcp-calnot (in-frame Notes/CloudKit observation) · github.com/DrHazemAli/iCloud-API-Exploits (webservices map) · github.com/zeynalnia/icloudjs · andrews.substack.com/p/reverse-engineering-iwork · oss.sheetjs.com/notes/iwa · github.com/flyfish-dev/iwork-viewer · mjtsai.com/blog/2020/04/17/iwork-10-in-webassembly · springenwerk.com/2011/08/about-apples-cloudos

**Press/history:** MacRumors, The Verge, MacStories (2022 icloud.com redesign + jiggle mode) · Ars Technica (Apple Music web) · 9to5Mac (SproutCore, 2011) · InfoWorld (iWork for iCloud review) · Hacker News threads 20891216, 25357409, 47221611

---

*Companion documents: [README.md](README.md) (the design thesis) · [LLM.md](LLM.md) (the thesis for machines) · [APPLE_NATIVE.md](APPLE_NATIVE.md) (SwiftUI components) · [APPLE_WEB.md](APPLE_WEB.md) (Apple's web sites and the native-feel ingredients). Building a desktop-class web app: paste LLM.md + APPLE_WEB.md + this file.*
