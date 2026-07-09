# Fraxel documentation site

This is a Mintlify documentation site for the **Fraxel** 2D game engine.

## Project structure

- `docs.json` — main Mintlify config (navigation, theme, colors)
- `getting-started/` — intro, installation, core concepts, essentials
- `guides/` — rendering, gameplay, assets, advanced patterns
- `api/` — nodes, hooks, engine, JSX components
- `advanced/` — internals, systems, performance
- `.atlas-analysis.json` — generated project metadata (do not edit)

## Development

Install the Mintlify CLI globally, then run from repo root:

```bash
npm i -g mint
mint dev
```

Preview at `http://localhost:3000`. Pages are MDX with YAML frontmatter.

## Mintlify MCP

- Edit content/settings via MCP: `https://mcp.mintlify.com`
- Query Mintlify docs via MCP: `https://www.mintlify.com/docs/mcp`
- Install the Mintlify skill: `npx skills add https://mintlify.com/docs`

## Style rules

- Use active voice and second person ("you")
- Sentence case for headings
- Bold for UI elements: Click **Settings**
- Code formatting for file names, commands, paths, and code references
- Keep sentences concise — one idea per sentence

## Content rules

- Document Fraxel's public API, not internal implementation details
- Each API page covers one node/hook/engine module with props, events, examples
- Use `<ParamField>` for props, `<CodeGroup>` for multiple examples
- Reference related pages at the bottom of each API page

## Do not edit

- `api-declarations/` — auto-generated from Fraxel source
- `.atlas-analysis.json` — auto-generated project metadata
- `docs.json` navigation structure — coordinate changes with the team

---

# Fraxel Engine API Reference

The following is extracted from `api-declarations/` type definitions for quick reference when writing docs.

## Core Architecture

**Node system** — Base class `Node<T>` with properties:

- `id` — identifier (string/symbol), used with `child()` for lookup
- `zIndex` — Z-plane position for draw order
- `deltaIncrease` — speed multiplier for node and children
- `gameMode` — pause behavior via `GameMode` enum (`INHERIT`, `PLAYING`, `ALWAYS`, `NEVER`)
- `script` — attach `FraxelScript<T>` for logic separation
- Events: `started`, `updated`, `drawed`, `destroyed`, `zIndexChanged`

**PrimaryNode enum** — All node types:

- `Transform` — spatial container
- `Sprite` — texture rendering with filters
- `Text` — canvas text rendering
- `Geometry` — primitive shapes (rectangle, circle, capsule)
- `Collider` — collision detection
- `RigidBody` — physics simulation
- `Camera` — viewport control
- `AnimationPlayer` — sprite animation
- `AudioPlayer` — audio playback
- `Clickable` — pointer interaction
- `Timer` — countdown
- `RayCast` — ray projection

## Reactivity System

**Core types:**

- `SignalGetter<T>` — reactive getter with dependency tracking (`()`, `.value()`, `.signal`)
- `SignalSetter<T>` — reactive setter that notifies subscribers
- `Reactive<T>` — static value or `SignalGetterLike<T>` (accepts both)

**Hooks:**

- `useSignal<T>(initial)` → `[SignalGetter<T>, SignalSetter<T>]`
- `useComputed<T>(fn)` → `SignalGetter<T>` (derived value)
- `useEffect(fn)` → cleanup function (side effects)
- `useMount(fn)` → cleanup function (mount/unmount lifecycle)
- `useRef<T>()` → mutable ref object **(deprecated: use plain `let` instead)**
- `useUpdate(fn)` → execute once per frame
- `useAction(action)` → object with pressed, justPressed, justUnpressed properties `SignalGetter<boolean>` (action pressed state)
- `useActionAxis(neg, pos)` → `SignalGetter<number>` (-1/0/1)
- `createContext(default)` / `useContext(ctx)` — shared state
- `createTrigger()` / `useTrigger(trigger, callback)` — pub/sub

**Script reactivity:**

- `createSignal<T>(default)` → `SignalGetter<T>` (outside hooks, for FraxelScript)
- `signalSetterFrom(getter)` → `SignalSetter<T>` (extract setter from getter)

**Deprecation warnings:**

- `warnUseRef()` — logs console warning when `useRef` is used (exported from `fraxel/warn/use-ref.js`)

## Event System

Class `Event<T, K>` with methods:

- `on(cb)` → unsubscribe function (normal priority)
- `onFirst(cb)` → void (highest priority)
- `off(cb)` → void (remove specific listener)
- `emit(...params)` → void (fire all callbacks)
- `clean()` → void (remove all listeners)
- `getEventName(base)` → `'on' + PascalCase(base)`

## Input System

Static class `Input`:

