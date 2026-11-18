---
version: alpha-5
prev: alpha-4
date: 2026-11-18
commit: b38bd537b9fdb93c2614149c00f28599c0992def
range: 1ed6e43..b38bd53
breaking: yes
---
## breaking
- [engine] `[engine] version` of a project must equal `alpha-5`; an alpha-4 project is refused; migrate: set `version = "alpha-5"` under `[engine]` in `dalaran.toml`
- [script] `model_spawn(model, x, y, z)` -> `model_spawn(model, position: vec3, rotation: quat)`; a zero quat is refused, a non-unit one normalized; migrate: `model_spawn(m, vec3(x, y, z), quat())`
- [script] physics commands take whole `vec3`/`quat` instead of flat floats (`shape_*`, `joint_*` frames, `ray_cast`, `sphere_cast`, `sphere_overlap`, `body_*`); `CastHit`/`ContactHit` `point`/`normal` are `vec3`; migrate: pack the numbers with `vec3(..)`/`quat(..)`, read `.x .y .z`
- [script] `Transform` = `{ position: vec3, rotation: quat, scale: vec3 }` (was `px py pz qx qy qz qw sx sy sz`), local to the parent; migrate: `t.position.x` for `t.px`, `t.rotation` for `qx..qw`
- [script] `GlobalTransform` (world pose, `x_axis y_axis z_axis position`) is read-only: `write`, `insert`, `spawn`, `remove` of it are refused; migrate: write `Transform`; rotation from it: `quat_from_mat3(mat3(g.x_axis, g.y_axis, g.z_axis))`
- [rust] `dalaran_prefab::Transform` removed; `Transform`/`GlobalTransform` = `bevy_transform` 0.20.1 (re-exported by `storuk`); `instantiate(world, index, root: Transform)`; migrate: use bevy `Transform` (`translation`, `rotation`, `scale`)
- [rust] project ABI `ABI_VERSION` 7; host commands only through `command!` (`CommandSpec` private, `CommandArg` sealed); physics joint params grouped into `Limit`/`Motor`/`Spring`; migrate: rebuild the project library, declare commands with `command!`
- [run] the game no longer compiles scripts: the asset base compiles them, the game takes codes through streaming; a layout change restarts the client (exit 17); migrate: none, run the project through `storuk run`
## added
- [script] math types `vec2`-`vec4`, `ivec*`, `uvec*`, `quat`, `color`, `mat3`, `mat4` (glam layout): per-lane operators, quat/matrix products, `==`/`!=`, lane read/write, compound assignment; functions `dot cross length normalize distance lerp slerp quat_from_axis_angle quat_from_euler quat_from_mat3 conjugate inverse transpose determinant` and matrix builders; in data, messages, resources, channels, function params/results
- [script] lookups of other entities: `if let a = q.get(e): .. else: ..`; `distinct q` for write lookups inside `par for`; `atomic add|sub|min|max` on shared lookup fields; alias checks (second writing binding, `par for` races)
- [script] compiler warnings (`Code.warnings`): shown by `storuk` wait and in the game's notices; control calls in game code warn
- [script] debug draw functions `debug_<object>[_<sub>]_<verb>` (lines, shapes, lists, splines, polygons, instances, meshes by `MeshRender.mesh`, retained sets `debug_set_begin/end/remove`), style `debug_style_make(depth, width, unit, dash, segments, life, secs)`, constants `DEBUG_DEPTH_*`, `DEBUG_WIDTH_*`, `DEBUG_LIFE_*`, `DEBUG_MESH_*`, `DEBUG_UNIT_*`; callable from systems, `fn`, `par for`
- [script] host command argument `Text` (text literals)
- [debug-draw] overlay over the picture: depth test / x-ray / on top, px or metre widths, dashes, sphere impostors, order-independent transparency; groups with children (enable/disable/toggle/solo); lifetime by tick or seconds of the caller's clock; meshes solid/wire/outline from resident GPU buffers; retained sets by key; zero-copy chunked records with per-chunk culling; Rust `debug_draw!` (feature `debug_draw`, on by default)
- [engine-api] Engine API: one table of entry points (`api.entry.list`, `api.entry.get`, `debug_draw.group.enable|disable|toggle|solo|list`, editor `base.chain`/`layer`/`asset`), typed refusals and warnings; agent scripts run once per frame after settles
- [cli] `storuk api [--editor | --pid <n>] <entry> [<word>...]` and `storuk api [--editor | --pid <n>] exec <script file>`: call an entry or run a script in the running game or the editor service; help topic `api`
- [cli] `storuk run --engine debug`: runs the debug engine (`engine-debug/`, opt-level 2, PDBs); a service of the other engine kind is refused; a project library of the other kind is refused at load (`KindMismatch`)
- [project] `storuk new` writes `knowledge/` into a project: `.storuk/` engine reference (from the Engine API tables and the game API), skills `.claude/skills/storuk-*`, root `AGENTS.md` + `CLAUDE.md`; refreshed when the engine changes
- [release] `engine-debug/` folder (debug engine, hosts, PDBs), `knowledge/`
- [preset] `debug_draw`: every debug draw function from scripts in one view, groups driven through the Engine API
- [preset] `knight_ragdoll_override` chains `ragdoll_100`, `ragdoll_1000`, `ragdoll_5000` (awake knights on a grid)
## changed
- [engine] facade library `storuk.dll` -> `storuk_engine.dll`, one PDB per binary
- [base] an asset whose output bytes did not change keeps its version: readers are not rerun (128-bit output sum)
- [script] an importer reads only its direct import headers; a plain function body change stops one header up
- [script] fn/data/model/path numbers given at install, kept across reloads (known name keeps its number)
- [run] a failing frame (command refusal, script fault, Rust panic in a guarded function, failed task, record on a gone entity) ends the client with exit 18 and the text of the failure
- [transform] propagation runs in the timeline before the frame group and inside the fixed group; drawn poses computed once per frame from interpolated entities
## removed
- [script] disk code cache (`DALARAN_CODE_CACHE`)
## fixed
- [script] JIT stack overflow (`0xC0000005`) of large scripts in release (`ragdoll_5000`): stack probes, one command argument block per function
- [script] hot reload of a code changed by the same manifest that asks a new list
- [anim] orphan rig keeps its GPU mark (frame done never refused)
## internal
- [lang] script code format 15
- [game] `FrameTimes` (intake, schedule, stream, settle, propagate, prepare, render, exec)
- [crates] `storuk_stream_wants` (shared request/release feed of scripts, animation, physics); large crates split by responsibility (core, storage, memory, threads, tasks, service)
