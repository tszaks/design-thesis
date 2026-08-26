# The Apple-Native SwiftUI Catalog

An LLM-optimized catalog of the components, behaviors, and quirks that make an iOS app feel like Apple built it. Every entry answers three questions: **EXISTS** (what it's called, what it looks like), **USE WHEN** (the moment it's for, where Apple's own apps use it), **BUILD** (the recipe, minimum OS, gotchas). Scan the USE WHEN lines against the screen you're building before inventing anything custom: there is almost always a native thing for it.

Verified against Apple documentation, WWDC sessions (23/24/25), and shipping code in two production apps (Vero, Payday). Compiled 2026-08.

## Cross-cutting facts that save the most pain

1. **Spring presets are back-deployed to iOS 13.** `.snappy`, `.smooth`, `.bouncy`, and `spring(duration:bounce:)` are `@backDeployed`; do not gate them at 17. Only the `Spring` physics struct is 17+.
2. **`.wiggle`, `.breathe`, `.rotate` symbol effects are iOS 18** (SF Symbols 6), not 26, despite what 2026 blogs claim. iOS 26 added `.drawOn/.drawOff`.
3. **`contentTransition` of any kind does nothing without a driving animation** (`withAnimation` or `.animation(_:value:)`). Silent no-op otherwise.
4. **Scroll snap/position APIs silently no-op without `scrollTargetLayout()`** on the lazy stack (not the ScrollView).
5. **`RoundedRectangle` defaults to `.circular` corners.** The Apple squircle requires `style: .continuous`, always.
6. **On iOS 26, bar-tinting code fights the system.** Gate `toolbarBackground`, opaque tab-bar appearances, and hand-rolled glass to `#unavailable(iOS 26)`.
7. **Grouped backgrounds must be hidden before custom backgrounds work:** `.scrollContentBackground(.hidden)` first, then `.background`.
8. **Pinned/sticky headers ship with transparent backgrounds.** Always add `.background(.bar)` or a material.
9. **Scroll-effect closures (`scrollTransition`, `visualEffect`) run on the render server.** Visual modifiers only, no state writes, no layout changes.
10. **Exits are faster than entrances, nothing enters from scale(0), and every animation respects Reduce Motion** with a designed crossfade fallback.

## Availability ladder (quick lookup)

| Floor | What lives there |
|---|---|
| 13–14 | springs (back-deployed), transitions, semantic fonts, redacted, @ScaledMetric, Menu, fullScreenCover, GroupBox, DisclosureGroup, quickLookPreview, statusBarHidden, compositingGroup, trim |
| 15 | searchable, swipeActions, refreshable, safeAreaInset, materials, FocusState/submitLabel/keyboard toolbar, confirmationDialog, interactiveDismissDisabled, bordered buttons, symbolRenderingMode/Variant, monospacedDigit, badge, headerProminence, textSelection, privacySensitive, contentShape kinds, Menu primaryAction, foregroundStyle multi |
| 16 | NavigationStack/SplitView, presentationDetents, toolbarBackground, Charts, Grid, ViewThatFits, LabeledContent, scrollContentBackground, scrollDismissesKeyboard, numericText(countsDown:), ShareLink, photosPicker, requestReview, Gauge, MultiDatePicker, AnyLayout/Layout, ImageRenderer, Color.gradient, shadow ShapeStyle, toolbarTitleMenu, navigationDocument, persistentSystemOverlays, scrollDisabled, menuOrder, contextMenu preview, fontWidth, textScale via 17? (see entry), findNavigator, defersSystemGestures, onGeometryChange (back-deploys here) |
| 16.4 | presentationBackground/BackgroundInteraction/CornerRadius/ContentInteraction/CompactAdaptation, detent selection binding, menuActionDismissBehavior, ControlGroup-in-Menu, UnevenRoundedRectangle + .rect(cornerRadii:), scrollBounceBehavior |
| 17 | scrollTransition, visualEffect, scrollPosition(id:), scrollTargetBehavior/Layout, contentMargins, safeAreaPadding, defaultScrollAnchor, scrollClipDisabled, containerRelativeFrame, ContentUnavailableView, sensoryFeedback, symbolEffect + .replace, numericText(value:), phaseAnimator/keyframeAnimator, TipKit, chart scrolling/selection, inspector, buttonRepeatBehavior, .circle border shape, defaultFocus, geometryGroup, springLoadingBehavior, palette pickers, badgeProminence, listSectionSpacing, toolbar(removing:), toolbarTitleDisplayMode, textScale, typesettingLanguage, allowedDynamicRange, onKeyPress, scoped animation(_:body:), CustomAnimation, containerBackground(for: .widget), SubscriptionStoreView, DragGesture velocity, blurReplace, .extraLarge controls, scrollIndicatorsFlash |
| 18 | Tab(value:role:)/.search role, sidebarAdaptable, tabViewCustomization, navigationTransition(.zoom) + matchedTransitionSource, onScrollGeometryChange/onScrollPhaseChange, ScrollPosition struct, onScrollVisibilityChange, presentationSizing, toolbarVisibility, TextRenderer (usable), wiggle/breathe/rotate + Magic Replace, MeshGradient, Color.mix, onModifierKeysChanged, ContactAccessButton, @Entry/@Previewable (Xcode 16) |
| 26 | glassEffect family, GlassEffectContainer, glassEffectID/Union/Transition, .glass/.glassProminent buttons, backgroundExtensionEffect, scrollEdgeEffectStyle/Hidden, safeAreaBar, ToolbarSpacer, sharedBackgroundVisibility, tabBarMinimizeBehavior, tabViewBottomAccessory, searchToolbarBehavior(.minimize), DefaultToolbarItem(.search), navigationSubtitle (iOS), sectionIndexLabel, sectionActions, listSectionMargins, ConcentricRectangle, scrollInputBehavior, Slider ticks, WebView/WebPage, @Animatable macro, ScrollGeometry.isScrolledToTop/Bottom |

---

# Part 1 · Navigation and chrome

### Large title collapse — NavigationStack + navigationTitle
**EXISTS:** The signature iOS settle: big left-aligned bold title living in the content, compressing into a small centered bar title on scroll. Nothing says native faster; every cross-platform fake gets the timing wrong.
**USE WHEN:** Every root screen of a section. Apple: Settings, Mail, Music, Health. Detail screens go `.inline`.
**BUILD:**
```swift
NavigationStack {
    List(items) { NavigationLink($0.title, value: $0) }
        .navigationTitle("Library")
        .navigationBarTitleDisplayMode(.large)
        .navigationDestination(for: Item.self) { DetailView(item: $0).navigationBarTitleDisplayMode(.inline) }
}
```
iOS 16 (title APIs 14). iOS 17 adds `.toolbarTitleDisplayMode(.inlineLarge)`: large text pinned inline, never collapses (App Store Today).
**GOTCHAS:** Collapse requires the scrollable view to be the stack's direct content (a wrapping ZStack/GeometryReader kills it). A sheet presented at first appear can freeze the title collapsed. Put `navigationTitle` on the content, never on the NavigationStack itself. Large→inline pushes can jump; keep modes consistent per level.

### iOS 26 scroll edge effect (content frosting)
**EXISTS:** Content itself progressively blurs/dims as it slides under bars: a pocket of frost, no bar slab. The core legibility mechanism of Liquid Glass. Automatic, zero code.
**USE WHEN:** Everywhere content underlaps chrome. `.soft` = diffuse (default feel), `.hard` = near-opaque cut for dense control clusters.
**BUILD:** `.scrollEdgeEffectStyle(.soft, for: .top)`; hide with `scrollEdgeEffectHidden(_:for:)` (beta blogs show `scrollEdgeEffectDisabled()`, the shipping name is Hidden). iOS 26.
**GOTCHAS — the key fact:** The effect only exists where a real BAR exists (nav/tab/toolbar or `safeAreaBar`). `overlay(alignment: .bottom)` gets nothing. For custom floating chrome use `.safeAreaBar(edge: .bottom) { CustomBar() }` (iOS 26) or a toolbar placement (which also glasses it automatically — don't add your own glassEffect on top). A bar-less screen (`navigationTitle("")` + custom in-content header) gets NO effect: hand-roll a top fade there (see House Patterns). Pre-26 fallback for a soft bottom edge: `.ultraThinMaterial` masked by a ~30pt clear→black gradient.

### Toolbar semantics + ToolbarSpacer
**EXISTS:** Semantic placements render Cancel top-left, bold Done top-right, correct on every platform, with keyboard Return/Esc wired on Mac. `ToolbarSpacer` (26) splits items into separate floating glass pods.
**USE WHEN:** Every sheet toolbar and every screen action. Apple: all stock edit sheets.
**BUILD:**
```swift
.toolbar {
    ToolbarItem(placement: .cancellationAction) { Button("Cancel") { dismiss() } }
    ToolbarItem(placement: .confirmationAction) { Button("Done") { save() }.disabled(!isValid) }
    ToolbarItem(placement: .primaryAction) { Button("Share", systemImage: "square.and.arrow.up") { } }
}
// iOS 26 pods: ToolbarSpacer(.flexible) / (.fixed) between ToolbarItems breaks the shared glass capsule
```
Semantics iOS 15-era; ToolbarSpacer 26 (pre-26: `Spacer()` in a ToolbarItemGroup).
**GOTCHAS:** `toolbarBackground(_:for:)` still hides until content scrolls beneath unless you also pass `.visible`. On iOS 26 don't tint bars at all. `.confirmationAction` doesn't auto-disable; pair with `.disabled`. Remove one item from the glass pod: `.sharedBackgroundVisibility(.hidden)` (26).

### Tab bar that minimizes on scroll + the search tab
**EXISTS:** iOS 26 floating glass tab bar shrinks to the selected icon on scroll down, re-inflates on scroll up. `Tab(role: .search)` pins a separate circular search tab trailing. `tabViewBottomAccessory` is Music's mini-player bar as an API.
**USE WHEN:** Browse-apps. Apple: TV, Music, App Store, Podcasts.
**BUILD:**
```swift
TabView(selection: $selection) {
    Tab("Home", systemImage: "house", value: Tabs.home) { HomeView() }     // needs scrollable content to minimize
    Tab(value: Tabs.search, role: .search) { SearchView() }               // label+magnifier inferred, pinned
}
.tabBarMinimizeBehavior(.onScrollDown)      // iOS 26, iPhone only
.tabViewBottomAccessory { MiniPlayerView() } // read @Environment(\.tabViewBottomAccessoryPlacement); render compact when .inline
```
`Tab`/roles iOS 18; minimize/accessory iOS 26.
**GOTCHAS:** One `.search` tab max (a second = undefined behavior). Minimize only fires when the selected tab's content scrolls. iPad ignores minimize (bar is top/sidebar). >5 tabs still spills to More. The pinned slot can't be repurposed as an Add button — but selecting a role-search tab CAN open a sheet and snap back (see House Patterns: action tab).

### searchable + scopes + iOS 26 bottom placement
**EXISTS:** One modifier, system-owned placement: iOS 26 puts it in a thumb-reachable bottom glass capsule on iPhone when attached to the NavigationStack. Scopes give Mail's All/Sender/Subject segmented row. `.minimize` collapses to a lone magnifier.
**USE WHEN:** Any searchable content. Apple: Mail (scopes), Messages (bottom field), Files.
**BUILD:**
```swift
NavigationStack { ContentList() }
    .searchable(text: $query, prompt: "Search notes")      // attach to the STACK for iOS 26 bottom placement
    .searchScopes($scope) { Text("All").tag(Scope.all); Text("Sender").tag(Scope.sender) }  // iOS 16
// iOS 26 minimized: .searchable(..., placement: .toolbar).searchToolbarBehavior(.minimize)
```
searchable 15; scopes 16; minimize 26.
**GOTCHAS:** Attachment point decides placement (inside content = old drawer). `isSearching`/`dismissSearch` environment only work from SUBVIEWS of the searchable view. Scope resets on dismissal with `.onSearchPresentation` activation.

### NavigationSplitView (sidebar apps)
**EXISTS:** Mail/Notes skeleton: sidebar → list → detail, collapsing to a NavigationStack on iPhone from the same code.
**USE WHEN:** iPad/Mac-serious apps.
**BUILD:**
```swift
NavigationSplitView(columnVisibility: $vis) {
    List(folders, selection: $folder) { Text($0.name) }      // selection-based Lists drive columns
        .navigationSplitViewColumnWidth(min: 220, ideal: 260, max: 320)
} detail: {
    if let note { NoteEditor(note) } else { ContentUnavailableView("Select a Note", systemImage: "note.text") }
}
.navigationSplitViewStyle(.balanced)
```
iOS 16.
**GOTCHAS:** Sidebar selection highlight needs `List(_, selection:)`, not just NavigationLinks. Push-within-detail = NavigationStack in the detail column only (stacks in the sidebar break iPhone collapse). Always give detail an empty state — it's visible before selection on iPad.

### Zoom transition — navigationTransition(.zoom)
**EXISTS:** The Photos-grid zoom: the tapped cell IS the detail, continuously scaling, interactive dismiss included (pinch/drag it back into its cell). Crosses presentation boundaries, which matchedGeometryEffect cannot.
**USE WHEN:** Card/thumbnail → detail. Apple: Photos, App Store Today, Music album art.
**BUILD:**
```swift
@Namespace private var ns
// source cell:
ItemCell(item).matchedTransitionSource(id: item.id, in: ns)
// destination ROOT (push, sheet, or fullScreenCover all work):
DetailView(item).navigationTransition(.zoom(sourceID: item.id, in: ns))
```
iOS 18. Fallback: plain push. Never fake it with matchedGeometryEffect across a sheet — it can't cross presentation roots.
**GOTCHAS:** Modifier goes on the destination's root, outside containers. Stable unique sourceIDs (a constant id makes every cell zoom from cell one). Scrolled-away source degrades the dismiss to a slide. Detail-owned drag gestures can fight the interactive dismiss.

### Menus done natively
**EXISTS:** System pull-down (glass, checkmarks, red destructive rows) + long-press context menus with a lifted preview card + `ControlGroup` icon rows. Nobody can rebuild the physics/haptics of real UIMenu.
**USE WHEN:** Sort/filter/actions. Apple: Files, Mail, Safari, Photos.
**BUILD:**
```swift
Menu {
    ControlGroup { Button("Cut", systemImage: "scissors"){}; Button("Copy", systemImage: "doc.on.doc"){} } // 16.4 in menus
    Picker("Sort by", selection: $sort) { ... }.pickerStyle(.menu)   // checkmarked rows
    Divider()
    Button("Delete", systemImage: "trash", role: .destructive) { }
} label: { Label("Options", systemImage: "ellipsis.circle") }
  primaryAction: { performDefault() }   // tap = default, long-press = menu (iOS 15)

Cell().contextMenu { buttons } preview: { PreviewCard().frame(width: 300) }   // iOS 16
Menu("Text size") { steppers }.menuActionDismissBehavior(.disabled)            // keep-open, 16.4, not macOS
```
**GOTCHAS:** Menu content is system-rendered: custom views/colors/fonts inside `Menu { }` are stripped (only Button/Toggle/Picker/Divider/Section/Menu/ControlGroup/Label survive). Don't add `.foregroundColor(.red)` on destructive-role rows (double-styles). Custom preview must be sized/clipped yourself or the open jumps. `menuOrder(.fixed)` stops iOS reversing item order by screen position.

---

# Part 2 · Presentation

### Sheets with detents — the full kit
**EXISTS:** The half-sheet grammar of Maps/Find My: snaps between heights with a grabber, and can leave the background alive.
**USE WHEN:** Contextual panels over persistent context.
**BUILD:**
```swift
.sheet(isPresented: $show) {
    SettingsView()
        .presentationDetents([.height(120), .medium, .large], selection: $detent)  // selection = programmatic snap
        .presentationDragIndicator(.visible)
        .presentationBackground(.ultraThinMaterial)                                // 16.4
        .presentationBackgroundInteraction(.enabled(upThrough: .height(120)))      // 16.4: touch the map behind
        .presentationCornerRadius(24)                                              // 16.4
        .presentationContentInteraction(.scrolls)                                  // 16.4: inner scroll beats detent-resize
        .interactiveDismissDisabled(hasUnsavedChanges)                             // 15
}
```
Detents/grabber 16; the 16.4 family as marked. Custom detent: `CustomPresentationDetent` with `height(in context:)`.
**GOTCHAS:** Default swipe-up grows the sheet before inner scrolling — users read it as broken; `.presentationContentInteraction(.scrolls)` fixes it. `upThrough:` detent must be in the detents set. Detent modifiers go on the sheet CONTENT. `.medium` + keyboard = sheet grows; test fields at partial detents. iOS 26 sheets get glass + inset shape automatically and can morph out of their source button; heavy custom backgrounds opt out of the system look. sheet = dismissible card; fullScreenCover (14) = modal mode (onboarding, camera). If you're using `interactiveDismissDisabled` + only `.large`, you want fullScreenCover.

### Popovers on iPhone, dialogs, inspector
**EXISTS:** `presentationCompactAdaptation(.popover)` (16.4) = real arrow-balloon popovers on iPhone. `confirmationDialog` (15) = the bottom action sheet ("ways to proceed with what you just did"), auto-Cancel, red destructive. `alert` = rare centered interruption. `inspector` (17) = Freeform's trailing details column, sheet on iPhone.
**BUILD:**
```swift
.popover(isPresented: $info, arrowEdge: .top) { InfoView().presentationCompactAdaptation(.popover).frame(minWidth: 260, minHeight: 160) }

.confirmationDialog("Delete \(item.name)?", isPresented: $confirm, titleVisibility: .visible) {
    Button("Delete", role: .destructive) { delete() }
} message: { Text("This can't be undone.") }

Canvas().inspector(isPresented: $show) { StyleInspector().inspectorColumnWidth(min: 250, ideal: 300).presentationDetents([.medium, .large]) }
```
**GOTCHAS:** Dialog titles are HIDDEN by default on iOS — pass `titleVisibility: .visible` or users see context-free buttons. Button order is system-controlled. iPhone-adapted popovers need explicit frames or size flakily. Semantics: dialog = user-initiated choice; alert = app-initiated or destructive-confirm, 2-3 buttons max, buttons ignore custom styling (role drives red/bold). Prefer an alert over confirmationDialog when it's anchored to a toolbar button and every exit must be visible (popover-dialogs hide Cancel behind tap-outside).

---

# Part 3 · Liquid Glass and materials

### glassEffect and friends (iOS 26)
**EXISTS:** Real-time lensing material that refracts content behind it and flexes on touch. Custom glass controls join the same optical layer as system bars — that co-membership is what reads "built for iOS 26."
**USE WHEN:** Chrome ONLY: floating buttons, capsules, custom bars. Content (cards, rows, lists) stays solid. If everything is glass, nothing is.
**BUILD:**
```swift
Text("Hello").padding()                       // padding BEFORE glass or the shape won't wrap it
    .glassEffect()                            // .regular in a capsule (default)
    .glassEffect(.regular.tint(.orange).interactive(), in: .rect(cornerRadius: 16))

Button("Confirm") { }.buttonStyle(.glassProminent)   // the ONE primary action
Button("Action") { }.buttonStyle(.glass)

// Morphing between glass elements:
GlassEffectContainer(spacing: 40) {
    HStack(spacing: 40) {
        icon1.glassEffect().glassEffectID("a", in: ns)
        if expanded { icon2.glassEffect().glassEffectID("b", in: ns).glassEffectTransition(.matchedGeometry) }
    }
}
// Fuse several views into one glass shape: .glassEffectUnion(id:namespace:)
// Content extending under floating sidebars: hero.backgroundExtensionEffect()
```
Fallback (canonical):
```swift
if #available(iOS 26, *) { content.glassEffect(.regular, in: .rect(cornerRadius: 16)) }
else { content.background(.ultraThinMaterial, in: RoundedRectangle(cornerRadius: 16, style: .continuous))
              .overlay(RoundedRectangle(cornerRadius: 16, style: .continuous).stroke(.primary.opacity(0.15), lineWidth: 1)) }
// .glassProminent → .borderedProminent; .glass → .bordered
```
**GOTCHAS:** Never glass-on-glass (no glassEffect on toolbar/tab items — they're glassed automatically; double glass is a visible artifact). `.clear` variant needs a dimming layer under it over busy content. Batch related effects into ONE container (each loose effect is a render pass). Morphing needs container spacing ≥ the travel distance or `.matchedGeometry` degrades to a fade. `.interactive()` on non-tappable decoration violates HIG and costs GPU. Route ALL glass through one shared wrapper per app and lint raw call sites (see House Patterns).

### Materials (pre-26 and in-content forever)
**EXISTS:** The frosted ladder: `.ultraThinMaterial → .ultraThickMaterial`, plus `.bar`. Colorless blur recipes; system handles light/dark. Content over them gets automatic vibrancy IF it uses hierarchical styles.
**USE WHEN:** Pre-26 chrome fallback, and in-content surfaces (media scrims, overlay cards) even on 26 — glass is for the layer above content, materials for surfaces within it.
**BUILD:** `.background(.regularMaterial, in: RoundedRectangle(cornerRadius: 12, style: .continuous))` + `.foregroundStyle(.secondary)` on contents (ShapeStyle `.secondary` vibrates; concrete `Color.gray` doesn't). iOS 15.
**GOTCHAS:** Materials can't be tinted (color-over-material = mud). Over a flat background a material is just gray — blur needs rich content to earn its cost. In sheets use `presentationBackground(.ultraThinMaterial)`, not `.background`.

---

# Part 4 · Scroll behavior

### scrollTransition + visualEffect (rows materialize)
**EXISTS:** Rows fade/scale/blur by scroll position, gesture-tracked and interruptible, computed off the layout tree. `visualEffect` is the continuous cousin (parallax, position-driven effects) with a GeometryProxy and no GeometryReader.
**USE WHEN:** Editorial card lists, carousels, lyric-style emphasis. Apple: Photos Memories, App Store cards, Music lyrics.
**BUILD:**
```swift
ItemCard(item).scrollTransition(axis: .vertical) { content, phase in
    content.opacity(phase.isIdentity ? 1 : 0.3)
           .scaleEffect(phase.isIdentity ? 1 : 0.92)
}   // phase.value: -1/0/+1 interpolating; thresholds: .interactive.threshold(.centered)

Card().visualEffect { content, proxy in content.offset(y: proxy.frame(in: .scrollView).minY * 0.05) }
```
iOS 17. Pre-17: per-row GeometryReader (slow) or skip — it's decorative.
**GOTCHAS:** Visual-only modifiers inside the closure (no frame/padding). Never copy phase into @State. Hit-testing follows the untransformed frame (by design).

### Snapping carousels — scrollTargetBehavior + scrollPosition
**EXISTS:** Native card-snap with correct deceleration physics, plus a two-way binding for "which card is current."
**USE WHEN:** Card shelves, pagers. Apple: App Store Today, Weather hourly.
**BUILD:**
```swift
ScrollView(.horizontal) {
    LazyHStack(spacing: 12) {
        ForEach(items) { ItemCard($0).containerRelativeFrame(.horizontal, count: 1, spacing: 12) }
    }
    .scrollTargetLayout()                       // REQUIRED, on the stack
}
.scrollTargetBehavior(.viewAligned)             // .viewAligned(limitBehavior: .always) = one card per swipe
.scrollPosition(id: $scrolledID)                // withAnimation { scrolledID = x } to page programmatically
.contentMargins(.horizontal, 20, for: .scrollContent)  // peeking edges
.scrollIndicators(.hidden)
```
iOS 17. iOS 18: `ScrollPosition` struct binding (`position.scrollTo(edge: .top)`, offsets, ids) replaces ScrollViewReader. Pre-17: `TabView.page` style.
**GOTCHAS:** No `scrollTargetLayout` = binding never updates (the #1 failure). Binding reports the LEADING visible id. `.paging` pages by container width and drifts with contentMargins — peeking layouts want `.viewAligned` + containerRelativeFrame. Center-snap needs `anchor:`.

### Custom collapsing/stretchy headers — onScrollGeometryChange
**EXISTS:** Scroll-linked chrome at 120Hz without the GeometryReader+PreferenceKey tax. `onScrollPhaseChange` gives the gesture state machine (.idle/.tracking/.interacting/.decelerating/.animating).
**USE WHEN:** Stretchy heroes, collapse-on-scroll bars, hide-keyboard-on-drag, commit-analytics-on-settle.
**BUILD:**
```swift
ScrollView { Image(.hero).resizable().scaledToFill().frame(height: 300 + max(0, overscroll)).offset(y: -max(0, overscroll)); content }
    .onScrollGeometryChange(for: CGFloat.self) { geo in
        -(geo.contentOffset.y + geo.contentInsets.top)      // ALWAYS add insets back
    } action: { _, new in overscroll = new }

.onScrollPhaseChange { _, new in if new == .interacting { hideBar = true }; if new == .idle { hideBar = false } }
```
iOS 18. (Sibling `onGeometryChange` back-deploys to 16.) Pre-18 fallback: GeometryReader + PreferenceKey + named coordinateSpace.
**GOTCHAS:** Return a tiny derived value (CGFloat/Bool), never the whole geometry — the action fires only when the transformed value changes. `contentOffset.y` at rest is `-contentInsets.top`, not 0. State writes that change content height cause feedback loops; clamp. Works on List and TextEditor too.

### Clearing floating bars — the decision table
**EXISTS:** Three different tools for "content scrolls under the bar but starts/ends clear of it."
| API | What | Use when |
|---|---|---|
| `safeAreaInset(edge:)` (15) | puts a real VIEW in the safe area; content scrolls under it | you own the bar on this screen (mini-player, bottom CTA) |
| `safeAreaPadding` (17) | same math, empty space | the chrome is drawn elsewhere; you just need the height respected |
| `contentMargins(_,for:)` (17) | insets scroll content and/or indicators independently | breathing room, carousel peeks, indicator-only insets |
**BUILD:** `ScrollView{...}.safeAreaInset(edge: .bottom) { MiniPlayer().background(.ultraThinMaterial) }` · `.contentMargins(.horizontal, 24, for: .scrollContent)` · chat pinning: `.defaultScrollAnchor(.bottom)` (17; iOS 18 adds `for: .sizeChanges` etc. roles).
**GOTCHAS:** `.padding` on scroll content kills the under-scroll — the tell-tale amateur artifact. `contentMargins` cascades to every scrollable below it; scope tightly. An opaque safeAreaInset bar hides the under-scroll anyway — use `.bar`/material. safeAreaInset bars ride the keyboard (keyboard is safe area); `.ignoresSafeArea(.keyboard)` to opt out. In practice: ~88pt clearance above a floating glass button, ~112pt above the iOS 26 tab bar.

### Sticky headers + shadow bleed
**EXISTS:** `LazyVStack(pinnedViews: [.sectionHeaders])` = Contacts-style pin-and-push headers (14). `scrollClipDisabled()` (17) lets card shadows/artwork escape the scroll viewport.
**BUILD:** header views need `.background(.bar)` — they're transparent by default. Shelf: `ScrollView(.horizontal){...}.scrollClipDisabled()` then re-clip wider with `.clipShape(Rectangle())` if it overdraws neighbors.
**GOTCHAS:** ScrollViewReader.scrollTo overshoots pinned headers (FB13447486); iOS 18 ScrollPosition behaves better. Headers pin to the frame top, not the safe area.

---

# Part 5 · Lists, forms, and the five states

### List — the highest-leverage native component
**EXISTS:** Two decades of UITableView physics free: full-swipe actions with haptic commit, rubber-banded pull-to-refresh, edit-mode reorder.
**BUILD:**
```swift
List {
    ForEach(emails) { email in
        EmailRow(email)
            .swipeActions(edge: .trailing, allowsFullSwipe: true) {
                Button(role: .destructive) { delete(email) } label: { Label("Delete", systemImage: "trash") }
                Button { flag(email) } label: { Label("Flag", systemImage: "flag") }.tint(.orange)
            }
            .swipeActions(edge: .leading) { Button { toggleRead(email) } label: { Label("Read", systemImage: "envelope.open") }.tint(.blue) }
    }
    .onDelete { ... }   // edit-mode delete
    .onMove { ... }     // reorder grips
}
.listStyle(.plain)          // dense, full-bleed, headers pin (Contacts). .insetGrouped = floating groups (Settings)
.refreshable { await reload() }   // async; spinner stays until return — don't fire-and-forget
.toolbar { EditButton() }
```
Custom-card-in-List reset trio: `.listRowInsets(EdgeInsets())` + `.listRowSeparator(.hidden)` + `.listRowBackground(Color.clear)`. Full-width separators: `.alignmentGuide(.listRowSeparatorLeading) { _ in 0 }`. A-Z scrubber: `sectionIndexLabel` + `listSectionIndexVisibility(.visible)` (26; pre-26 hand-roll a letter rail).
**GOTCHAS:** ANY custom swipeActions removes the automatic swipe-delete — re-add it. Full swipe triggers the FIRST listed button. Multiple buttons in one row all fire on row tap unless each gets `.buttonStyle(.borderless)`. Swipe buttons render icon-only in most contexts. Don't use `role: .destructive` just for red — use `.tint(.red)`.

### Form vs List + LabeledContent
**EXISTS:** `Form` = Settings.app: insetGrouped visuals PLUS control-aware adaptation (pickers become menu rows, toggles right-align). `LabeledContent` = the label-left/gray-value-right row with free accessibility pairing.
**BUILD:**
```swift
Form {
    Section {
        LabeledContent("Plan", value: "Pro")
        Toggle("Notifications", isOn: $on)
        Picker("Frequency", selection: $freq) { ... }
    } footer: { Text("Notifications include weekly summaries.") }   // the Settings place for explanations
}
.scrollContentBackground(.hidden)   // REQUIRED before .background works
.background(customCanvas)
```
Form 13; LabeledContent/scrollContentBackground 16. Rule: user configures → Form; user browses data → List. Stop auto-uppercased headers: `.textCase(nil)` on the Section.

### The five states (loading, content, empty, error, offline)
**EXISTS:** `ContentUnavailableView` (17) = the system empty state (big secondary symbol, title2-bold, footnote description). `.search(text:)` variant = the exact "No Results for 'x'". `redacted(reason: .placeholder)` (14) = skeletons: real layout, ghosted content, zero shift when data lands.
**BUILD:**
```swift
List(items) { ... }.overlay {
    if items.isEmpty && !searchText.isEmpty { ContentUnavailableView.search(text: searchText) }
    else if items.isEmpty { ContentUnavailableView { Label("No Trips", systemImage: "airplane.circle") }
        description: { Text("Trips you plan will appear here.") }
        actions: { Button("Plan a Trip") { plan() }.buttonStyle(.borderedProminent) } }
}

UserCard(user: user ?? .placeholder)                 // dummy data sized like real data
    .redacted(reason: user == nil ? .placeholder : [])
    .disabled(user == nil)                           // redaction does NOT disable interaction
```
**GOTCHAS:** Present via `.overlay`, never as a List row. Feed real-shaped placeholder data — an empty string redacts to nothing. `unredacted()` exempts static headers. VoiceOver reads redacted dummy text; add `.accessibilityLabel("Loading")`. First-run empty ≠ filtered-empty (the latter names the recovery). No shimmer ships; Apple mostly uses static redaction (a `phaseAnimator` opacity pulse 0.4→0.8 is closest to their feel). Inside a ScrollView it collapses to intrinsic height — `containerRelativeFrame(.vertical)` fixes centering. Note: a documented counter-case exists — inside a ScrollView above a floating glass tab bar its intrinsic height can sit it behind the bar; a single quiet line of secondary text is the correct downgrade (House Patterns).

---

# Part 6 · Text and numbers

### Rolling numbers — the money recipe
**EXISTS:** The odometer roll of Clock/Fitness/Stocks: unchanged digits hold still, changed digits slide with blur-fade, direction matching semantic up/down.
**USE WHEN:** Any live-updating number. Money always.
**BUILD (the full canonical stack):**
```swift
Text(amount, format: .currency(code: "USD"))
    .font(.system(size: 48, weight: .heavy, design: .rounded))   // money = rounded heavy, fixed sizes (Apple Wallet)
    .monospacedDigit()                                            // no width jitter
    .contentTransition(.numericText(value: amount))               // 17; direction from delta. countsDown: for timers (16)
    .animation(.snappy, value: amount)                            // REQUIRED or nothing rolls
    .lineLimit(1).minimumScaleFactor(0.5)
```
**GOTCHAS:** No animation = silent crossfade. Wrong `countsDown:` rolls the wrong direction. 99→100 reflows width: stable `frame(minWidth:)`. Reduce Motion: numericText auto-degrades to fade — but branch anyway for the animation itself. `.contentTransition(.interpolate)` morphs weight/color changes on text — hidden gem. Self-updating `Text(timerInterval:)` swaps digits (it's outside the animation system); drive the string from a per-second `TimelineView` instead so numericText can roll it.

### Dynamic Type contract
**EXISTS:** Text that honors the user's size is the strongest native-citizen signal; semantic styles also buy optical size/leading/weight adjustments.
**BUILD:** semantic styles always (`.font(.headline)`), custom fonts via `.font(.custom("X", size: 17, relativeTo: .headline))`; scale metrics with `@ScaledMetric(relativeTo: .body) var iconSize = 24.0`; reflow with `ViewThatFits { HStack{...}; VStack{...} }` (16); branch at `dynamicTypeSize.isAccessibilitySize`; shrink only as a last resort (`.lineLimit(1).minimumScaleFactor(0.6)`); clamp sparingly `.dynamicTypeSize(...DynamicTypeSize.accessibility2)`.
**GOTCHAS:** ViewThatFits candidates must have honest intrinsic sizes (a greedy `.frame(maxWidth: .infinity)` child "fits" everywhere and short-circuits). Don't @ScaledMetric hairlines. Test at AX5.

### Glyph-level text — TextRenderer
**EXISTS:** iOS 18 per-glyph drawing (Text.Layout → lines → runs → slices in a GraphicsContext): staggered reveals, glows, karaoke, the AI-shimmer class of effects.
**BUILD:** conform to `TextRenderer`, make `progress` the `animatableData`, per-slice `copy.opacity/translateBy/addFilter(.blur)` staggered by index, apply with `.textRenderer(MyRenderer(progress:))`, animate progress. Target text ranges with `TextAttribute`.
**GOTCHAS:** You must draw every slice or text vanishes. Disables selection; display text only. Check reduce-motion yourself. Protocol is stamped 17+ but treat the feature as 18+.

---

# Part 7 · Motion

### Springs — the only sanctioned curve for interactive UI
**EXISTS:** Since iOS 17, bare `withAnimation { }` IS a spring. Three personalities: `.smooth` (no bounce — system chrome), `.snappy` (~0.15 — controls that track fingers), `.bouncy` (~0.3 — playful/physical moments only).
**BUILD:** `withAnimation(.snappy) { ... }` · tune: `.spring(duration: 0.4, bounce: 0.2)` (duration = perceptual settle; bounce -1...1). Completion: `withAnimation(.smooth) { } completion: { }` — never asyncAfter off a spring's "duration."
**GOTCHAS:** Back-deployed to iOS 13 — don't gate at 17. Springs preserve velocity when replaced mid-flight on the same property; easeInOut restarts dead — that's the whole argument against easing curves on anything interactive. Bounce > 0.4 reads cartoonish. House pattern: ONE spring per interaction class, named tokens, never inline literals (see House Patterns: animation tokens).

### Transitions
**EXISTS:** Enter/exit vocabulary. Apple's style: elements arrive from a direction that explains where they came from; nothing hard-pops.
**BUILD:** `.transition(.move(edge: .top).combined(with: .opacity))` (the workhorse) · `.asymmetric(insertion: ..., removal: .opacity)` (exits simpler and faster) · `.scale(scale: 0.9, anchor: .topTrailing).combined(with: .opacity)` (context-menu feel) · `.blurReplace` (17, in-place content swap).
**GOTCHAS:** Bare `.scale` = scale from 0 at center: always wrong (illegible mid-flight, no spatial meaning); use 0.85–0.97 + opacity + a meaningful anchor. Transitions only run inside an animation. Identity changes (`.id()`) force remove+insert — use deliberately or suffer accidentally. Crossfading siblings need a ZStack. `.blurReplace` is for swaps in place, not navigation.

### phaseAnimator / keyframeAnimator
**EXISTS:** Multi-step choreography without state machines. Phase = discrete steps; keyframes = independent per-property tracks (scale peaks while rotation mid-swings).
**BUILD:** shake: `.phaseAnimator([0,-12,12,-8,8,0], trigger: attempts) { $0.offset(x: $1) } animation: { _ in .spring(duration: 0.08, bounce: 0.4) }` · celebrate: `keyframeAnimator(initialValue:trigger:)` with `KeyframeTrack(\.scale) { SpringKeyframe(1.3, duration: 0.2, spring: .bouncy); SpringKeyframe(1.0, ...) }`.
iOS 17. Pre-17 shake: `AnimatableModifier` with `sin(animatableData * .pi * shakes)`.
**GOTCHAS:** Trigger-less phaseAnimator loops forever — gate on reduce motion. The keyframe content closure runs every frame: transforms/opacity only. Keyframes restart on re-trigger (not velocity-preserving) — never for gesture-driven UI.

### matchedGeometryEffect (within one screen)
**EXISTS:** One element morphs between two layouts in the same hierarchy.
**BUILD:** same `id` + `@Namespace` on both branches of an if/else in a ZStack; drive with a spring.
**GOTCHAS:** Cannot cross NavigationStack pushes or sheets (use zoom transition instead). One `isSource: true` per id. Text morphs badly (frames interpolate, font metrics don't) — crossfade text separately. Match corner radii on both ends or they pop.

---

# Part 8 · Symbols, haptics, celebration

### SF Symbol animation
**EXISTS:** Icons that acknowledge state. The cheapest native-feel upgrade in the catalog.
**BUILD:**
```swift
Image(systemName: "bell.fill").symbolEffect(.bounce, value: count)                       // once per change (17)
Image(systemName: "wifi").symbolEffect(.variableColor.iterative, isActive: isConnecting) // while active (17)
Image(systemName: "bell.circle").symbolEffect(.wiggle, options: .repeat(.periodic(delay: 2)))  // 18
Image(systemName: isPlaying ? "pause.fill" : "play.fill").contentTransition(.symbolEffect(.replace)) // morph (17); 18: .replace.magic(fallback: .downUp)
Image(systemName: "cloud.sun.rain.fill").symbolRenderingMode(.hierarchical).foregroundStyle(.blue)   // 15; Apple's default depth look
Image(systemName: "heart").symbolVariant(isFav ? .fill : .none)                          // semantic variants beat hardcoded ".fill" names
```
**GOTCHAS:** Replace transition needs a driving animation. variableColor no-ops on symbols without variable layers. `.iterative` = searching, `.cumulative` = progress. Indefinite effects across a whole List cost real CPU — one status indicator at a time. Symbol names have their own OS floors (check the SF Symbols app).

### Haptics — sensoryFeedback
**EXISTS:** The full catalog: `.success/.warning/.error` (outcomes), `.selection` (value changes), `.increase/.decrease` (threshold steps), `.start/.stop`, `.alignment`, `.levelChange`, `.impact(weight:intensity:)` / `.impact(flexibility:intensity:)`.
**BUILD:**
```swift
.sensoryFeedback(.success, trigger: isSaved)
.sensoryFeedback(.impact(weight: .medium, intensity: 0.9), trigger: detent) { _, new in new == .expanded }
.sensoryFeedback(trigger: state) { _, new in switch new { case .done: .success; case .failed: .error; default: nil } }
```
iOS 17. Pre-17 / zero-latency gestures: UIKit generators with `prepare()` on touch-down (engine stays warm ~1-2s; keep the generator alive). Custom textures (sub-notification-strength ticks): CoreHaptics `CHHapticTransient` with intensity/sharpness ~0.3–0.5 (see House Patterns: haptic tiers).
**GOTCHAS:** Trigger must be Equatable AND change (use a counter for repeatable events). Simulator plays nothing. Never the only feedback channel (System Haptics can be off). Apple's grammar: haptic on outcome and threshold, not on every tap — routine reversible changes are silent. Gate on Low Power Mode.

### The Apple celebration (no confetti, ever)
**EXISTS:** System celebration = a layered instant, over in under a second: symbol morph + one bounce + color sweep + success haptic. Fitness ring-close, Apple Pay check.
**BUILD:**
```swift
Label(done ? "Saved" : "Save", systemImage: done ? "checkmark.circle.fill" : "circle")
    .contentTransition(.symbolEffect(.replace))
    .symbolEffect(.bounce, options: .nonRepeating, value: done)
    .foregroundStyle(done ? .green : .secondary)
    .sensoryFeedback(.success, trigger: done)
// optional decaying color sweep behind it: Circle().fill(.green.opacity(sweep ? 0 : 0.25)).scaleEffect(sweep ? 2.2 : 0.4)
```
**WHY:** particles fight Reduce Motion, age the app, and celebrate the app instead of the user. The haptic is felt in the hand — more visceral than anything visual.

---

# Part 9 · Controls

### Buttons
**BUILD:** the canonical CTA: `Button { } label: { Text("Continue").frame(maxWidth: .infinity) }.buttonStyle(.borderedProminent).buttonBorderShape(.capsule).controlSize(.large)` (frame on the LABEL for full-width fill). Hierarchy: exactly one `.borderedProminent` per region; `.bordered` secondary; `.borderless`/plain tertiary. `role: .destructive` = free red + correct menu treatment (don't stack `.foregroundColor(.red)`). Hold-to-repeat: `.buttonRepeatBehavior(.enabled)` (17).
**Pressed feel:** system styles dim; cards scale — custom `ButtonStyle` reading `configuration.isPressed` → `scaleEffect(0.95–0.97)` + ~100ms ease-out or `.snappy`, on pointer-down. Never `.onTapGesture` on a card (no feedback, breaks VoiceOver) — Button + style is the native path. Scale ladder from shipping code: 0.95 small grid cells, 0.97 full-width primaries, 0.98 large cards.
**GOTCHAS:** In List rows buttons default borderless and the whole row fires all of them — `.buttonStyle(.borderless)` each. `buttonBorderShape` no-ops on plain styles.

### Pickers / Toggle / Slider / Stepper / Gauge / ProgressView
**BUILD:** segmented (2–5 exclusive views of the same content, never actions): `.pickerStyle(.segmented)` · menu rows in forms: `.menu` · Settings-style push: `.navigationLink` (16, needs a NavigationStack) · Toggle: keep the system switch, `.tint` it · Slider: commit in `onEditingChanged`, add `.sensoryFeedback(.increase/.decrease)` at bounds · Stepper: free hold-to-repeat with acceleration · Gauge (16): `.gaugeStyle(.accessoryCircular)` + `.tint(Gradient(...))` (widget/watch look; sized ~50-60pt) · ProgressView: determinate bar / `.circular` spinner; `ProgressView(timerInterval:)` self-updating countdown (16).
**GOTCHAS:** **`tag()` type must exactly match the selection binding type, including Optional-ness** — the most common Picker bug in existence; mismatch = selection silently never updates. Segmented ignores per-segment colors (UIKit appearance only). Wheel fights ScrollView for drags. navigationLink style outside a stack renders as a dead row with no error. Gauge gradient maps across the RANGE, not the filled part.

### TipKit — the native coach mark
**EXISTS:** Rule-driven, dismissible, never modal. One feature, taught at the moment it's relevant.
**BUILD:** `struct FavoriteTip: Tip { var title: Text {...}; var message: Text? {...}; var image: Image? {...}; var rules: [Rule] { #Rule(Self.$hasFavorited) { $0 == false } } }` · `try? Tips.configure([.displayFrequency(.daily)])` once at launch (forgetting it = no tips, no error) · inline `TipView(tip)` or anchored `.popoverTip(tip)` · `tip.invalidate(reason: .actionPerformed)` when the user does the thing.
iOS 17.
**GOTCHAS:** Datastore persists across launches — `Tips.resetDatastore()` in development or you'll think it's broken. displayFrequency throttles globally. popoverTip blocks its anchor until dismissed. Never for onboarding sequences (no ordering guarantees).

### Text fields — the native form kit
**BUILD:**
```swift
TextField("Email", text: $email)
    .focused($focus, equals: .email)
    .textContentType(.emailAddress)          // QuickType autofill; .oneTimeCode = SMS magic; .newPassword = Keychain suggest
    .keyboardType(.emailAddress)
    .submitLabel(.next)
.onSubmit { focus = nextField() }            // return key advances focus
.scrollDismissesKeyboard(.interactively)     // Messages-style drag-along (16)
.toolbar { ToolbarItemGroup(placement: .keyboard) {   // attach ONCE at container level
    Button("Prev"){}; Button("Next"){}; Spacer(); Button("Done") { focus = nil }
}}
.defaultFocus($focus, .email)                // 17; pre-17: asyncAfter(0.5) in onAppear
```
**GOTCHAS:** Multiple fields each adding a keyboard ToolbarItemGroup = stacked bars/crashes — one toolbar, branch on focus. No software keyboard (iPad + hardware kb) = no accessory bar; never put functionality only there. Number pads have no return key — the keyboard toolbar Next/Done chain is the fix (House Patterns).

---

# Part 10 · Gestures and physics

### Rubber-banding + velocity dismissal
**EXISTS:** Apple surfaces never hard-stop (resistance grows past limits) and release decisions honor velocity: a flick dismisses from anywhere; a slow drag must cross ~40-50%.
**BUILD:**
```swift
func rubberBand(_ offset: CGFloat, limit: CGFloat) -> CGFloat {
    offset <= limit ? offset : limit + 0.55 * limit * log1p((offset - limit) / limit)
}
DragGesture()
    .onChanged { dragY = rubberBandedValue($0.translation.height) }
    .onEnded { v in
        let velocity = v.velocity.height                     // 17; pre-17: predictedEndTranslation
        if velocity > 1200 || v.translation.height + velocity/8 > height * 0.5 { dismissWithSpring() }
        else { withAnimation(.snappy) { dragY = 0 } }        // spring snap-back, never easeOut (the web-feel tell)
    }
```
**GOTCHAS:** `highPriorityGesture(DragGesture())` over a ScrollView kills scrolling — use `simultaneousGesture` + axis locking, or `DragGesture(minimumDistance: 20)`. Prefer `@GestureState` for transient offsets (auto-resets on system cancellation). Thresholds: 1000–1500 pt/s. Asymmetric timing: slow where the user decides (hold-to-delete 2s linear), fast where the system responds (release 200ms ease-out).

### Hit areas + iPad pointer
**BUILD:** `.contentShape(Rectangle())` makes Spacers/padding tappable (transparent regions aren't, by default); small icons get `.frame(width: 44, height: 44).contentShape(Rectangle())`. Kind-specific shapes fix polish bugs: `.contentShape(.contextMenuPreview, RoundedRectangle(cornerRadius: 16))` kills the square flash behind a rounded card's context menu (the #1 polish fix); `.contentShape(.hoverEffect, .capsule)` shapes the iPad pointer. `.hoverEffect(.lift)` (icon-like) / `.highlight` (cell-like), iPadOS 13.4.

---

# Part 11 · Accessibility contract

- `@Environment(\.accessibilityReduceMotion)` — swap animation (`reduceMotion ? nil : .snappy`), swap transition (`.opacity` fallback), gate every `repeatForever` before starting, drop gesture-driven decorations entirely. Crossfades and opacity stay: removing all feedback feels broken. Non-View contexts read `UIAccessibility.isReduceMotionEnabled` (re-read on its notification).
- Reduce Transparency: system materials adapt automatically; custom translucency doesn't — branch to solid. Never encode information only in what shows through glass.
- Increase Contrast: `\.colorSchemeContrast`. iOS 26 "Tinted" display setting raises glass opacity system-wide — never hardcode transparency assumptions.
- VoiceOver: real Buttons, never tap-gesture rectangles. Composite rows: `.accessibilityElement(children: .combine)` + `.isButton` trait + `.accessibilityValue("Expanded")` + a hint, and put the action on `.accessibilityAction`. Live announcements: `UIAccessibility.post(notification: .announcement, argument:)`. A ticking clock speaks a fact ("On shift, 47 minutes"), not its digits.
- Dynamic Type: stack, never clip; labels never hyphen-wrap; AX5 test before done.

---

# Part 12 · The long tail (know these exist)

Compact format: **name** `floor` — exists / use when / build-hint. Grouped by theme. All verified.

## Lists and sections
- **badge(_:)** `15` — trailing count/text on List rows and tab items (Mail's inbox count) / unread counts / `Text("Inbox").badge(42)`. Invisible outside List/TabView.
- **badgeProminence(.increased)** `17` — filled-capsule badge / actionable counts vs passive info.
- **headerProminence(.increased)** `15` — large bold section header instead of small caps gray.
- **listSectionSpacing / listRowSpacing** `17` — gaps between sections / card-feel gaps between rows without leaving List.
- **listRowSeparator/Tint, listSectionSeparator(edges:)** `15` — hide/tint hairlines, per edge (kill the stray line above row one).
- **sectionActions** `26` — native "Add" affordance scoped to a Section.
- **listSectionMargins** `26` — full-bleed content (maps, images) inside inset-grouped sections.
- **alternatingRowBackgrounds** `macOS 14` — Finder zebra striping for Table (never on iOS product UI).

## Toolbar and nav chrome
- **toolbarTitleMenu** `16` — pull-down menu on the nav title itself (chevron; Files/Notes) / document actions / pairs with **RenameButton** + `.renameAction`.
- **toolbar(removing: .title)** `17` — keep the back button, drop the title (screens whose content is the title).
- **navigationDocument(url)** `16` — title becomes a draggable document chip / Mac proxy icon for free.
- **navigationSubtitle** `26 iOS / 11 macOS` — "Updated 5m ago" under the title.
- **statusBarHidden(true)** `13` — photo viewers, players.
- **persistentSystemOverlays(.hidden)** `16` — hide the home indicator (it's a preference; returns on touch).
- **toolbarVisibility(.hidden, for: .tabBar)** `18` — hide the tab bar on a detail push.

## Presentation extras
- **presentationSizing(.form / .page / .fitted)** `18` — UIKit formSheet parity on iPad/Mac; composable `.form.fitted(horizontally:)`.
- **dialogSeverity(.critical) / dialogIcon / dialogSuppressionToggle** `macOS 13/13/14` — caution-triangle alerts, custom alert icons, native "Don't ask again" (you persist the Bool).

## Scrolling extras
- **scrollBounceBehavior(.basedOnSize)** `16.4` — no rubber-band when content fits (short chats). Horizontal needs explicit `axes:`.
- **scrollIndicatorsFlash(onAppear:/trigger:)** `17` — "there's more content" signal after loading.
- **scrollDisabled** `16` — lock scroll during drag-reorder without killing inner gestures.
- **scrollInputBehavior(.disabled, for: .touch)** `26` — touch draws, trackpad pans.
- **onScrollVisibilityChange(threshold:)** `18` — autoplay at 50% visible, impression analytics.

## Geometry and render tree
- **onGeometryChange(for:of:action:)** `16 (back-deployed)` — THE GeometryReader killer: transform proxy → tiny Equatable, action fires on change only.
- **geometryGroup()** `17` — fixes children "swimming"/teleporting when an ancestor's frame animates. The go-to animation-anomaly fix.
- **compositingGroup()** `13` — flatten before opacity/shadow so the effect applies to the silhouette, not each subview (fixes double-dark overlaps, per-subview shadows).
- **drawingGroup()** `13` — Metal-rasterize complex static vector/gradient subtrees that jank; breaks subpixel text AA, profile first.
- **coordinateSpace(name:) + frame(in: .scrollView)** `13/17` — cross-view frame math.

## Layout
- **ViewThatFits** `16` — first child that fits wins (roomy HStack → AX-size VStack).
- **AnyLayout** `16` — swap `HStackLayout()`↔`VStackLayout()` at runtime with identity preserved → the switch animates.
- **Layout protocol** `16` — real flow layouts (tag clouds), radial menus: `sizeThatFits` + `placeSubviews`.
- **GroupBox** `14` — native rounded card-of-controls with automatic material.
- **DisclosureGroup** `14` — expand/collapse with rotating chevron; hierarchical Lists free via `List(children:)`.
- **fixedSize(horizontal: false, vertical: true)** `13` — the "text truncates instead of wrapping" fix. **layoutPriority** — two Texts fighting in an HStack.
- **Grid/GridRow** `16` — cross-row column alignment (spec sheets); `gridCellUnsizedAxes(.horizontal)` on Dividers or they inflate columns; non-GridRow children span all columns.
- **containerRelativeFrame** `17` — "2.5 cards visible" math: `(.horizontal, count: 5, span: 2, spacing: 12)`; container = scroll view/column/window, NOT any parent stack; misbehaves inside List.

## Pointer, pencil, keys, focus
- **springLoadingBehavior(.enabled)** `17` — drag-hover over a row opens it (Files/folders).
- **pointerStyle(.frameResize/.grabActive/...)** `macOS 15` — real pointer shapes.
- **onKeyPress(.upArrow)** `17` — hardware keys on a focusable, focused view.
- **onModifierKeysChanged(mask: .option)** `18` — Option-key alternate buttons (Save → Save As).
- **onPencilSqueeze / onPencilDoubleTap** `17.5` — Pencil Pro tools; respect `preferredPencilSqueezeAction`.
- **defersSystemGestures(on: .bottom)** `16` — your edge swipe wins the first swipe.
- **keyboardShortcut(.defaultAction / .cancelAction)** `14` — Return/Esc semantics on buttons; ⌘-hold HUD free on iPad.
- **defaultFocus** `17` — focus the right field when a scope activates.
- **focusSection()** `tvOS 15/macOS 13` — directional focus reaches off-axis sidebars.

## Text and typography
- **fontWidth(.condensed/.expanded)** `16` — the SF width axis (dense numerics, score aesthetics).
- **fontDesign(.rounded/.serif/.monospaced)** `16.1` — design switch keeping semantic sizes.
- **textScale(.secondary)** `17` — proper small text that still tracks Dynamic Type (units next to big numbers).
- **typesettingLanguage** `17` — correct line boxes for tall scripts inside another locale.
- **kerning vs tracking** `13` — kerning preserves ligatures (fine typography); tracking is uniform and kills them (uppercase headline spacing).
- **textCase(nil)** `14` — stop Form section headers auto-uppercasing.
- **textSelection(.enabled)** `15` — long-press copy on static Text (order numbers, error codes). Whole-Text selection only.
- **findNavigator(isPresented:)** `16` — native find-and-replace, TextEditor only.
- **privacySensitive()** `15` — auto-redacts in the app switcher snapshot and locked widgets (balances, card numbers).
- **help("Export as PDF")** `14` — macOS tooltip + accessibility hint elsewhere; every icon-only Mac button.

## Controls extras
- **pickerStyle(.palette) + paletteSelectionEffect(.symbolVariant(.fill))** `17` — swatch row inside menus (Reminders color picker).
- **ControlGroup** `15` — undo/redo-style clusters; `.compactMenu`/`.palette` styles; max 3 in `.menu`.
- **MultiDatePicker** `16` — multiple non-contiguous dates (`Set<DateComponents>`).
- **EditButton / RenameButton / PasteButton** `13/16/16` — system buttons; PasteButton reads Transferables with NO paste-permission alert.

## Shapes, color, styles
- **.rect(topLeadingRadius:...) clip shorthand** `16` — top-rounded sheets without a custom Path.
- **ConcentricRectangle** `26` — inner radius derived from container corners (the Liquid Glass nesting rule); `containerShape(.rect(corners: .concentric(minimum: 12)))`; pre-26 rule of thumb: innerRadius = outerRadius − inset.
- **strokeBorder vs stroke** `13` — strokeBorder insets so borders don't clip at view edges (needs InsettableShape).
- **trim(from:to:)** `13` — progress rings, draw-on lines: `Circle().trim(from: 0, to: progress).stroke(style: .init(lineWidth: 8, lineCap: .round)).rotationEffect(.degrees(-90))`.
- **Color.gradient** `16` — free subtle vertical gradient on any color: `.fill(.blue.gradient)`. Instant depth, zero stops.
- **Color.mix(with:by:)** `18` — perceptual blends for programmatic ramps.
- **shadow ShapeStyle: .fill(.blue.shadow(.inner(radius: 3, y: 2)))** `16` — inner shadows, finally; travels with the fill (etched controls, shadowed text fills).
- **foregroundStyle(primary, secondary, tertiary)** `15` — one modifier sets the hierarchy; palette symbols pick the levels up.
- **backgroundStyle(.blue.gradient)** `16` — redefines what `.background` semantic resolves to for descendants (GroupBox theming).
- **MeshGradient** `18` — 3×3 control-point organic gradients, animatable by moving points.

## Images and export
- **ImageRenderer** `16` — any view → UIImage/CGImage/PDF. SET `renderer.scale = displayScale` or it's blurry. No async content (AsyncImage won't render). MainActor.
- **allowedDynamicRange(.high)** `17` — real HDR display in photo viewers (never on chrome).
- **imageScale(.large)** `13` — symbol size relative to font context.

## System integrations
- **ShareLink + SharePreview** `16` — share sheet + AirDrop; SharePreview REQUIRED for custom Transferables or the sheet is blank. Lazy payloads via FileRepresentation build only when the sheet asks.
- **photosPicker** `16` — out-of-process picker, NO permission prompt, multi-select, filters.
- **fileImporter/Exporter/Mover** `14` — remember `url.startAccessingSecurityScopedResource()`.
- **@Environment(\.requestReview)** `16` — StoreKit prompt, system-throttled; fire after a success moment.
- **SubscriptionStoreView / StoreView / ProductView** `17` — Apple-built paywall driven by App Store metadata.
- **translationPresentation** `17.4` — the Safari translate sheet over your text, on-device, no keys.
- **quickLookPreview($url)** `14` — PDF/USDZ(AR)/video/Office preview in one modifier; local file URLs only.
- **ContactAccessButton** `18` — per-contact limited-access grants inside your own search UI.
- **WebView + WebPage** `26` — first-party SwiftUI WebKit; kills the representable boilerplate.
- **ImageAnalysisInteraction** `16, UIKit bridge` — Live Text (select text in screenshots) on your image views.

## Environment and plumbing
- **\.displayScale** `13` — hairlines: `1 / displayScale`; ImageRenderer scale.
- **\.scenePhase** `14` — hide sensitive content on `.inactive` (app-switcher), pause timers.
- **dynamicTypeSize(...accessibility2) clamp** `15` — badges/chrome that break at AX5 (sparingly).
- **interactionActivityTrackingTag("Feed")** `16` — hangs/hitches in Instruments and MetricKit attributed to named screens. Zero visual cost.
- **@Entry macro** `Xcode 16` — one-line custom EnvironmentValues keys.
- **@Previewable** `Xcode 16` — @State directly inside #Preview for interactive previews.
- **transaction { $0.animation = nil }** `13` — kill an inherited animation on one child; scoped `transaction(_:body:)` 17.
- **animation(_:body:) scoped** `17` — one property animates slow while position uses the ambient spring.
- **CustomAnimation protocol** `17` — your own physics where built-ins can't express it.
- **@Animatable macro** `26` — auto-synthesized animatableData for custom Shapes.
- **containerBackground(for: .widget)** `17` — mandatory on widgets; StandBy/lock strip it automatically.
- **contentTransition(.interpolate / .opacity)** `16` — morph text weight/color in place; crossfade content without layout jumps.

## UIKit escape hatches every native app needs
- **window.overrideUserInterfaceStyle** `13` — theme toggles must reach UIKit-presented chrome (share sheets, alerts); `preferredColorScheme` doesn't.
- **UIImpactFeedbackGenerator.prepare()** `10` — pre-warm on touch-down for zero-latency gesture haptics; generator stays warm 1-2s; keep it alive.
- **UITabBarController delegate hook** — SwiftUI has no "reselected the current tab" signal; a `UIViewControllerRepresentable` that chains the tab bar delegate detects re-tap → pop to root.

---

# Part 13 · House patterns (proven in production)

Battle-tested compositions from Vero and Payday, with the reasoning that made them. These are the patterns that make the apps feel native beyond what single APIs give you.

### The tucked drawer (hero card + peeking recess)
The signature: a lifted card with a gray shape tucked underneath, peeking ~14pt, expanding on tap. Reads as ONE surface opening, never a second card.
```swift
VStack(spacing: 0) {
    card()                                      // owns its own shadow
        .contentShape(Rectangle())
        .onTapGesture { HapticThenSpring() }    // light tap + spring(duration: 0.5, bounce: 0.24)
        .zIndex(1)
    drawer
        .padding(.top, -cardRadius + 2)         // the tuck (e.g. -22pt under a 24pt-radius card)
        .zIndex(0)
}
// drawer:
let shape = UnevenRoundedRectangle(cornerRadii: .init(topLeading: 0, bottomLeading: 24, bottomTrailing: 24, topTrailing: 0), style: .continuous)  // square top = "slides out from under"
VStack(spacing: 0) {
    lipRow.padding(.top, cardRadius + 12)       // constant in BOTH states — changing padding = one more thing moving
    if isExpanded { rows.transition(.opacity) } // opacity ONLY: .move reads as a second drawer sliding out
}
.background(shape.fill(insetFieldColor))        // a RECESS, not a lift: no shadow (one raised object per screen)
.clipShape(shape)                                // clip BEFORE any shadow so the reveal can't paint ghost rows outside
```
Rules learned in production: chevron rotates 180°, never symbol-swaps (pops mid-animation). Accordion state = one optional id (`expandedGroupId`), nil = all closed. Host in a ScrollView, NOT a List (List animates row resizes on its own clock and everything below stutters). The collapsed lip text must reconcile to the number above it. Accessibility: combine the card into one element, add `.isButton`, `.accessibilityValue("Expanded"/"Collapsed")`, and put the toggle on `.accessibilityAction`.

### Undo toast (delete now, forgive later)
Apple's own grammar (Mail, Notes) for destructive actions: act immediately, offer Undo in a floating glass capsule, never ask "are you sure?"
```swift
// 1. Snapshot EVERY field into a plain struct BEFORE deleting (the model instance is gone by Undo time)
// 2. delete → save → medium haptic → auto-dismiss Task (~4s, cancellable)
// 3. Undo → reinsert restored models → success haptic
// 4. Overlay at the container level:
.overlay(alignment: .bottom) {
    if state.snapshot != nil {
        toast  // HStack("Shift deleted" | Undo button in accent) in a glass rounded rect
            .transition(reduceMotion ? .opacity : .move(edge: .bottom).combined(with: .opacity))
            .onAppear { UIAccessibility.post(notification: .announcement, argument: "Shift deleted") }
    }
}
.animation(.smooth, value: state.snapshot != nil)
```
Details that matter: delete whole logical units (all rows of a shift), not fragments. Cancel the pending dismiss when a new delete arrives. 4-5s window. Server-backed variant: optimistic restore + let the next fetch show true server state.

### Glass wrappers (one sanctioned call site)
All glass routes through app-level wrapper functions; a CI grep bans raw `.glassEffect(` in features.
```swift
@ViewBuilder func nativeGlassCapsule(interactive: Bool = true) -> some View {
    if #available(iOS 26.0, *) {
        self.glassEffect(.regular.interactive(interactive), in: .capsule)
            .glassEffectTransition(UIAccessibility.isReduceMotionEnabled ? .materialize : .matchedGeometry)
    } else {
        self.background(Capsule().fill(.ultraThinMaterial))
            .overlay(Capsule().stroke(Color.primary.opacity(0.15), lineWidth: 1))
    }
}
// Companions: legacyHiddenToolbarBackground (pre-26 only — never suppress iOS 26 chrome),
// tabBarMinimizeOnScroll (26-gated no-op), sheetBackground (pre-26 presentationBackground fix)
```

### The action tab (Tab role .search as a floating button)
The `.search` role gives a separate circular glass button beside the tab pill. Repurpose it as the app's core action: selecting it opens a sheet and snaps back.
```swift
Tab("Log", systemImage: "plus", value: .log, role: .search) { Color.clear }   // never actually navigated to
// onChange(of: selection):
if newTab == .log {
    pendingSheet = .new
    isRestoring = true                       // reentrancy guard — the restore itself fires onChange
    DispatchQueue.main.async { selection = previousTab }
    return
}
previousTab = newTab
```
Tint it when it IS the app's core action; leave it neutral when it opens a secondary feature. iOS 17 fallback: a hand-rolled floating glass capsule button, bottom-trailing, hidden while its sheet is up.

### Bar-less screens: the manual top fade
A screen with `navigationTitle("")` + custom in-content header gets NO bar and therefore NO iOS 26 scroll edge effect. The manual fade is correct on ALL versions there:
```swift
.overlay(alignment: .top) {
    LinearGradient(colors: [pageColor, pageColor.opacity(0)], startPoint: .top, endPoint: .bottom)
        .frame(height: 118).ignoresSafeArea(edges: .top).allowsHitTesting(false)
}
```
Screens with a real title get the native effect free — don't add fades there.

### Floating-bar clearance numbers
Content must scroll under floating chrome but end clear of it: `contentMargins(.bottom, 88, for: .scrollContent)` above a floating action button (leaves indicators/edge-blur alone); `safeAreaInset(edge: .bottom) { Color.clear.frame(height: 112).allowsHitTesting(false) }` above the iOS 26 floating tab bar (List-friendly). Pinned CTA variant: real button in `safeAreaInset` with the page background.

### Rolling money (the complete recipe)
Display ladder: SF Pro Rounded heavy at fixed sizes (60/48/40/36/30/24/22/20 — money must not scale with Dynamic Type), semantic SF Pro with bumped weights for everything else, exactly one deliberate regular-weight token for paragraph copy (stacked semibold paragraphs read as a wall of bold).
```swift
Text(Money.string(fromCents: cents))          // integer cents in, formatted string out, ONE formatter
    .font(display.heroSize).monospacedDigit()
    .contentTransition(.numericText())
    .animation(reduceMotion ? nil : .smooth, value: cents)
    .lineLimit(1).minimumScaleFactor(0.5)
```
The reveal moment (post-save): replace the whole sheet body for ~2s with the total + one true sentence; record nights get the single earned flourish — the amount sweeps to the accent once, paired with the success haptic. Tappable to skip, fully static under Reduce Motion, announced to VoiceOver. No confetti.

### Currency input (digits shift in from the right)
Apple's own currency-field feel (Wallet, Cash): a visible formatted Text + an invisible TextField driving it.
```swift
ZStack {
    Text(Money.string(fromCents: cents))       // ALWAYS derived from cents — never parse a string to Double
        .monospacedDigit().contentTransition(.numericText())
        .accessibilityHidden(true)
    TextField("", text: $digitsText)
        .keyboardType(.numberPad).focused($focus).opacity(0.01)
        .accessibilityLabel("Amount").accessibilityValue(Money.string(fromCents: cents))
}
.contentShape(Rectangle()).onTapGesture { focus = true }
.onChange(of: digitsText) { _, new in
    let filtered = String(new.filter(\.isNumber).prefix(7))   // $99,999.99 cap
    if filtered != new { digitsText = filtered }
    cents = Int(filtered) ?? 0
}
.onChange(of: cents) { _, new in    // external writes (a scan, a chip) must resync the hidden buffer
    guard Int(digitsText) ?? 0 != new else { return }
    digitsText = new == 0 ? "" : String(new)
}
```
Focus ring: 2px accent strokeBorder when this field drives the keypad, hairline otherwise. Suggestions render as placeholder-only hints while the value is zero — a hint to tap into, never a value that saves itself. A count field with no value shows nothing, never a misleading "0".

### Keyboard toolbar focus chain
Number pads have no return key. The keyboard accessory bar carries the whole flow:
```swift
ToolbarItemGroup(placement: .keyboard) {
    Button("Scan receipt") { scan() }          // secondary action, leading
    Spacer()
    if let focused, let next = nextField(after: focused) {
        Button("Next") { selectionHaptic(); focus = next }
    }
    Button(isEditing ? "Done" : "Save") { commit() }.disabled(!canSave)
}
// + ScrollViewReader: on every focus change, proxy.scrollTo(field, anchor: .center) with a spring —
// system keyboard avoidance only keeps the LAST field visible; Next can jump to rows it never tracked.
// The scroll id IS the focus enum case: .id(Field.cash) on each row.
```
The chain is a pure function `nextFocusField(after:)` over an enum, shortening when sections are collapsed, cycling at the end.

### Breathing dot (live/recording indicator)
The one sanctioned ambient loop: opacity only, never size.
```swift
Circle().fill(accent).frame(width: 8, height: 8)
    .opacity(dimmed ? 0.55 : 1)
    .onAppear {
        guard !reduceMotion else { return }
        withAnimation(.easeInOut(duration: 1.6).repeatForever(autoreverses: true)) { dimmed = true }
    }
```
High-frequency variant (inside glass chrome that must not re-render): hoist onto Core Animation with a `UIViewRepresentable` + `CABasicAnimation` on layer opacity — the compositor animates, SwiftUI never invalidates. Companion elapsed clock: drive the string from `TimelineView(.periodic(by: 1))` so `contentTransition(.numericText)` can roll digits (a self-updating `Text(timerInterval:)` swaps them).

### Chart scrubbing (Health-style)
```swift
Chart(points, id: \.date) { p in
    BarMark(x: .value("Date", p.date, unit: granularity), y: .value("Value", p.value))
        .foregroundStyle(accent.opacity(barOpacity(p)))   // single hue, 0.35–1.0 by relative size; selected = 1.0, rest halve
        .cornerRadius(3)
}
.chartXSelection(value: $selectedDate)      // raw value under the finger — snap to nearest datum yourself
.chartYAxis(.hidden)
.onChange(of: selectedDate) { old, new in if old != new { selectionHaptic() } }
.accessibilityElement(children: .ignore)    // one spoken summary beats per-bar elements on a small chart
.accessibilityLabel(chartSummary)
```
The header doubles as the scrub readout (selected value replaces the title). Auto-granularity: ≤21 days daily, ≤120 weekly, ≤730 monthly, else yearly — the chart gets calmer as data grows. Align the visible domain to whole buckets or edge bars clip.

### Haptic tiers + animation tokens (the system, not the API)
One named tier per action class, all gated on Low Power Mode, raw generators banned outside the tokens file:
`lightTap` navigation-level (drawer toggle, start) · `medium` destructive-but-undoable · `success` real saves only · `error` failed saves · `selection` scrubbing/pickers/Next · CoreHaptics transients (intensity 0.35/sharpness 0.30 per step; 0.50/0.45 on settle) for sub-notification texture in agentic/progress UI. Routine reversible changes are silent — no haptic in live-save.
Animation tokens mirror it: durations on a 100ms grid; `paperSpring = .snappy` (precise), `premiumSpring = .smooth` (gentle/amounts), one deliberate drawer spring `(duration: 0.5, bounce: 0.24)` ("livelier than snappy, shy of .bouncy so it never reads cartoonish"), and a first-class `reducedMotionToken = .easeInOut(0.15)` so every call site is a two-way ternary against named constants.

### Reduce Motion idioms (the complete set)
```swift
.transition(reduceMotion ? .opacity : .move(edge: .bottom).combined(with: .opacity))
.animation(reduceMotion ? nil : token, value: x)                      // or a named RM token instead of nil
.contentTransition(reduceMotion ? .identity : .numericText())
if reduceMotion { mutate() } else { withAnimation(token) { mutate() } }
.onAppear { guard !reduceMotion else { return }; startLoop() }        // gate every repeatForever
.offset(y: appeared || reduceMotion ? 0 : 8)                          // static offset guard
if !reduceMotion { scrubbing UI }                                     // whole decorative features omitted
```
Fixed-edge lens transitions: each segmented lens keeps a constant home edge (computed edges capture the wrong render's state on removal); plain crossfade under RM.

### The enforcement layer
A grep-based design lint in CI keeps all of this true: no raw glassEffect/materials/haptic generators/hex colors/toolbarBackground outside the design-system file, no forbidden hues, no emojis in user-facing strings, no fake glass, and per-file stricter laws for the most disciplined surfaces. ~8 seconds, zero tolerance. The catalog is only as good as the lint that defends it.

---

# Sources

Every entry was verified against at least one primary or established source at compile time (2026-08); nothing here is from model memory alone.

**Primary (Apple):**
- developer.apple.com/documentation — per-API pages (availability floors, signatures, doc text quoted in entries), including SwiftUI updates pages per release
- Human Interface Guidelines (materials, glass, motion, feedback)
- WWDC sessions: 23-10158 "Animate with springs", 23-10157 "Wind your way through advanced animations", 23-10159 "Beyond scroll views", 23-10258 (SF Symbols 5), 24-10145 "Enhance your UI animations and transitions", 24-10188 "What's new in SF Symbols 6", 24-10151 "Create custom visual effects", 24-10074 "Get started with Dynamic Type", 25-323 "Build a SwiftUI app with the new design"
- Technote TN3154 (NavigationStack migration), Apple sample code (Destination Video, Landmarks)

**Established community sources (recipes, gotchas, teardowns):**
- Hacking with Swift (Paul Hudson) — incl. the iOS 18/26 API roundups and the canonical zoom-transition recipe
- fatbobman.com — geometryGroup, containerRelativeFrame, TextRenderer, custom scroll-target deep dives
- Donny Wals, Sarunw, Use Your Loaf, Swift with Majid, createwithswift.com, avanderlee.com, augmentedcode.io, mobilea11y.com, mehmetbaykar.com
- Douglas Hill (zoom transitions), Kai Oelfke (TipKit pitfalls), Rudrank Riyam (Music lyric effects), mackuba.eu SwiftUI availability index, exploreswiftui.com, iOS-26-by-Examples
- Open radar FB13447486 (pinned headers vs scrollTo), Stack Overflow threads cited inline where a gotcha is developer-reported

**Shipping code:** Vero (`AskVero/ios`) and Payday production sources — every Part 13 pattern is real code, trimmed, with its original reasoning comments.

Corrections encoded against common misinformation: spring presets are back-deployed to iOS 13 (Apple doc pages, not blog consensus); `.wiggle/.breathe/.rotate` are iOS 18 per the WWDC24 SF Symbols 6 transcript (several 2026 blogs misattribute them to 26); `scrollEdgeEffectHidden` is the shipping name (beta articles show `scrollEdgeEffectDisabled`).

---

*Companion documents in this repo: [README.md](README.md) (the design thesis, what should feel native and why) and [LLM.md](LLM.md) (the thesis in machine form). This catalog is the how. For iOS work, paste LLM.md and this file together.*
