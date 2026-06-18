# konva-motion Documentation Plan

Tutorial-focused docs ("how to" + "when"), not an API reference. Built on the
existing Fumadocs + React Router site in `packages/docs`, with live previews via
the `<km-player>` web component. Tone is imperative and frame-first, and leans on
Konva's config-object conventions.

## Locked decisions

- **Teach the real API.** Every page uses `Composition` / `Sequence` /
  `seq.register()` / `interpolate()` / `Easing`. The current
  `introduction.mdx` / `installation.mdx` teach a `Timeline.to()` API that does
  **not** exist in core — these get rewritten.
- **Don't name Remotion in page prose.** Remotion is inspirational only. Credit
  Remotion and Konva once, in a **Credits** section on the `introduction` page,
  and nowhere else. No "Remotion-style" / "API-compatible with Remotion" /
  "mirrors Remotion" phrasing in the published pages or demo comments. (Internal
  design docs under `doc/` may still reference it.)
- **Studio is a full section** (4 pages), banner-marked **Preview** (it is
  `0.0.0`, React-only, lives in `demo2`).
- **Full live demo library** — one `.ts` composition per concept in
  `packages/docs/src/demos/`, rendered live in `<km-player>` and shown as
  source from the *same file* (single source of truth).
- **All demos run at 60fps.** New demos use `fps: 60`; when you need a specific
  real-time pace, size `durationInFrames` (and any frame anchors / interpolate
  ranges) for 60fps, not 30. The four ported demos were converted: frame counts,
  `from`/`trim` windows, and frame-rate-dependent oscillators were all doubled or
  halved so timing and media sync are unchanged. If a page shows a demo's code
  inline (e.g. `installation` mirrors `first-tween`), keep the inline copy in
  sync with the demo file.

## Tone of voice

- Imperative, frame-first. Lead with the smallest thing that runs.
- Reinforce one mental model everywhere: composition owns `fps` +
  `durationInFrames`; a `Sequence` is a range-gated layer; each tick runs
  updaters and `batchDraw`s active sequences. Animation = "what is this
  attribute *at frame N*" — never CSS transitions or `setInterval`.
- Config-object parity with Konva: "every `Konva.RectConfig` prop works, plus
  flex props." Link out to Konva docs instead of re-documenting raw attributes.
- Each page shape: *what it's for, smallest runnable example, live demo,
  2-3 variations, when to reach for it + gotchas.*

### Write like a human, not an LLM

The docs should read as if a developer wrote them. Avoid the tells that mark
machine-generated prose:

- **No em-dashes or en-dashes (`—`, `–`).** Use a plain hyphen `-`, a comma, a
  colon, parentheses, or just split the sentence. This is the biggest tell;
  scrub every one.
- **No "it's not just X, it's Y" / "X isn't merely Y" constructions.** Say the
  thing directly.
- **Drop filler superlatives** ("seamless", "robust", "powerful", "effortless",
  "leverage", "delve", "boasts", "unlock", "elevate"). State what it does.
- **Go easy on bold.** Bold a term on first definition, not every other phrase.
- **No "Let's dive in", "In conclusion", "It's worth noting that".** Cut throat-
  clearing; lead with the point.
- **Vary sentence length; allow short sentences.** Don't make every sentence a
  balanced two-clause structure.
- Use straight quotes/apostrophes (`'` `"`) in prose where practical.

Apply this to page prose, demo source comments, and code-block comments. When
editing existing pages, treat an em-dash as a defect to fix.

## Best practices to teach throughout

These are the recommended-way principles. Define each once on a short
**`best-practices`** page (Getting Started, after core-concepts), then reinforce
the relevant one as a `<Callout>` on the pages where it bites. Frame them as
*recommended*, not *required* — most have a working escape hatch.

