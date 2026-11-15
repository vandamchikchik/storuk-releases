---
version: alpha-1
prev: pre-alpha
date: 2026-11-15
commit: d97c010cbf25fdcd1a03f7633ce40fb026bb050c
range: 1ea297a..d97c010
breaking: yes
---
## breaking
- [engine] `[engine] version` of a project must equal `alpha-1`; a pre-alpha project is refused; migrate: set `version = "alpha-1"` under `[engine]` in `dalaran.toml` (or `storuk new` a new project), then `storuk run` builds it again
- [release] unzip over the pre-alpha folder leaves old files; migrate: unzip into a new folder
## added
- [render] Vulkan renderer in games: forward pass, depth, mesh + material + texture + shader, one light + ambient, HDR frame blitted to an sRGB window; meshes, textures, materials, shaders reach it through streaming only
- [run] `storuk run` opens a window per client (`--window`); `--headless` starts clients with no window and no renderer
- [client] window words after `--`: `--window-size WxH`, `--window-background`, `--present fifo|mailbox|immediate`, `--validation on|off`, `--snapshot <png>`
- [client] windowed and headless clients print the `world prefabs=.. placed=.. ..` census; windowed clients also print `render meshes=.. textures=.. materials=.. pipelines=.. ..`
- [gpu] any Vulkan 1.3 GPU with the needed features, discrete preferred; choice words after `--`: `--gpu <name part>`, `--gpu-vendor <id>`, `--gpu-discrete-only`
- [gpu] no suitable GPU or no Vulkan: client prints `storuk_game: <why>`, run ends with `ended with code 10 (Platform)`
- [camera] a game with no camera sees what it placed: default camera frames the drawn instances
- [template] `storuk new` makes a game with a car (`in/car.glb`, Kenney Car Kit, CC0-1.0, `in/car.entity_text`, `scripts/main.dal`): `storuk run` shows it at once
- [models] own `x.glb` in `in/` + `x.entity_text` (`include renderable x`) + `instantiate(model("x"), x, y, z)` in a script = drawn; glb edit while running swaps mesh, material, texture live
- [models] glb meshes without `TEXCOORD_0` accepted and drawn; tangents by MikkTSpace
- [entity] game entities described in `.entity_text` files next to the model; glb hierarchy kept
- [sequences] explicit converter sequences per chain (`converters/sequences/*.toml`); template ships `glb.toml`
- [shaders] surface shader = project file `in/shaders/surface.slang` (pass `forward`); edit while running swaps the shader; compiled by shipped `slang/slangc.exe`, found from the release folder (no Vulkan SDK needed)
- [textures] BCn textures encoded by the asset service; empty material slots take default maps `in/engine/default_white.glb`, `in/engine/default_normal.glb`
- [requirement] Vulkan 1.3 driver; Vulkan only
- [requirement] validation layer not shipped: `--validation on` needs the Vulkan SDK; default off
- [release] `slang/` (shader compiler, Apache-2.0 WITH LLVM-exception), template asset licences in `licenses/THIRD-PARTY.md`, `changelog/alpha-1.md`
## changed
- [run] exit code of `storuk run` -> code of the first failed client
- [process] child processes (service, clients, compilers) -> start without console windows
