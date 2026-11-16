---
version: alpha-2
prev: alpha-1
date: 2026-11-16
commit: 4c12d3ca37b334a4929422228355af0afde50f3a
range: d97c010..4c12d3c
breaking: yes
---
## breaking
- [engine] `[engine] version` of a project must equal `alpha-2`; an alpha-1 project is refused; migrate: set `version = "alpha-2"` under `[engine]` in `dalaran.toml` (or `storuk new` a new project), then `storuk run` builds it again
## added
- [physics] Box3D 3D physics (upstream commit `be8c7318`) in games: bodies, shapes, joints stepped in the engine fixed step on the engine thread pool; determinism across worker counts
- [physics] entity traits `PhysShape` (sphere, capsule, box, hull, mesh, compound; `density`, `friction`, `restitution`, `category`, `mask`), `PhysBody` (`body_type` 0 static / 1 kinematic / 2 dynamic, `gravity_scale`, `never_sleep`), `PhysJoint` (all Box3D joint kinds); omitted options = 0: write `gravity_scale=1 density=1`
- [physics] colliders baked from glb meshes (hull, mesh) and TOML compounds (nested compounds flattened); reach the game through streaming only; collider edit while running swaps the body shapes
- [physics] script commands: `body_create`, `body_spawn`, `body_destroy`, `body_set_type`, `body_teleport`, `body_set_velocity`, `body_set_angular_velocity`, `body_apply_impulse`, `body_apply_force`; `shape_sphere`, `shape_capsule`, `shape_box`; `joint_revolute`, `joint_distance`, `joint_weld`, `joint_spherical`, `joint_prismatic`, `joint_wheel`, `joint_destroy`; `ray_cast`, `sphere_cast`, `overlap_sphere` -> `CastHit` / `OverlapHit` entities
- [physics] contact, sensor, hit and joint events per frame; optional pose interpolation between fixed steps (`Interpolated`)
- [time] fixed step + frame clock: script groups ending in `Fixed` run per step; resource `Time` (`dt`, `elapsed`, `tick`, `alpha`, `real_dt`, `real_elapsed`, `frame`, `speed`, `paused`, `fixed`); commands `pause()`, `resume()`, `set_speed(x)` (0..=64), applied at the next frame
- [new] `storuk new <name> --preset <preset>`: project made from a shipped preset; unknown preset refused with the list; `storuk help new` lists presets and their chains
- [preset] `box3d`: physics scenes, one chain each: `main` (box stack), `large_pyramid`, `many_pyramids`, `joint_grid` (Box3D benchmark scenes, upstream positions), `pendulum`, `chain`, `ragdoll`, `chair_static`, `chair_dynamic`, `two_worlds`; every scene is one game entity `in/scene.entity_text` composed of object entities by `append entity`; run: `storuk run --chain <chain>`
- [client] window word `--snapshot-after <frames>`: snapshot taken that many frames after the first drawn one
- [release] `presets/` next to `storuk.exe`; preset asset licences in `licenses/THIRD-PARTY.md`; `changelog/alpha-2.md`
## changed
- [new] engine default maps `in/engine/` written by `storuk new` into each project -> not by the release
## internal
- [streaming] Swiftwing channels, per-channel priority scheduler, disk division between channels, memory rule (`WaitingMemory`, `MemoryFull`), window metrics; manifest format v4 with part table, part requests in resolver and GPU destinations
- [streaming] unload mark: `frame_done(frame, gpu_done)`; memory freed at `gpu_done >= mark`
- [base] shared assets by claim key (dedup by input)
- [build] `template/Cargo.lock` tracked; `storuk_release` refuses a shipped crate project without lock (`NoLock`); relative `out`, `--changelog`, `--target` resolved from the current folder
- [render] retained render graph: steady frame with no hashing and no allocations
- [physics] chain overhead over the Box3D step: one parallel pose + move-event pass per world, body -> entity table; 0.4-1.3 ms -> 0.05-0.15 ms per step