- **Prefer konva-motion wrappers over raw `Konva.*`.** `Rect`/`Circle`/`Text`/
  `Image`/… from core extend their Konva equivalents *and* add full flex
  participation. NOTE (verified against `layout/flex/flex.ts`): a raw
  `Konva.Rect`/`Text`/`Image` with a fixed numeric size **is** measured and
  placed (and origin-corrected) inside a `Flex` — the old "won't reflow / won't
  be measured" framing is inaccurate. What raw nodes *can't* do is express
  percentage sizes (`width: "100%"`), flex child props
  (`flexGrow`/`flexShrink`/`flexBasis`/`alignSelf`), nested `Flex`/`Block`
  containers, or the rich `Text` (fit/highlights/typewriter) / `Image`
  (objectFit) / `Block` (background/border/shadow) features. Rule of thumb:
  import the drawing vocabulary from `@konva-motion/core`, reach for `Konva.*`
  only for something core doesn't wrap yet. *(Not never, just not the default.)*
- **Animate as a function of frame.** Drive attributes from `localFrame` via
  `seq.register()` + `interpolate()`/`Easing`. Never `setInterval`,
  `requestAnimationFrame`, `Konva.Tween`, or CSS transitions — only frame-derived
  values render deterministically and seek/scrub correctly.
- **Keep compositions pure.** A composition should be a pure function of
  `frame` + `props` — no external mutable state, no wall-clock reads. This is
  what makes preview, scrubbing, and headless render produce identical output.
- **Add a node before you animate it.** Attach to a `Sequence`/layer first;
  tweening a detached node updates attrs but never repaints (existing gotcha).
- **Pin one copy of `konva`.** It's a peer dep and core extends its classes —
  two copies break `instanceof` and layout. Dedupe in the consumer.
- **Let layout reflow; don't hand-position when you mean layout.** Use
  `Flex`/`Block` and animate `gap`/size/`padding`; the engine recomputes each
  tick. Manual `x`/`y` math is for one-off motion, not structure.
