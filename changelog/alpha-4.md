---
version: alpha-4
prev: alpha-3
date: 2026-11-17
commit: 1ed6e4325085c2bedbded78b6da88ea4c6d0f565
range: 5dde45e..1ed6e43
breaking: yes
---
## breaking
- [engine] `[engine] version` of a project must equal `alpha-4`; an alpha-3 project is refused; migrate: set `version = "alpha-4"` under `[engine]` in `dalaran.toml`
- [script] `bone_read` takes a 4th argument, the space (`SPACE_LOCAL`/`SPACE_MODEL`/`SPACE_WORLD`); migrate: add the space to every `bone_read` call
- [rust] `BoneOverride` is `{ mode, space, channels, weight, value }` (was a local transform); `AnimDriven` and `ControlError::Driven` removed; core `set_override`/`clear_override` -> `set_write`/`clear_write`/`clear_writes`; migrate: write `BoneOverride` with `mode` VALUE and `space` LOCAL for the old behaviour, drop `AnimDriven` (a skinned model gets its rig when something drives a bone)
## added
- [animation] bone access at two levels: `BoneOverride` on a bone entity (VALUE or FOLLOW, space LOCAL/MODEL/WORLD, channels ROT/POS/SCALE, weight) and a bone stream on the rig (`PoseLists`: lists `bones`, `pos`, `rot`, `scale`, `weight`, `space`, `channels`); one write pass per rig, no call per bone
- [animation] FOLLOW: the bone takes the pose of its entity (e.g. a physics body); bone sync leaves that entity's `Transform` alone; a bone entity that left its hierarchy without FOLLOW is skipped, said once
- [animation] rig without a clip: a skinned model gets a rig when an override or a stream drives it, also with no window (skeleton through streaming)
- [animation] pose tasks for providers: `Writes` task, kernel tasks (`TwoBoneIk`, custom kernels registered at setup), the result of the previous provider as input; a task chain too deep is an error
- [animation] explicit lane per rig: `anim_set_lane` with `LANE_FIXED`/`LANE_FRAME`; FOLLOW reads the drawn pose in the frame lane, the simulation pose in the fixed lane; the fixed lane runs before physics
- [animation] bone owner = the model instance the bone was made in (several models under one parent, lifted bodies)
- [script] the built-in `Transform` of any entity, read and write (`px py pz qx qy qz qw sx sy sz`; simulation pose with `Interpolated`)
- [script] data `BoneOverride`, `PoseLists`; constants `SPACE_*`, `BONE_VALUE`/`BONE_FOLLOW`, `BONE_ROT`/`BONE_POS`/`BONE_SCALE`, `LANE_*`
- [script] list fields in data components declared by the engine or a Rust project
- [run] words `--anim-scratch`, `--anim-tasks`, `--anim-writes`, `--anim-kernel-params`, `--anim-write-slots`, `--anim-write-transforms`
- [preset] `knight_ragdoll_override`: ragdoll knight from data only (bodies, shapes, joints on bone sections + `BoneOverride` FOLLOW)
- [preset] `knight_ragdoll_stream`: ragdoll bodies as their own entities, a script writes the knight's bone stream in one batch per frame
## removed
- [animation] the fixed bone-override table per slot (8 overrides); write pools are the `--anim-write-*` words
## fixed
- [prefab] `model_spawn` places models in the order of the calls (streaming order changed entity order and broke physics determinism)
- [animation] overrides apply on slots that are not due this frame (step 2/4/8) and on slots with no task result
## internal
- [lang] script cache format 7
