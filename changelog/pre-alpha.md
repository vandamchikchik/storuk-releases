---
version: pre-alpha
prev: none
date: 2026-11-13
commit: 1ea297af4ec50deb883b28a1788f8cf3b8eedbdd
range: 30d8e15..1ea297a
breaking: no
---
## added
- [release] archive `storuk-pre-alpha.zip`, Windows x64: programs `storuk.exe` (project commands + project service, host of project libraries), `storuk_game.exe` (game client); engine `storuk.dll` + `storuk.dll.lib` + Rust std DLL; `deps/` (rmeta + proc-macro DLLs of engine dependencies); `toolchain/` (rustc + cargo 1.96.0, target libs, license files); `template/`; `template-builds/`; `licenses/THIRD-PARTY.md`; `changelog/pre-alpha.md`
- [requirement] MSVC `link.exe` (Visual Studio or Build Tools) on the machine; not shipped; a failed link says so
- [new] `storuk new <name>`: project folder from `template/` (`dalaran.toml`, `Cargo.toml` dylib crate, `src/lib.rs`, `in/`, `chains/`, `scripts/`, `converters/`, `.gitignore`), template library installed: first `storuk run` compiles nothing; `git init` when git is found
- [engine] `[engine] version` in `dalaran.toml`; every project command refuses a project of another engine version, refusal names the line to write
- [run] `storuk run [--chain <c>]... [--instances <n>] [--no-build] [-- <game words>]`: starts or finds the project service, builds project/chain library when sources or engine changed, starts N game clients per chain; closing a client leaves the base running
- [run] live: asset edit in `in/` swaps into running clients; `scripts/` edit reloads game scripts or restarts the client (exit 17); `src/` edit rebuilds the library and restarts clients (exit 16); Ctrl-C / console close = orderly close
- [service] project service = own process; base follows the project folder (`in/`, `configs/`, `converters/`, `scripts/`, `chains/<c>/`); chain layers over parent; commands are file operations
- [chain] `storuk chain new <c> [from <parent>]`; merge of a chain into its parent by files
- [chain] `storuk chain new-rust <c> [from <parent>]`: chain with its own crate (copy of the parent crate set, `rust = true` in `chain.toml`); a plain chain holding crate files is refused; merging a Rust chain replaces the parent crate set (resumable after a stop)
- [scripts] game scripts in `scripts/`: `model("car.glb")` asset key literal, `instantiate(model, x, y, z)` places a prefab in the world
- [converters] converter settings on disk (`converters/sequences/*.toml`), converter scripts per chain, Rust converters per chain from the project library
- [build] project library = Rust dylib against the shipped engine; incremental through the cargo profile; libraries kept per source state (same sources build nothing)