- **`play()` is browser-only; `setFrame(n)` works anywhere.** Use `setFrame`
  for offline/server paths (it's also what the renderer drives).
- **Use `interpolate` extrapolation, not manual clamping.** Reach for
  `extrapolateLeft/Right: "clamp"` instead of `Math.min/max` around a ratio.

## Authoring gotchas (engine reality, verified)

Practical pitfalls hit while building the demos. The contributor copy lives in
[`doc/authoring-demos.md`](authoring-demos.md); keep the two in sync. (This is an
internal plan, so em-dashes here are fine — the no-em-dash rule is for published
page prose and demo comments.)

- **Layer order = add order.** A `Sequence` is a Konva layer; later-added
  sequences paint on top. Add a full-canvas background `Sequence` **first** so it
  sits under the content, not over it. (Bit `range-gate`: cards were hidden
  behind a background added last.)
- **Update `Text` content with `setText(...)`.** The core `Text` wrapper extends
  `Konva.Group`, not `Konva.Text`, so it has no `.text()` accessor; `.text(...)`
  throws. Other wrappers (`Rect`/`Circle`/`Line`/…) extend their `Konva.Shape`,
  so `.opacity()`/`.x()`/`.points()`/etc. work as normal.
- **Don't grow a flex leaf with `.width(n)`.** `FlexShape.width(n)` sets the flex
  *size value*, not the rendered width, so it won't visibly resize a leaf from an
  updater. To make a child fill space use `flexGrow` (or a `%` width) and let the
  layout reflow. To animate a Flex *container's* own size, `setAttrs({ flexWidth })`
  (the constructor takes `width`, stored as the `flexWidth` attr).
- **Raw `Konva.*` IS laid out inside a Flex.** Verified against
  `layout/flex/flex.ts`: a raw `Konva.Rect`/`Text`/`Image` with a numeric size is
  measured, placed, and origin-corrected. The wrappers' real edge is `%` sizes,
  flex child props, nested containers, and the rich Text/Image/Block features
  (see the wrappers bullet above). Don't teach "raw won't reflow."
- **60fps scaling** when authoring or porting: see the 60fps locked decision.

## Target information architecture (`meta.json`)

```
---Getting Started---
introduction          (rewrite)
installation          (rewrite)
core-concepts         (new)
best-practices        (new)
---Animating---
interpolation         (new)
easing                (new)
sequencing            (new)
---Drawing---
shapes                (new)
text                  (new)
images                (new)
---Layout---
flex                  (split from layout-and-shapes)
block                 (split — "box")
---Media---
video                 (new)
audio                 (new)
---Transitions---
transitions           (new)
---Player---
player                (rewrite/expand)
---Rendering---
rendering             (new)   ← headless MP4 export via @konva-motion/renderer
rendering-media       (new)   ← server-side assets, fonts, shader transitions (./gl)
---Studio---  (Preview)
studio-overview       (new)
studio-setup          (new)
studio-interface      (new)
studio-rendering      (new)
```

The existing `components.mdx` (design-system kitchen-sink) drops out of public
nav; its role is replaced by the focused Drawing/Layout pages. Keep the file as
an internal authoring reference.

---

# Build steps

## Step 0 — Foundations (do first, unblocks everything) ✅ DONE

- [x] Restructure `content/docs/meta.json` to the IA above (group headers +
      ordered slugs). *Applied the new group headers (Getting Started, Player)
      with only the pages that exist today; later steps append their slugs as
      pages are authored. `components`/`layout-and-shapes` dropped from public
      nav (files kept on disk as internal reference).*
- [x] Formalize the **demo authoring contract**: each demo is
      `src/demos/<name>.ts` whose default export is a `Composition` (or a
      `() => Composition` factory). `<Demo>` renders it live via `?url` into
      `<km-player src>` and shows source via `?raw` — from the same file.
- [x] Decide + (optionally) implement the `<Demo name="...">` single-prop
      helper that resolves both `?url` and `?raw` from one name, replacing the
      two-import-per-page boilerplate. *Implemented via two eager
      `import.meta.glob` maps in `src/components/demo.tsx`; `name` resolves both
      views (throws on unknown name). `src`/`source` props kept as an override.
      `src/demos/registry.ts` deleted (superseded). `player.mdx` migrated to
      `<Demo name="orbit" />` and verified live.*
- [x] Port the 4 demos that already exist in core into `src/demos/`:
      `flex-layout`, `text-highlight`, `video-sync`, `audio-mixer`. *Ported from
      `demo/src/compositions/*/composition.ts`; media copied into
      `src/demos/assets/` and imports rewritten to `./assets/...`.*
- [x] Add a short contributor note (`doc/authoring-demos.md`) describing the
      contract.

## Step 1 — Getting Started (priority: current docs are actively wrong) ✅ DONE

Pages: `introduction`, `installation`, `core-concepts`, `best-practices`.

- [x] **introduction** — pitch; frame-clock mental model (ASCII playhead diagram
      over `[from, from+duration)`); when to reach for it vs plain Konva /
      single CSS tween; `hero` live demo. *Next-steps links repointed to
      installation/core-concepts/best-practices (dropped the `components` link).*
- [x] **installation** — `npm install konva @konva-motion/core`; konva-as-peer
      Callout; **rewritten** first composition (`Composition` + `Sequence` +
      `register` + `interpolate`); mount via `setContainer` + `play()`; `<Steps>`
      walkthrough rebuilt around the real API. The `first-tween` demo IS the page
      example (View source). *Replaced the old non-existent `Timeline`/`tl.to`
      API.*
- [x] **core-concepts** — `Composition({id, fps, durationInFrames, width,
      height, loop})`; `Sequence({from, durationInFrames})` + range-gating
      (`range-gate` demo); local vs global frame; the tick loop (updaters → media
      → layout → `batchDraw`, `frame-clock` demo); `play()` (browser) vs
      `setFrame(n)` (anywhere).
- [x] **best-practices** — the recommended-way principles (see "Best practices
      to teach throughout"); leads with "prefer the core wrappers over raw
      `Konva.*`" (`wrapper-vs-raw` demo) and "animate as a function of frame."
      Each principle has a one-line *why* + a do/don't snippet. Later pages link
      back here via `<Callout>`s.
- [x] Demos: `hero`, `first-tween`, `frame-clock`, `range-gate`,
      `wrapper-vs-raw`. *All five added to `src/demos/`, use core wrappers, run at
      `fps: 60`, verified live in `<km-player>`.*
      - `wrapper-vs-raw` shows **flexGrow**: as a Flex widens, the core `Rect`
        (`flexGrow: 1`) fills the row while a raw `Konva.Rect` can't and leaves a
        gap. *(NOT the original "raw won't reflow" idea — that was found false;
        see the corrected wrappers bullet under "Best practices to teach
        throughout".)*
      - Gotchas caught: (1) the core `Text` wrapper extends `Konva.Group`, so
        update its content with `setText(...)`, not Konva's `.text(...)`; (2) the
        `FlexShape` `.width()` setter changes the flex size value, not the render
        width — to grow a leaf use `flexGrow`/layout, not `.width(n)` in an
        updater; (3) add the full-canvas background `Sequence` **first** so it
        sits under the content layers (Konva layer order = add order).
- [x] Nav: appended `core-concepts` + `best-practices` to the Getting Started
      group in `content/docs/meta.json`.

## Step 2 — Animating ✅ DONE

Pages: `interpolation`, `easing`, `sequencing`.

- [x] **interpolation** — `interpolate(input, inputRange, outputRange,
      {easing/extrapolateLeft/Right})`; multi-stop ranges; the four extrapolate
      modes (table) with `"clamp"` as the common one; `easing` option (link to
      easing); `interpolateColors`. *`interpolate-basics` + `color-shift` demos.*
- [x] **easing** — easing reshapes the 0→1 progress inside `interpolate`'s
      `easing` option (no separate tween API); presets table; `in/out/inOut`
      modifiers; factories `bezier`/`elastic`/`back`/`poly`; same move under 5
      curves via `easing-compare`.
- [x] **sequencing** — recap `from`/`durationInFrames` + local frame (link to
      core-concepts); staggering N nodes in one sequence via `frame - i * step`
      (`stagger`); `Series` for back-to-back scenes (`series-scenes`), `offset`
      semantics (0/gap/overlap), `comp.add(series)` auto-expands.
- [x] Demos: `interpolate-basics`, `color-shift`, `easing-compare`, `stagger`,
      `series-scenes`. *All run at `fps: 60`, loop, core wrappers only.*
- [x] Nav: appended `---Animating---` group + the three slugs after Getting
      Started in `content/docs/meta.json`.

Verified facts / corrections caught while building:
- `Easing.in(fn)` is an **identity no-op** in konva-motion (a bare preset already
  eases in) — documented as such; reach for `out`/`inOut` to change the shape.
- `interpolateColors(input, inputRange, outputRange)` takes **no options arg** and
  **always clamps** both ends (unlike `interpolate`). Documented as a Callout.
- `Series.add({durationInFrames, offset}, (seq) => …)`: `offset` 0 = back-to-back,
  positive = gap, negative = overlap (cross-fade); `comp.add(series)` expands via
  `SequenceProvider`. `series-scenes` adds a base background `Sequence` first so
  scenes have a backdrop while they cross-fade.
- Sequence has **no trim props** (those are on `Video`/`Audio`) — Callout on the
  sequencing page so the distinction is explicit.
- Forward links to not-yet-authored pages (`video`/`audio`/`shapes`/`transitions`)
  left in place, matching how Step 1 linked forward to these pages before authoring.

## Step 3 — Drawing

Pages: `shapes`, `text`, `images`.

- [ ] **shapes** — the `FlexShape` wrapper story; full shape list (`Rect`,
      `Circle`, `Ellipse`, `Line`, `Arrow`, `Star`, `Ring`, `Arc`, `Wedge`,
      `RegularPolygon`, `Path`, `TextPath`, `Sprite`); Konva config parity
      (link out); the "delete absent width/height" gotcha Callout; `getSelfRect`
      intrinsic size.
- [ ] **text** — `Text` config; typewriter; animated `highlights.progress`;
      `fitText`/`wrap`/`maxLines`.
- [ ] **images** — `src`, `objectFit`, `objectPosition`, `cornerRadius`,
      custom `loader` (note the server-render asset-URL helper).
- [ ] Demos: `shapes-gallery`, `shape-morph`, `typewriter`, `text-highlight`
      (ported), `image-objectfit`.

## Step 4 — Layout ✅ DONE

Pages: `flex`, `block`.

- [x] **flex** — flexbox via flexily; `flexDirection`/`justifyContent`/
      `alignItems`/`gap`/`padding`; child props
      (`flexGrow`/`flexShrink`/`flexBasis`/`alignSelf`/`margin`); px/`%` sizing;
      reflows every tick - animate `gap`/size and watch layout follow. Container
      and child props each in a table; `flex-reflow` + `flex-layout` demos.
- [x] **block** ("box") — styled flex container: `background` (solid +
      `{gradient}` linear/radial), `borderSize`/`borderColor`/`borderStyle`,
      `cornerRadius`, `shadow` (`{color,blur,offsetX,offsetY,opacity}`); built a
      card with `block-card`. Note that Block is a Flex, so every flex prop
      applies.
- [x] Demos: `flex-layout` (ported, reused), `flex-reflow`, `block-card`. *Both
      new demos run at `fps: 60`, loop, core wrappers only, dark-bg Sequence
      first.*
- [x] Nav: inserted `---Layout---` group + `flex`/`block` slugs after Drawing in
      `content/docs/meta.json`.
- [x] Verified: docs `pnpm build` passes; both pages load and every demo paints
      (checked the per-layer Konva canvases directly - flex-reflow's four chips
      with the grow-chip widened, block-card's gradient/border/shadow/text).

Verified facts / corrections caught while building:
- **One Sequence, one Konva layer, one canvas.** Each `Sequence` is its own
  `Konva.Layer`, so the player has one `<canvas>` per active sequence. Sampling
  "the canvas" only sees the bottom layer; the content layer is a separate
  canvas. (Bit verification, not the page - noting so the next author doesn't
  chase a "blank" demo that is actually painting on layer 2.)
- **Layout roots must be DIRECT children of the Sequence.** `Sequence._apply`
  calls `_kmComputeLayout()` only on its own top-level children
  (`sequence.ts:73-75`), after running updaters. So the `Flex`/`Block` you
  animate must sit directly under the Sequence that holds its `register` updater,
  or it never reflows. Both demos put the layout root directly in its Sequence.
- **`flexWidth`/`flexHeight`, not `.width()`.** Confirmed live: the Flex/Block
  constructor stores `width`/`height` config as the `flexWidth`/`flexHeight`
  attrs (`flex.ts:54-55`, `block.ts:115-116`); `layoutRoot` reads those, not
  Konva's bbox `width()`. Animate the container's own size with
  `setAttrs({ flexWidth })`. Documented as a `<Callout type="warn">` on the flex
  page.
- **Block always sizes to content.** `computeLayout()` calls `layoutRoot(this,
  true)` so the background rect is never 0x0 even when the Block hugs its
  children. The bg rect is `moveToBottom()`'d so children render on top
  (`block.ts:129-167`).
- Forward links from the Drawing pages (`shapes.mdx`, `text.mdx`) to `/docs/flex`
  and `/docs/block` now resolve.

## Step 5 — Media ✅ DONE

Pages: `video`, `audio`.

- [x] **video** — `Video({src, trimBefore/After, loop, muted, volume,
      objectFit})`; frame-clock sync; multi-clip sync; server-render Callout
      (skia backend). Props table + `setVolume`/`setMuted`/`setPlaybackRate`;
      `objectFit`/`objectPosition` cross-link to `images`.
- [x] **audio** — `Audio({src, name, volume, muted, trim…})`; `comp.mixer`
      master volume/mute; layering music + voiceover; frame-driven volume
      automation via `register` + `interpolate` + `setVolume`. Dropped the
      "markers mention" - there is no public marker API (see below).
- [x] Demos: `video-sync` (ported), `audio-mixer` (ported). Both already lived in
      `src/demos/` from Step 0; this step was page prose + nav + verification.
- [x] Nav: inserted `---Media---` group + `video`/`audio` slugs after Layout in
      `content/docs/meta.json`.
- [x] Verified: docs `pnpm build` passes; both pages serve 200 and render; the
      `video-sync` clip paints a real video frame on its content layer (canvas 1
      colored, base layer separate); the `audio-mixer` visual layer animates
      between frames (Music A ducked to 25%, Voice at 100% at frame 300). New nav
      group + prev/next links resolve.

Verified facts / corrections caught while building:
- **No public marker API.** `media/media-marker.ts` (`MEDIA_MARK`/`VIDEO_MARK`/
  `AUDIO_MARK`/`TICK_MARK`) is internal node-discovery only - nothing is exported
  from the barrel. The planned "markers mention" was dropped; media nodes are
  auto-discovered by the `Sequence`, no manual registration.
- **`.volume`/`.muted` are read-only signals.** Both `Video` and `Audio` expose
  `volume`/`muted` as `ReadonlySignal`s; set them with `setVolume(0..1)` /
  `setMuted(bool)` (and `setPlaybackRate(n)`), never by assignment. Documented as
  a `<Callout type="warn">` on the audio page.
- **`Video` reuses `Image`'s fit logic.** `objectFit`/`objectPosition` are the
  exact `layout/image.ts` controls, so the video page links to `images` rather
  than re-documenting the fit modes.
- **Effective level = master × track.** `comp.mixer` scales every channel; a
  track is silent when either the master or the track is muted. The audio demo
  drives intrinsic levels from the frame and the mixer applies master on top.
- **Audio renders nothing.** An `Audio` node is `visible:false`/`listening:false`,
  so its `Sequence` produces a blank canvas; the demo's visuals live on a
  separate base `Sequence` (one Konva layer/canvas per sequence - relevant when
  verifying, sample all canvases not just the first).
- **Audio sequences added before the visual layer.** `audio-mixer` adds the five
  audio sequences first, then the base visual `Sequence` last so visuals paint on
  top (Konva layer order = add order).
- Forward links to not-yet-authored `/docs/rendering` and `/docs/rendering-media`
  left in place, matching how earlier steps linked forward before authoring.

## Step 6 — Transitions

Page: `transitions`.

- [ ] **transitions** — `TransitionSeries({composition})`; `.scene(...)` +
      `.transition({presentation, timing})`; geometric presentations (`fade`,
      `slide`, `wipe`, `clockWipe`, `iris`, `flip`, `bookFlip`); `linearTiming`
      vs `springTiming`; Callout on shader presentations (`dissolve`,
      `crosswarp`, …) and the renderer `./gl` path.
- [ ] Demos: `transition-fade`, `transition-slide`, `transition-spring`.

## Step 7 — Player

Page: `player` (rewrite/expand).

- [ ] **player** — `<km-player>` attributes (`src`, `controls`, `loop`,
      `autoplay`, `muted`, `volume`, `playbackrate`, `initialframe`); the `src`
      module contract (a `?url` module the player `import()`s); composing custom
      controls (`KmPlayerPlayButton`, `Progress`, `SoundControl`,
      `createDefaultControls`); imperative API + events (`time`, `play`,
      `seekTo`, `setProps`); the standalone `<script type="module">` CDN path.
- [ ] Demos: `orbit` (exists), `player-custom-controls`.

## Step 8 — Rendering

Pages: `rendering`, `rendering-media`. The renderer is Node/headless and
**API-only (no CLI)** — these pages use annotated code blocks, not
`<km-player>` previews. Where helpful, embed the resulting MP4 as a `<video>`.

- [ ] **rendering** — "your composition → MP4 file." `import
      "@konva-motion/renderer/register"` (= `setupServerRendering()` at import)
      before building the comp; `renderComposition(comp, {output, fps,
      resolution, fit, quality, range, mute, onProgress, signal})`; quality
      presets (`low`/`medium`/`high`/`max`); `RenderResult`; progress +
      `AbortSignal`. Single frame with `renderStill({frame, type})`; raw frames
      with the `renderFrames()` async generator. Call out the no-CLI note (run
      a `.ts` file with `tsx`/`node`).
- [ ] **rendering-media** — server-side assets & advanced: the `mediaSrc`
      asset-URL helper (Vite URL → fs path) so media comps render headlessly;
      custom image/video source factories and `setupServerRendering({video,
      fonts, videoDecodeCap})`; font registration (`registerFonts`); how audio
      is collected + muxed (`collectAudioTrack`, ffmpeg mix); shader (Tier B)
      transitions via `import "@konva-motion/renderer/gl"` + the headless-gl
      optional dep and `fade()` fallback when `gl` is absent.
- [ ] No live demos — annotated code + optional embedded result `<video>`.

## Step 9 — Studio (Preview)

Pages: `studio-overview`, `studio-setup`, `studio-interface`,
`studio-rendering`. Preview banner on each.

- [ ] **studio-overview** — what it is; when to use; screenshot/embed;
      `0.0.0` preview Callout.
- [ ] **studio-setup** — `<Studio registry render>`; a registry entry
      (`{id, title, propsSchema, render}`); the `kf` DSL (`text`/`number`/
      `color`/`select`/`boolean`/`object`/`array`/`divider`) → props form;
      `selectedId`/`onNavigate` routing.
- [ ] **studio-interface** — compound parts: `Studio.Stage`, `Studio.Timeline`
      (`TimelineHeader`/`Transport`/`Scrubber`/`LayeredBody`), `Studio.Panel`/
      Tabs (inspector), `Studio.Header`/`Zoom`; hooks (`useStudio`,
      `useComposition`, `useSignalValue`, `usePlayback`, `useShortcuts`).
- [ ] **studio-rendering** — `Studio.RenderDialog`, `Studio.ExportFrameDialog`,
      `Studio.RenderQueue`; render backend prop; how the in-app render queue
      submits to `@konva-motion/renderer` (link back to the Rendering section).
- [ ] Assets: Studio is React (not a `<km-player>` composition) — use static
      screenshots for v1; revisit live React embed/iframe of demo2 later.

## Step 10 — Verify & ship

- [ ] `pnpm build` the docs.
- [ ] Load every page; confirm each `<km-player>` demo imports and plays (use
      preview tools); fix failures.
- [ ] Screenshot proof of key pages.
- [ ] Update `doc/README.md` if any public API examples changed.

---

# Demo library map

Authoring contract: `src/demos/<name>.ts`, default export = `Composition` or
`() => Composition`. Live preview via `?url`; displayed source via `?raw`.

| Page | Demo files |
| --- | --- |
| introduction | `hero` |
| installation | `first-tween` |
| core-concepts | `frame-clock`, `range-gate` |
| best-practices | `wrapper-vs-raw` |
| interpolation | `interpolate-basics`, `color-shift` |
| easing | `easing-compare` |
| sequencing | `stagger`, `series-scenes` |
| shapes | `shapes-gallery`, `shape-morph` |
| text | `typewriter`, `text-highlight`¹ |
| images | `image-objectfit` |
| flex | `flex-layout`¹, `flex-reflow` |
| block | `block-card` |
| video | `video-sync`¹ |
| audio | `audio-mixer`¹ |
| transitions | `transition-fade`, `transition-slide`, `transition-spring` |
| player | `orbit` (exists), `player-custom-controls` |
| studio | screenshots + one registry snippet |

¹ Already exists as a core demo — port into `src/demos/`.

# Open questions

- Fast-track the `introduction` + `installation` rewrite as a standalone fix
  (current published docs teach a non-existent API), or bundle into the full
  build?
- Adopt the `<Demo name="...">` single-prop helper, or keep explicit
  two-import-per-page?
- Studio live previews: static screenshots for v1 (default), or invest in an
  embedded React island / iframe of demo2?
