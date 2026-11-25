---
version: alpha-7
prev: alpha-6
date: 2026-11-25
commit: 1b1f179c737bc485db5f58542be1fed404e46f8f
range: df373d2..1b1f179
breaking: yes
---
## breaking
- [engine] `[engine] version` of a project must equal `alpha-7`; an alpha-6 project is refused (`ProjectError::Engine`); the launcher `storuk.exe` hands a project to the install of its version, and an alpha-6 install is not in its index (no `--identity`); migrate: `storuk engine set alpha-7` in the project folder, or set `version = "alpha-7"` under `[engine]` in `storuk.toml`
- [chain] a chain is pinned on its parent (`chains/<c>/base.toml`, written on first start when missing): parent changes after the pin no longer reach it; `storuk chain merge` is refused while the parent changed what the chain changed; migrate: `storuk chain pull <c>` to take parent changes, before `chain merge`
- [pipeline] a sequence row with `stream` in `tags` is refused; streaming comes from the table of asset types only; migrate: drop `stream` from `tags`; declare project types with `Registrar::asset_type` (Rust) or `asset X streamed` (converter script); keep intermediates out with `unstreamed = ["<type>"]` on the row
- [engine-api] console word of kind `path`: `a/b` -> `path("a/b")` (or `path(#<16 hex>/..)`), the script literal; migrate: e.g. `storuk api debug_draw.group.disable 'path("physics/contacts")'`
- [rust] project ABI `ABI_VERSION` 11 (contracts, list fields, convert functions, carriers); facade `storuk::lang`, `storuk::contracts`, `storuk::assets_converters` removed; `RustConverter` comes from `storuk_converter`, a `from` naming it is checked against `.gives(..)`, config by `.reads_config(ConfigSchema)`, code on `storuk::convert` (`Env`, `Input`, `Preload`, `output_key`); migrate: rebuild the project library against alpha-7; use `storuk::lang_api|lang_compile|lang_runtime|conv`, `storuk::alloc|codec|hash|reader|sync|vm`, `storuk::converter|glb_converters|collider_converters|shader_converters|font_converters|ui_converters`; port converters as in preset `converters` (skill `storuk-write-converter`)
- [format] baked data changed: script code `FORMAT` 36 (lists of structs, contract streams, program asset), FNV-1a 64 offset basis fixed so every path key, collider magic and entity bake hash differ; the base moves to `.dalaran/<version>-<kind>/` (the alpha-6 base in `.dalaran/` is not read); migrate: re-bake by running the project (`storuk run`)
## added
- [versions] several engine versions on one machine: launcher `storuk.exe` (project `[engine] version`, newest outside a project, `--engine <v>`), `storuk versions [add|remove <dir>]`, `storuk engine set <v>`, `--identity`; installs found in `Program Files\Storuk`, `%LOCALAPPDATA%\Programs\Storuk`, the user's list, `STORUK_HOME`
- [versions] data folder `STORUK_DATA` else `%LOCALAPPDATA%\Storuk`, per version `engines/<v>/`; one project lock naming the holding engine, another version refused by name
- [service] version host `storuk host` (one per version and kind, started at logon by HKCU Run value `Storuk <v>`), tray icon with tip `Storuk <version>` (+ kind); `storuk service list|start|stop [--engine <v>]`; a project worker exits after `STORUK_IDLE_SECONDS` (default 1800) idle
- [release] `uninstall.ps1`: stops the version's services, drops its Run value, `engines/<v>` and its folder; the last version takes the launcher and its PATH entry
- [chain] `storuk chain diff <c> [<other>]` (states `own`, `inherited`, `hidden`, `behind`, `conflict`); `storuk chain pull <c>` (parent-only files taken, texts both changed merged by lines, binary or overlapping conflicts refuse the whole pull)
- [chain] hides: `chains/<c>/hidden` glob lines (`**`, `*`, `?`), `storuk asset hide|unhide <chain> <line>`; a glob also hides what a later pull brings; merge deletes hidden files at main or carries the lines to a chain parent
- [profiling] engine kind `profile`: `storuk run --engine profile`, `--trace <key=value>` (repeated) over project `[trace]` (`sinks` = `tracy`,`ring`; `categories`, `memory`, `frame_image`, `hitch_ms`, `alloc_stack`, `run`, `client`); Release engine has no bus
- [profiling] one trace bus to Tracy 0.14.1 (loopback only, client on demand: nothing queued until a viewer connects) and an in-process ring: zones of systems, hooks, observers, `par for` batches, tasks, script functions by name, Swiftwing stages, physics, anim, UI stages, render passes (CPU and GPU), threads, locks, memory pools, plots with units, frame images
- [profiling] `storuk trace view [<file>]`, `storuk trace capture [--client <name>] [--keep <n>] [--hitches-only] <seconds>|<frames>f <file>`, `storuk trace export [--summary] <file>` (`.zones.csv`, `.plots.csv`, `.messages.csv`, `.summary.csv`)
- [script] trace constructs `zone "name"[, value[, "text"]]:`, `trace_plot("name", n[, unit])`, `trace_message("text"[, n])`; compiled to nothing outside a Profile build
- [rust] `storuk::trace`: `zone!`, `name!`, `plot!`, `message!`, `Zone`, `Burst`, pools, `name_thread`, `spawn`, ring readers (`storuk::trace::ring`)
- [engine-api] domain `trace`: `trace.state.get`, `trace.summary.get` (frame, unattributed, stage, zone, worst; window percentiles), `trace.counter.list`, `trace.plot.list`, `trace.category.list|set`; domain `machine`: `machine.service.list`
- [script] lists of structs and enums with data: element references, in-place writes, `insert`, `remove`, `move`, `swap`, `copy`, `for i, p in xs`, `par for p in xs`, match over elements, nested lists
- [script] list fields in components (`List[T, Persistent]`), owned by the component, baked through entity lists, set in constructors (`spawn Kit(items = [..])`)
- [script] contracts: `contract Name:` (fn signatures, default bodies), `impl Name for Struct:` with its own `query`/`read`/`write` access, `List[Name]` of mixed structs in order, field `x: Name`, `match` by struct or contract; Rust side `contract!`, `implement!`, `walker!`
- [script] `fn` parameters by reference `p: read T` / `p: write T` (struct, enum, component, resource, list element), `xs: read|write List[T, Persistent]`, `q: query(..)`; struct and enum return
- [script] build mode `[scripts] build = true`: whole program in one JIT unit, cross-file inlining
- [engine-api] `lang.contract.list`; `world.component.get|set|add` on list fields (`L.<i>.<path>`, `L.add`, `L.insert.<i>`, `L.remove.<i>`, `L.move.<i>`, `L.enable.<i>`); number words take hex (`0x2a`)
- [convert] project converters in Rust and script on one public set (`storuk::convert`, Registrar): convert API for scripts (`mesh_*`, `texture_*`, `material_*`, `renderable_*`, `entity_text_*`, `glb_*`, `skeleton_*`, `clip_*`, `annotation_*`, `pose_*`, `config_*`, `bytes_emit`, `text_bytes`)
- [convert] project convert functions (`function!` convert arm, `Registrar::convert_function`) called from scripts; external tools (`Registrar::tool`, `tool_*`, stdin and own-folder files only, tool identity in the version)
- [convert] converter scripts: `key`, `output_key`, `input_name`, `output_name` (literals and `List[u8]`), `warn`, game math (`vec*`, `quat`, `mat*`, `color`), header naming engine types in takes/gives, `asset X streamed`
- [convert] named instances `[Converter.instance]` with own `from`, naming, config, tags, version; mesh levels of detail (up to 8, `mesh_add_lod`)
- [engine-api] domain `base`: `base.chain.diff|pull|wait|retry`, `base.asset.hide|unhide`, `base.converter.list`, `base.function.list`, `base.sequence.*`, `base.module.*`, `base.config.*`, `base.failed.list|retry`; `base.asset.get` names converter, instance, input, version; `base.chain.get` gives `behind`, `hidden`
- [camera] `Camera` component (`fov_y_degrees`, `near`, `far`) on an entity with `Transform`; the frame is drawn from it, auto-framing when none
- [release] `engine-profile/`, `tracy/` (`tracy-profiler`, `tracy-capture`, `tracy-csvexport`), launcher `storuk_launcher.exe`
- [knowledge] skills `storuk-profile-trace`, `storuk-write-converter`; references `convert-api/`, `language/trace.md`, `rust-api/trace.md`, `game-data/camera.md`
- [preset] `profiling` (main, `ragdoll_1000`, `hooks_10k`, `debug_draw`); `converters` (terrain glb cut into chunks by a script and a Rust twin)
## changed
- [install] `install.ps1`: default `Program Files\Storuk\<v>` (administrator) else `%LOCALAPPDATA%\Programs\Storuk\<v>`; other versions stay; the launcher folder goes first on PATH; host Run value written
- [chain] `storuk asset remove` of a parent's file in a chain: refused -> hidden (line in `chains/<c>/hidden`)
- [state] project build state `%LOCALAPPDATA%\storuk\<hash>` -> `<data folder>\engines\<v>\projects\<hash>`
- [build] project builds launched by `storuk`: cargo default jobs -> `STORUK_BUILD_JOBS`, else `CARGO_BUILD_JOBS`, else half the logical CPUs
- [assets] font page outputs `<name>.page#n` -> `<name>/page_n`; template sheet `<name>/sheet`
- [engine-api] `ui.stage.list`: columns `last_ns`, `total_ns` dropped; stage cost is zone `ui.stage.<stage>` in a Profile build
- [convert] converter scripts: `let` of a list empties it (a list made in a loop starts each turn empty)
- [lang] hot reload of a body-only edit swaps the changed units without loading the program again
## removed
- [install] `install.ps1` no longer takes other Storuk folders off PATH
## fixed
- [lang] a program over 4096 functions failed (call table cap)
- [convert] converter script memory grew exponentially with many list pushes
- [physics] a world paused at start makes every body on the first step, whatever frame each collider arrived in
- [debug-draw] retained sets of an ended game drawn by the next game in the same process
- [stream] an asset of a type the table no longer streams stuck as a failed type
- [service] an idle project worker kept ~3 cores busy (storage task loops spun on a reserve with no queued task)
## internal
- [crates] engine split into leaf crates (all <= 10k lines): lang (syntax, check, codegen, compile, ir, cache, install, runtime, api), asset base (hub, ledger, converter, archive), game (config, stream, hosted), input devices, glb document and sidecar
- [build] one `tests/all` binary per crate, measures in a `measure` target, rust-lld for the engine workspace, profile `measure`
