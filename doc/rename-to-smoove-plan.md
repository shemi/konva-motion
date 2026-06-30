# Rebrand plan: `konva-motion` → `smoove`

The project is rebranding from **konva-motion** to **smoove** — a unified
change that covers both the **mechanical rename** (package scope, identifiers,
tags, files) *and* the **visual identity** (palette, type, logo marks, product
look). It ships as **one coordinated rebrand** on a single branch; the phases
below are review checkpoints, and Phase 9 is the integration gate.

- **npm org:** `smoove` (scope `@smoove/*`) — secured
- **GitHub org/repo:** `smoove-dev/smoove` — secured
- **Domain:** `smoove.dev` — secured

**Decisions locked in:**

- `km-` prefix (web-component tag + sub-element tags + CSS classes) → `smoove-`
- Full identifier rename: `KmPlayer`→`SmoovePlayer`, `KmStudio`→`SmooveStudio`,
  `KonvaMotion`→`Smoove`, `window.KonvaMotion`→`window.Smoove`
- Rename asset files, source filenames, and the working folder too
- **Per-package styling** — each package hard-codes brand values; there is **no**
  shared token module. (Values are listed once in [§ Brand reference](#brand-reference)
  for copy-paste consistency.)
- **Animated logo mark is deferred** — static marks only in this pass.

**Scale:** ~907 substring hits across ~250 files for the mechanical rename
(mostly scoped find-replace), plus the design repaint of docs / player / studio.

> **Guiding rule:** only rebrand the `konva-motion` tokens. **Leave the
> underlying Konva library untouched** — the `konva` peerDep, `Konva.Stage`,
> `window.Konva`, and `import Konva from "konva"` all stay. Replace
> `konva-motion` / `@konva-motion` / `KonvaMotion` / `km-`, never bare
> `konva` / `Konva`.

---

## Design assets (source of truth)

The visual identity was handed off as an HTML/CSS/JS design bundle from Claude
Design. **These prototypes are the spec for the repaint phases** — recreate
their visual output (colors, type, spacing, the mark), don't copy their internal
structure.

- **Bundle (committed):** [`doc/color-palette-and-product-design.zip`](./color-palette-and-product-design.zip)
- **Extracted:** `doc/_design_extracted/color-palette-and-product-design/`
  - `README.md` — handoff notes (read first)
  - `project/Smoove Brand Brief.dc.html` — **the master spec**: full palette with
    hex + usage, gradient rules, typefaces, mark variants, do/don't, and the
    "How products should look" + "In practice" (studio) product mocks
  - `project/Smoove Lockup.dc.html` — lockups, size ladder (54/34/21/14px),
    backgrounds (white / cream / ink / gradient knockout), stacked lockup
  - `project/Smoove Logo Marks.dc.html` — mark exploration gallery
  - `project/Smoove Logo Animation.dc.html` — animated mark *(deferred)*
  - `project/smoove-mark.svg` — **primary** mark, gradient
  - `project/smoove-mark-dark.svg` — one-color dark (ink on paper)
  - `project/smoove-mark-light.svg` — one-color light (paper on ink)
  - `project/smoove-mark-animated.svg` — animated mark *(deferred)*
  - `project/screenshots/` — reference renders

Companion doc: [`doc/smoove-brand-brief.md`](./smoove-brand-brief.md) — the
*why/feel* (voice, pillars, positioning) behind the visuals above.

---

## Brand reference

Copy these values into each package (per-package styling — no shared module).

**Palette**

| Token | Hex | Role |
| --- | --- | --- |
| Coral | `#FF5640` | Primary. Gradient **start**. Buttons, big moments. (Dark text `#E03A28` for small labels.) |
| Mint | `#15CDA8` | Gradient **end** / the "trail". Live states, success. (Dark `#0E9E84`.) |
| Sunshine | `#FFC23C` | The wink. Badges, the keyframe dot. **Always dark text on top.** (Dark `#B07A00`.) |
| Grape Ink | `#261733` | Type, wordmark, dark surfaces. Warm, never flat black. |
| Cream | `#FFF6EC` | Default canvas. Warm, not clinical. |
| Paper White | `#FFFFFF` | Working surfaces. |

**The Smoove gradient:** `linear-gradient(102deg, #FF5640, #15CDA8)`. Hero
accent only — the mark, key accents, **one** focal surface per screen. Never a
full-page wash. Gradient on text only at display size.

**Ratio of use:** cream + grape carry ~90%; coral/mint/sunshine are the ~10%
spark.

**Ink-surface support tints** (from the mocks): panel `#34204A`, border
`#43305A`; light borders `#ECEAF0` / `#E6E2D8` / `#F0EEE7`; body text `#3B3947`,
muted `#6A6678` / `#9A96A6` / `#8E8AA0`.

**Type** (all free on Google Fonts):

| Face | Weights | Job |
| --- | --- | --- |
| **Comfortaa** | 400–700 | Display & wordmark (rounded geometric; headlines, logo) |
| **Hanken Grotesk** | 400–700 | Body & interface (paragraphs, labels, buttons) |
| **JetBrains Mono** | 400–600 | Code & data (samples, timestamps, metrics) |

**Surfaces:** rounded corners (~10–22px, default 16), warm paper backgrounds,
crisp 1px borders, restrained shadows, generous whitespace. The gradient is the
only loud thing on screen.

---

# Group A — Mechanical rename

Foundation. The build must stay green at the end of each phase. The hard guard
throughout: **never touch bare `konva` / `Konva` / `window.Konva`** — only the
`konva-motion` family of tokens.

| Old | New |
| --- | --- |
| `konva-motion` (root) | `smoove` |
| `@konva-motion/core` | `@smoove/core` |
| `@konva-motion/player` | `@smoove/player` |
| `@konva-motion/renderer` | `@smoove/renderer` |
| `@konva-motion/transitions` | `@smoove/transitions` |
| `@konva-motion/studio` | `@smoove/studio` |
| `@konva-motion/timeline` | `@smoove/timeline` |
| `@konva-motion/docs` | `@smoove/docs` |
| `@konva-motion/google-fonts` | `@smoove/google-fonts` |
| `@konva-motion/vite` | `@smoove/vite` |

## Phase 1 — Package identity & workspace relink ✅ DONE (2026-06-30)

- Rename `"name"` in **every `package.json`**: root → `smoove`, packages →
  `@smoove/*` per the table above.
- Update all `dependencies` / `peerDependencies` / `devDependencies` entries that
  reference the old scope (the `workspace:*` cross-deps).
- Add the (currently absent) `repository`, `homepage`, and `bugs` fields to the
  publishable `package.json`s, pointing at `github.com/smoove-dev/smoove` and
  `https://smoove.dev`.
- **Do not hand-edit `pnpm-lock.yaml`** (20 hits) — it regenerates.

**Verify:** `pnpm install` relinks the workspace under `@smoove/*` and
regenerates the lockfile with no errors.

**Status / notes:**

- All 10 `package.json` names renamed (`smoove`, `@smoove/*`); all `workspace:*`
  cross-deps repointed (player, renderer, transitions, studio, docs, demo).
- `repository` / `homepage` / `bugs` added to the **7 publishable** packages
  (`core`, `player`, `renderer`, `transitions`, `studio`, `google-fonts`,
  `vite`). Skipped the **3 private** ones — root `smoove`, `@smoove/docs`,
  `demo` — since they aren't published.
- Also swapped the product name inside each package's `description` field
  (`konva-motion` → `smoove`) — it's package-identity metadata sitting on the
  same lines. Bare `konva` / `Konva` left untouched per the guard.
- **No `timeline` package on disk.** The plan's table + Group A list
  `@konva-motion/timeline`, but `packages/timeline` does not exist (only a
  placeholder mention in CLAUDE.md). Nothing to rename; if it's ever scaffolded
  it should be created directly as `@smoove/timeline`.
- `pnpm install` (v10.33.4) succeeded — lockfile regenerated, **0** residual
  `@konva-motion` entries, 20 `@smoove/` entries, all 9 projects relink. (pnpm
  flagged an optional upgrade to 11.9.0 — not applied.)
- ⚠️ **`pnpm build` will fail until Phase 2** — source files still
  `import … from "@konva-motion/*"`. That's expected; Phase 1's gate is only
  `pnpm install`. Don't run the build as a Phase 1 check.

## Phase 2 — Imports, identifiers & browser globals ✅ DONE (2026-06-30)

- Every `import … from "@konva-motion/*"` → `@smoove/*` across `packages/*`,
  `demo/`, and docs (193 `core` imports alone).
- `KmPlayer` → `SmoovePlayer` (~50), `KmStudio` → `SmooveStudio` (~21).
- `KonvaMotion` → `Smoove` (~19), incl. the `import * as KonvaMotion` alias in
  `standalone.ts`.
- `window.KonvaMotion` → `window.Smoove`, `window.KonvaMotionPlayer` →
  `window.SmoovePlayer` (standalone bundle + its JSDoc / `vite.config` comments).
  **`window.Konva` stays.**

**Verify:** `pnpm build` compiles all packages (incl. player's Vite build +
standalone bundle).

**Status / notes:**

- Applied to **compiled source only** — every tracked `.ts`/`.tsx` under
  `packages/*` + `demo/` (incl. `vite.config.ts`, `react-router.config.ts`,
  `scripts/`, `examples/`, and source `*.d.ts`). **243 source files changed**;
  `dist/` + `demo/build/` are build artifacts (regenerated by the gate) and
  `.mdx`/`.md` prose is **deferred to Phase 4** — so 29 `.mdx` files still hold
  `@konva-motion` / `konvaMotion()` references on purpose.
- Replacement order (longest-token-first, so `Konva` is never touched):
  `@konva-motion/` → `@smoove/`; `KonvaMotion` → `Smoove`; `\bkonvaMotion\b` →
  `smoove`; `konva-motion` → `smoove`; then the `Km*` identifiers. Used `perl`
  (not BSD `sed`) for reliable `\b`, and escaped `@` in the perl regex.
- `KonvaMotion` → `Smoove` as a **substring** cleanly swept the non-obvious
  cases the plan didn't enumerate: the internal stage marker
  `__KonvaMotionComposition` → `__SmooveComposition` (core
  `composition.ts`/`environment.ts`), the vite-plugin type `KonvaMotionOptions`
  → `SmooveOptions`, and `window.KonvaMotionPlayer` → `window.SmoovePlayer`.
- **Vite plugin public API renamed** (not spelled out in the plan, but required —
  `konvaMotion` is the camelCase brand token and the Phase 4 gate forbids
  residual `konva-motion`): exported fn `konvaMotion()` → `smoove()`, interface
  `KonvaMotionOptions` → `SmooveOptions`, and the runtime plugin `name:
  "konva-motion"` → `"smoove"` in `packages/vite/src/index.ts`. The legacy
  `reactMotionStudio` alias is untouched (not a brand token); its RHS now points
  at `smoove`. **mdx samples + the `default`/named export shape still need a
  prose pass in Phase 4** for the `konvaMotion()` call sites in
  `vite-plugin/*.mdx`.
- **Extra `Km*` identifiers renamed beyond the plan's `KmPlayer`/`KmStudio`** to
  honor the locked "full identifier rename" decision and avoid a half-done look:
  `KmControl` → `SmooveControl` (player base class), `KmField` → `SmooveField`,
  `KmContainer` → `SmooveContainer`, `KmSchema` → `SmooveSchema` (player/studio
  internals). ⚠️ These are **not** caught by the Phase 4/9 grep gate (which only
  scans `km-`, lowercase) — they were found by hand. **Guard:** the
  `@smoove/google-fonts` font-data files (`src/fonts/*.ts`) contain coincidental
  `Km…` substrings (e.g. `KmHesPOL`) that are **not** brand tokens; replacements
  targeted exact brand names only, so those were left intact.
- **CSS class/tag prefix `km-` left for Phase 3** — the case-sensitive subs above
  never match lowercase `km-player` tags or `km-*` classes/tag maps in the
  source `*.d.ts`; only their `@konva-motion` imports changed.
- Removed two **stale, untracked** core dist artifacts from the pre-`engine/`
  layout (`dist/composition.{js,d.ts}`, `dist/environment.{js,d.ts}`, dated May)
  that still held `KonvaMotion` and aren't referenced by the rebuilt barrel
  (which points at `dist/engine/*`). `tsc -b` incremental builds don't prune
  moved-source output; a `dist` wipe would also clear them.
- ✅ `pnpm build` green (exit 0) across all packages incl. player's Vite build +
  the standalone `player.global.js` (now pins `.Smoove`, keeps `window.Konva`).
  Grep confirms **0** residual `@konva-motion` / `KonvaMotion` / `konvaMotion` /
  `konva-motion` and **0** brand `Km*` identifiers in source, with bare
  `konva` / `Konva` imports untouched.

## Phase 3 — Web-component tags, CSS class prefix & file renames

- **Tags:** `<km-player>` → `<smoove-player>` and every sub-element tag
  (`km-player-controls`, `km-player-sound-control`, `km-player-progress`, …) →
  `smoove-player-*`. Includes `customElements.define(...)` registrations and the
  player registry.
- **CSS classes/parts:** `km-studio`, `km-studio-portal`, `km-doc`, `km-spin`,
  `km-default`, `km-boot`, `km-root`, `km-renderer-demo-*` → `smoove-*`. Touches
  `player.css` (41), `studio.css` (32), `demo/src/app.css` (12), docs
  `player.mdx` (50), `doc/studio-design.md` (38). *(Pure rename here; the
  values repaint happens in Group B, keeping the two diffs clean.)*
- **Ambient typings:** `packages/studio/src/km-player-element.d.ts` and
  `packages/docs/src/km-player.d.ts` (tag-name → interface maps).
- **Source file renames** (+ update their importers):
  - `packages/player/src/km-player.ts` → `smoove-player.ts`
  - `packages/studio/src/km-player-element.d.ts` → `smoove-player-element.d.ts`
  - `packages/docs/src/km-player.d.ts` → `smoove-player.d.ts`

**Verify:** `pnpm build` + `pnpm dev` smoke — the demo renders `<smoove-player>`.

## Phase 4 — Folder, docs & prose sweep

- Rename the working folder `konva-motion/` → `smoove/` (local; harmless to
  tooling).
- Sweep prose for `konva-motion` references: `README.md`, `CLAUDE.md`, all
  `doc/*.md`, `packages/docs/content/docs/*.mdx`, and the
  `docs/superpowers/specs/` + `memory/` notes.
- Update the `unpkg` / `jsdelivr` URLs and `@konva-motion/player` references in
  `packages/docs/content/docs/player.mdx`. The `exports` subpaths (`./standalone`,
  `./styles.css`, …) keep their relative names.
- Update prose links/badges to the new org + domain.

**Verify:** `pnpm check` (Biome) passes; `grep -rn` finds **zero** residual
`konva-motion` / `@konva-motion` / `KonvaMotion` / `km-` tokens (excluding bare
`konva` / `Konva`).

---

# Group B — Visual identity

Per-package styling, landing on the same branch. Consumes the
[design assets](#design-assets-source-of-truth) above; values from
[§ Brand reference](#brand-reference).

## Phase 5 — Brand assets (logo marks + favicon)

- Replace the old `konva-motion-{mark-white,mark-black,mark-gradient,…}.svg`
  assets with the new **edge-dot sunshine mark** — four timeline bars tapering to
  a play triangle, with the sunshine keyframe dot just past the last bar:
  - `smoove-mark.svg` (primary, gradient) ← `project/smoove-mark.svg`
  - `smoove-mark-dark.svg` (ink on paper) ← `project/smoove-mark-dark.svg`
  - `smoove-mark-light.svg` (paper on ink) ← `project/smoove-mark-light.svg`
- Regenerate `packages/docs/public/favicon.svg` from the mark.
- Update the brand component, `README.md`, and docs references to the new files.
- **Deferred:** `project/smoove-mark-animated.svg` / the animated lockup.

**Verify:** marks render crisp at favicon size through hero size; gradient,
dark, and light variants each keep the sunshine dot.

## Phase 6 — Brand fonts

- Wire the three faces in where text renders: **Comfortaa** (display/wordmark),
  **Hanken Grotesk** (body/UI), **JetBrains Mono** (code) — docs first, then
  player/studio chrome.
- Load via Google Fonts (`Comfortaa:wght@400..700`,
  `Hanken+Grotesk:wght@400..700`, `JetBrains+Mono:wght@400..600`).
- Check whether `@smoove/google-fonts` needs these registered for server-side
  rendering of compositions that use them.

**Verify:** docs + demo show Comfortaa headings, Hanken body, mono code blocks;
no FOUT regressions; SSR render (if applicable) embeds the faces.

## Phase 7 — Docs site repaint

Convert the existing KmStudio look to the Smoove system (`@smoove/docs`):

- Apply the palette + the Smoove gradient as a single focal accent per view;
  honor the ~90/10 calm-vs-bright ratio.
- New lockup in the header (mark + Comfortaa wordmark), gradient hero treatment.
- Rounded surfaces, warm paper / cream backgrounds, crisp 1px borders.
- Mono for all code/data; gradient on display type only.

**Verify:** docs build + visual smoke against `Smoove Brand Brief.dc.html` and
`Smoove Lockup.dc.html` (palette, type, lockup, gradient usage).

## Phase 8 — Player, Studio & demo repaint

- **`player.css`:** brand colors; gradient on the render/play focal accent;
  rounded control surfaces; restrained shadows.
- **`studio.css`:** match the product-look mock — warm paper frame → white
  working surface → **one** gradient focal moment (the render button + animated
  object); mono everywhere data lives; Comfortaa wordmark, Hanken UI. Reference:
  the "In practice" studio card in `Smoove Brand Brief.dc.html`.
- **`demo/src/app.css`:** inherits the player styling; align surrounding chrome
  to the palette.

**Verify:** `pnpm dev` — player controls, studio shell, and demo match the mock;
one gradient focal moment per surface.

## Phase 9 — Integration gate (the "one big rebrand")

1. `pnpm install` — clean relink under `@smoove/*`.
2. `pnpm build` — every package green (incl. player Vite + standalone bundle).
3. `pnpm check` — Biome lint/format.
4. `pnpm dev` — full smoke: `<smoove-player>` renders, new palette + fonts +
   mark present across docs / player / studio / demo.
5. Residual-token grep clean (`konva-motion` / `@konva-motion` / `KonvaMotion` /
   `km-`, excluding bare `konva` / `Konva`).
6. External links resolve: `github.com/smoove-dev/smoove`, `https://smoove.dev`,
   unpkg/jsdelivr standalone URLs.

---

## Suggested execution order

Group A in order (1 → 4), then Group B (5 → 8), then the Phase 9 gate:

`package.json` names → `pnpm install` → imports/deps → identifiers & globals →
tags/CSS/file renames → folder + docs/prose → **then** brand assets → fonts →
docs repaint → player/studio/demo repaint → integration gate.
