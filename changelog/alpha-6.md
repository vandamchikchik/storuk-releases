---
version: alpha-6
prev: alpha-5
date: 2026-11-21
commit: df373d2684a0318983f8eb94899a523f56755a9f
range: b38bd53..df373d2
breaking: yes
---
## breaking
- [engine] `[engine] version` of a project must equal `alpha-6`; an alpha-5 project is refused; migrate: set `version = "alpha-6"` under `[engine]` in `storuk.toml`
- [project] project file `dalaran.toml` -> `storuk.toml`; a folder with only `dalaran.toml` is refused (`ProjectError::OldFileName`), no automatic migration; migrate: rename `dalaran.toml` to `storuk.toml`
- [script] `model_spawn(..)` removed; `model("..")` -> `asset("..")`, `ui("..")` removed (`asset("..")` names any baked entity list: glb model, html page, other); `Request[Model]` -> `Request[Asset]`; `asset("x")` alone asks for nothing; migrate: declare `component Wait: src: Request[Asset]`, `spawn Wait(src = request(asset("x")))` in `Startup`, and in a `Simulation` system `gate query waiting: entity, read Wait` + `if w.src.ready(): spawn w.src; despawn e` (as in the `storuk new` template)
- [script] text literal `text("..")` -> `label("..")`; Engine API `answer.text.add` / `answer_text_add` -> `answer.label.add` / `answer_label_add`; migrate: rename the literal and the entry
- [rust] project ABI `ABI_VERSION` 9 (group systems, Rust events, typed extensions); facade `prefab_script` re-export and `dalaran_prefab_script` removed; `dalaran_prefab::instantiate`/`instantiate_into` -> `spawn`/`spawn_vec` with `Spawn` (outer parent, root extras, copies); migrate: rebuild the project library against alpha-6, call `spawn`
- [pipeline] `converters/sequences/glb.toml` runs only the tables it lists: fonts, CSS and HTML bake only with `[FontBake]`, `[FontBakeOtf]`, `[StyleSheet]`, `[TemplateBake]` tables; migrate: copy these tables from the alpha-6 template `converters/sequences/glb.toml` to use `in/*.html`, `in/*.css`, `in/fonts/*.ttf|otf`
- [format] baked data formats changed (script code `FORMAT` 23, entity lists, fonts, style sheets, templates, `Request`/`Asset` field kinds); migrate: re-bake by running the project (`storuk run`)
## added
- [input] agnostic input core (`storuk_input`): devices as entities, actors (any consumer, N layers each), layers as baked entity lists, actions, bindings, links, auto-link and route tables; records with time, lock-free driver sink, driver trait, virtual device
- [input] binding settings and moments: dead zone, curve, scale, press gap, tap, hold, double tap, repeat, toggle, chords; project stages; shapes `INPUT_BUTTON`, `INPUT_AXIS`, `INPUT_VEC2`, `INPUT_VEC3`, `INPUT_WHOLE`, `INPUT_DELTA`, `INPUT_POSITION`
- [input] actions raise entity events with cancel rules (a despawned held action gets a cancelled up); layer switch, live layer replace and rebinding patch tables in place
- [input] rebinding: listen for a control, replace and reset bindings, last device of an actor, hint names via labels; cursor modes `INPUT_CURSOR_FREE|HIDDEN|CAPTURED`, cursor shapes `INPUT_SHAPE_*`, rumble feedback
- [input] drivers: Windows keyboard and mouse (`storuk_input_win11`, Raw Input, coalescing, cursor lock, key names); gamepads (`storuk_input_gilrs`, hot plug, activity, rumble, idle without a timer); both only with a window; project drivers and archetypes in Rust as extensions
- [input] scripts: input components, events, constants and commands (`InputAction`, `InputActor`, `InputBinding`, `InputChange`, ...); consumers can use baked layer sets; game-api `input` domain
- [engine-api] domain `input`: `input.device.*`, `input.control.*`, `input.link.list`, `input.route.*`, `input.binding.set|reset`, `input.actor.reset`, `input.listen.*`, `input.last.get`, `input.label.get`, `input.event.add`, `input.trace.*`, `input.counters.*`
- [ui] HTML and CSS bake to entity lists (`TemplateBake`, `StyleSheet`): `<use template>`, `<slot>`, `with="Comp(field = value)"`; selectors, `@import`, `var()`, `calc()`, pseudo-classes, `@keyframes`, transitions; a source the grammar does not take is refused with file, line, column
- [ui] runtime: every node is an entity (`Node`, `Screen`, `Rect`, markers `Hovered`, `Focused`, `Disabled`, `Hidden`); a `Screen` is one root with its own rect, scale, draw order, `Screen.input` (`UI_INPUT_PASS|BLOCK_BELOW|BLOCK_ALL`), `Screen.target` (window), `Screen.sheet`; several roots per window
- [ui] styles: runtime cascade with invalidation by trigger, parallel matching; code styles `Style.set|get|number|clear`, values `px(..)`, `pct(..)`, `rgb(..)`; classes `Classes.add|remove|toggle|has`
- [ui] layout on Taffy (flex, grid, block) with registered leaf kinds; layout boundaries so a change does not lay out the whole tree
- [ui] text engine (`storuk_text`): bidi, font-chain shaping, Latin/Cyrillic/CJK/Arabic, hit testing, incremental layout and reshape (edits cost the changed lines); fonts as assets (`in/fonts/*.ttf|otf`, MTSDF pages loaded lazily when a glyph is first drawn)
- [ui] text from code: `Text.set_int|set_float|set_label|set_text|clear|get_int|get_float`, `t.append(label(..))`, `t.replace(from, to, label(..))` (byte range; a bad range is `bad_arg`), `Text(label = label(..))` in a spawn makes a text run
- [ui] draw: Vulkan overlay on persistent GPU tables (changed ranges only, one multi-draw-indirect pass, one shader); nodes outside the surface or clip write nothing
- [ui] events on node entities: hover, press, capture, click, double click, wheel, focus, keys, chars, `Resize`, cursor wish; `Tab` and press defaults preventable; hit test by cell; events `on <Event>` in scripts
- [ui] CSS animations: transitions and `@keyframes` on the engine clock, `linear()`, `spring()`, `bounce()`, per-stop easing; events `AnimationStart|Iteration|End|Cancel` reach scripts
- [ui] constants `UI_BUTTON_*`, `UI_MOD_*`, `UI_SOURCE_*`, `UI_TARGET_WINDOW`
- [engine-api] domain `ui`: `ui.node.find`, `ui.rect.get`, `ui.style.get`, `ui.stage.list`, `ui.font.stats`, `ui.asset.stats`, `ui.error.list`
- [script] `asset("..")` literal and `Request[T]` field kind: `spawn r` and `spawn N of r` (copies), `request(asset(..), priority, part)`, `r.ready()`, `r.failed()`, `r.reason()`, `r.set_priority(p)`
- [script] entity events: handlers bound to components, original target on every step; Rust raise/deliver API
- [script] system gate: `gate query` / `gate message` skips a system with nothing to do; `lang.gate.list`
- [script] `name("..")`, `label("..")`, `let e = spawn ..`, batch spawn from text lists, `q.empty()`, host methods and values, tracked writes reaching Rust
- [script] script components in baked entity lists (declaration assets, dynamic binding, type-checked add); omitted fields of a component constructor take the Rust type's `Default` (a type with no `Default` refuses the omission)
- [script] path key component field (`PathKey`, 48 bytes) shared by scripts, Rust, entity text and the Engine API
- [ecs] entity delivery: spawn of a baked list in one call (outer parent, extra root columns, N copies), lists without `Transform` accepted; relationship change sinks in the `bevy_ecs` fork
- [engine-api] domain `world`: `world.entity.*`, `world.child.*`, `world.component.*`, `world.event.list|trigger`, `world.asset.add|get|spawn`
- [engine-api] domain `render`: `render.frame.capture`, `render.frame.get`; domain `lang`: `lang.gate.list`; verbs `stats`, `trigger`
- [cli] `--gamepad-hz <n>`, `--ui-memory-mib <n>`, `--bindless-textures <n>` game words
- [memory] `GrowArray`: growable arrays on virtual memory with owner accounts
- [time] begin-of-frame group, `ClockHandle` clocks for any subsystem
- [release] `install.ps1` in every release: copies to `Program Files\Storuk\<version>`, puts it first on PATH (Machine or User, `%VAR%` kept, other storuk folders off), atomic replace, `-Force`, `-Path`, elevation on demand
- [preset] `input`: two players with exclusive auto-linked actors, console actor, own driver and archetype, layers as baked assets; `ui`: buttons, hover/press/focus/disabled looks, counter, code-built panel, multi-script text
- [template] `storuk new` template places the car and the page `in/main.html` through `asset(..)` requests; ships `in/ui/default.ttf`, `in/ui/base.css`, shaders `ui.slang` and `debug_draw.slang`
## changed
- [script] `Transform` omitted in a script spawn takes identity
- [script] a project component named like an engine game-data component (`Screen`, `Hidden`, ...) shadows the engine one instead of clashing
- [debug-draw] groups reset when a client takes or leaves the process instance
- [shader] consumer pipelines (debug draw, UI) declared by their crates and listed by the host for the converter and `storuk new`
- [threads] one chunking over the bevy pool for physics write-back, style matching, animations and layout; the `threads` settings are dropped
- [game] a request opened before its program is in force fails with a note naming the unbound component
## removed
- [script] `model_spawn`, `model(..)`, `ui(..)`, `Request[Model]`, `Request[Ui]`
- [rust] facade `prefab_script`, `dalaran_prefab_script`, implicit requests
## fixed
- [debug-draw] a group left off or soloed by one client shadowed the next client's meshes
## internal
- [crates] `storuk_text`, `storuk_fonts`, `storuk_font_formats`, `storuk_ui*` (style, css, layout, geometry, draw, vulkan, events, script, api), `storuk_input*`, `storuk_raw_input`, `storuk_extension`, `storuk_asset_types`, `storuk_test_alloc`; the game no longer links the converters
- [ui] cost law: counter tests for allocations of draw, events, text edits and script writes; CPU and GPU acceptance measures
