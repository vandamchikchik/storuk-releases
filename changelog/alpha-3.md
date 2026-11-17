---
version: alpha-3
prev: alpha-2
date: 2026-11-17
commit: 5dde45efa0a5e5008be861b0c86427e328b757f3
range: 4c12d3c..5dde45e
breaking: yes
---
## breaking
- [engine] `[engine] version` of a project must equal `alpha-3`; an alpha-2 project is refused; migrate: set `version = "alpha-3"` under `[engine]` in `dalaran.toml` (or `storuk new` a new project)
- [script] engine commands renamed to `<object>_<verb>`: `instantiate` -> `model_spawn`, `pause`/`resume`/`set_speed` -> `time_pause`/`time_resume`/`time_set_speed`, `overlap_sphere` -> `sphere_overlap`; migrate: rename the calls in `scripts/*.dal`
- [script] physics events are messages, not entities: `ContactBegin`, `ContactEnd`, `SensorBegin`, `SensorEnd`, `ContactHit`, `JointOverload`; migrate: read them with `read hits: ContactHit` (or `read hits: previous ContactHit`), not by `query`
- [rust] every project function and command needs a description: `function!(r, f, (..) -> .., "about")`, `function_guarded!` the same, `Registrar::command(name, params, apply, about)`; a blank one is refused (`RegisterError::NoAbout`); `ABI_VERSION` 2; migrate: add the description argument, rebuild the project crate
## added
- [animation] skeletal animation from glb: skeletons, skinned meshes (up to 12 influences per vertex), clips resampled at 30 Hz with 16-bit tracks, root motion; through `GlbExtract` and streaming only
- [animation] sidecar `configs/GlbExtract/<input>.toml`: clip cuts of a take, additive clips (base skeleton/frame/clip), root motion bone, near-only bones (far LOD), secondary skeletons, a foreign skeleton by name, masks with per-bone weights, annotations (slots + up to 32 tags per bone), limits
- [animation] clip key `<glb>/<animation>`; with several skeletons a clip per skeleton, `#<n>` suffix from the second one
- [animation] several skinned meshes on one skin; rigid meshes parented to joints follow their bone
- [animation] runtime: layers base/masked/additive, crossfade, speed, loop/once/posed, update step 1/2/4/8 per character, 2-level bone LOD, explicit bone override, live clip swap at the same fraction, skeleton swap
- [animation] root motion with 4 transition modes (blend, additive, ignore source, ignore target); root-motion rigs advance in the fixed step (same path at any frame rate); entity marked external takes the delta itself
- [animation] GPU skinning: pose pool with previous pose (motion vectors), palette compute pass, skinned forward and depth passes, culling by the bone box
- [entity] bones as optional sections of a renderable: only a touched bone becomes a child entity (`section Rig/<bone>:`); `drop` removes a section of an included renderable
- [script] animation commands: `anim_play`, `anim_set_speed`, `anim_set_step`, `anim_set_lod`, `anim_set_root_motion`, `anim_layer_play`, `anim_layer_set_weight`, `anim_layer_set_masked`, `anim_layer_set_additive`, `anim_layer_set_root`, `anim_layer_stop`, `anim_root_read`, `bone_read`; data `AnimRig`, `BoneRead`, `RootRead`; constants `ANIM_LOOP`/`ANIM_ONCE`/`ANIM_POSED`, `ROOT_*`, `LOD_*`
- [script] literals `clip("<glb>/<animation>")` (clips stream with the program; an unknown clip is a notice) and `path("a/b")` (bone or mask names, hashed at compile time)
- [script] named constants of the engine, listed in the reference with their description
- [run] words `--anim-max-bones`, `--anim-max-frames`, `--anim-max-mask-sets`, `--anim-max-slots`, `--anim-max-ops`, `--pose-matrices`, `--palette-matrices`, `--objects`, `--depth-pass 0|1`
- [preset] `animation`: KayKit Adventurers Knight (CC0, unchanged glb) with sword, shield, helmet, cape; `scripts/main.dal` plays Idle -> Walking_A -> Running_A -> Idle with 0.5 s crossfades
## changed
- [physics] `Transform` holds the simulation pose; render draws the interpolation blend between fixed steps
- [physics] a physics fault or a refused frame fails the frame: exit code 18
- [service] `get`/`show`/`delete` take the 32 digits of an unnamed asset (collider) as `list` prints them
## fixed
- [streaming] two waiting heads in different channels no longer hold each other (memory-full never came)
## internal
- [lang] script cache format 6
- [layers] physics schema in leaf `storuk_body_types`; collider writer behind feature `bake`
