# Architecture

A pnpm monorepo with two published packages and one local demo app.

## Packages

### `@konva-motion/core`

The engine. Exports `Composition`, `Sequence`, signal types, and a tiny
typed emitter. Declares `konva` as a **peer dependency** so the consuming
app pins the version.

Key idea: `Composition extends Konva.Stage` and `Sequence extends Konva.Layer`.
You don't wire the animation engine into Konva — the engine **is** a Stage,
and your sequences **are** Layers. `comp.add(seq)` is just `Konva.Stage.add`.

Engine responsibilities:

- Frame clock: `fps`, `durationInFrames`, current `frame` (readonly signal).
- Playback state: `isPlaying`, `isPaused`, `isStopped` (readonly signals).
- Tick loop driven by `requestAnimationFrame`. Each tick computes the
  integer frame from elapsed time and walks `Composition.getChildren()` —
  any child that is a `Sequence` gets `_apply(frame)`.
- For each active sequence: run its updaters with `localFrame = frame - from`,
  then `seq.batchDraw()`. On enter/leave range, toggle `seq.visible(...)`.
- Emit `"play"` / `"stop"` / `"time"` events with `{ frame, durationInFrames }`.

The `requestAnimationFrame`, `cancelAnimationFrame`, and `performance.now()`
references are read off `globalThis` with fallbacks, so importing core in
Node doesn't throw. `play()` throws a clear error in non-browser
environments; `setFrame(n)` works anywhere for offline / server rendering.

### `@konva-motion/timeline`

Planned home for React UI components that show and control a composition
(scrubber, play button, time display). Currently a placeholder.

## Tick loop

```
rAF → comp._tick(now)
        ├─ frame = floor((now - startWall) / 1000 * fps) + startFrame
        ├─ for each child of the stage:
        │     if child instanceof Sequence:
        │         in range:  visible(true), run updaters(localFrame), batchDraw()
        │         out range: visible(false), batchDraw()
        ├─ emit "time"
        └─ if frame >= last:
              if loop: startFrame = 0, startWall = now, continue
              else:    cancel rAF, isPlaying = false
```

`"time"` fires only when the integer frame changes — so a 30fps composition
on a 120Hz monitor emits 30 times/sec, not 120.

`loop` is a `ReadonlySignal<boolean>` set at construction (`{ loop: true }`)
or at runtime via `comp.setLoop(v)`. When the playhead reaches the last
frame and `loop` is true, the tick resets `_startFrame` to 0 and keeps
ticking — no `"stop"` event, playback just wraps.

## Why two packages

- `core` owns the engine + Konva integration. One install gets you the
  whole engine.
- `timeline` will own React-specific UI. Keeping it separate means a
  consumer that builds their own UI never pays for React.

## Teardown

`Composition.destroy()` overrides `Konva.Node.destroy()` to cancel the
in-flight rAF and clear `isPlaying` before destroying the stage. This
matters when the host app swaps compositions (e.g. the demo's sidebar),
since otherwise a queued rAF callback could fire against a torn-down stage.

## Singleton enforcement

A composition marks its underlying Stage with a `__KonvaMotionComposition`
property. Constructing a second `Composition` over the same Stage throws —
useful if you ever wrap an existing stage. Use `getComposition(stage)` to
read the marker.