- `createAction({key, ctrl?, shift?, alt?})` → `symbol` (unique action ID)
- `getAction(action)` → `InputKey` (binding config)
- `isActionPressed(action)` → boolean
- `justActionPressed(action)` → boolean (first frame only)
- `justActionUnpressed(action)` → boolean (first frame only)
- `isKeyPressed(key, ctrl?, shift?, alt?)` → boolean
- `isJustKeyPressed(key, ...)` → boolean
- `isJustKeyUnpressed(key, ...)` → boolean
- `getKeyAxis(positiveKey, negativeKey)` → -1 | 0 | 1
- `pointerPosition` → `Vector2` (read-only)
- `isPointerPressed` → boolean (read-only)
- Events: `pointerMoved`, `pointerPressed`, `pointerUnpressed`

## Asset System

- `loadTexture(url)` → `Promise<symbol>` (cached)
- `unloadTexture(id)` → void
- `loadSound(url)` → `Promise<symbol>` (cached)
- `unloadSound(id)` → void
- `loadBatch(loaders, options)` → batch loader with progress
- `loadBatchAsset(type, urls)` → typed batch loader

## Collision System

- **Broadphase**: `SpatialHash` (spatial hashing)
- **Narrowphase**: `Narrowphase` detector with shapes:
  - `shapes.rectangle(width, height)`
  - `shapes.circle(radius)`
  - `shapes.capsule(length, radius, direction?)`
- `PhysicsSystem` — gravity, simulation
- `resolveCollision()` / `computeOverlap()` — collision resolution
- `CollisionEmitter` — collision events

## Math Utilities

- `Vector2` / `vector2(x, y)` — 2D vector class
- `bounds(x, y, w, h)` — axis-aligned bounding box
- `Color` / `ColorLike` — RGBA color `[r, g, b, a]` (0-1 range)

## FraxelScript

Abstract class for logic-rending separation:

- `me` — attached node (read-only, throws if accessed before `init()`)
- `setup()` — override for initialization (register events, state)
- `connect(eventName, callback)` — type-safe node event subscription
- Supports `createSignal()` and `signalSetterFrom()` for reactive state

## JSX Components

- `<Game width height defaultScene>` — root component
- `<Scene name component>` — scene definition
- `<List array itemKey>` — reactive list rendering

## Core Game API

Static class `Game`:

- `setup({width, height, root, pauseOnBlur?, theme?})` — initialize canvas
- `play()` — start game loop
- `pause()` — pause game
- `destroy()` — stop loop, cleanup
- `paused` → `SignalGetter<boolean>` (reactive)
- `sceneManager` — manage scenes
- Event: `blurred` (on window blur)

---

# Documentation Audit (2026-07-09)

This section documents known issues found by comparing docs with `api-declarations/`.

## Known documentation gaps

### Missing API reference pages

- `animationFromSheet()` — exported from `fraxel`, used in examples, but no dedicated API page
- `keyframesFromSheet()` — exported from `fraxel`, no API page
- `tween()` / `tweenValue()` — exported from `fraxel`, no API page
- `easing` functions — exported from `fraxel`, no API page
- `getAudioContext()` — exported from `fraxel`, no API page
- `getParentScript()` — used in `fraxel-script.mdx` but no API page
- `getTexture()` / `getSound()` — documented in assets.mdx but no dedicated section

### Missing type documentation

- `SignalGetter<T>` — core reactivity type, no explicit docs page
- `SignalSetter<T>` — core reactivity type, no explicit docs page
- `SignalGetterLike<T>` — used in props, no docs
- `Reactive<T>` — used in most node props, no docs
- `Fun<T>` — callback type for events, no docs
- `EventName<T>` — utility type, no docs

### Missing node properties in docs

- `Node2D.globalPosition` — getter/setter, not documented
- `Node.globalZIndex` — getter/setter, not documented
- `Node.globalDeltaIncrease` — getter/setter, not documented
- `Node.isStarted` — boolean property, not documented
- `Node.isDestroyed` — boolean property, not documented
- `Node.parent` — read-only getter, not documented
- `Node.children` — read-only getter, not documented

### Inconsistencies found

- `Sprite.modulate` — docs say `Color | SignalGetter<Color>`, type is `Reactive<ColorLike>`
- `usePaused` / `useScene` — documented under "Game hooks" but are actually derived hooks from `fraxel/hooks`
- Import paths inconsistent — some examples use `fraxel`, others `fraxel/hooks`, etc.

### Organizational issues

- `advanced/` directory listed in structure but does not exist
- `getAudioContext` imported from `fraxel` in audio-player.mdx but not documented as export
- `animationFromSheet` imported from `fraxel` in animation-player.mdx but not in API reference
