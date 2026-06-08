---
name: boltffi
description: 'Generate and troubleshoot cross-platform Rust bindings with BoltFFI. Use when setting up boltffi in a crate, designing #[data]/#[export]/#[error] APIs, running boltffi pack for Swift/Kotlin/Java/CSharp/WASM, validating generated artifacts, or fixing type mapping and packaging failures.'
argument-hint: 'Describe your target and goal, e.g. "add async export and pack wasm" or "set up all targets from existing crate"'
user-invocable: true
disable-model-invocation: false
---

# BoltFFI Workflow Skill

## What This Skill Produces

A repeatable workflow for taking a Rust library from plain source code to validated multi-language bindings using BoltFFI.

## When To Use

- Add BoltFFI to a new or existing Rust library crate.
- Expose Rust APIs with `#[data]`, `#[export]`, and `#[error]`.
- Generate packages for Apple, Android, Java, CSharp, or WASM/TypeScript.
- Verify generated outputs and run platform smoke checks.
- Debug common BoltFFI configuration or type mapping issues.

## Workflow Map

1. Verify prerequisites and crate shape.
2. Define export-safe API surface.
3. Initialize and tune BoltFFI config.
4. Pack one or more targets.
5. Validate artifacts and binding behavior.
6. Troubleshoot and iterate.

## Step 1: Verify Prerequisites And Crate Shape

1. Confirm toolchain and install the CLI:
   - `rustc --version`
   - `cargo install boltffi_cli`
2. Ensure the crate is a library and supports native outputs:
   - `cargo new --lib mylib` (if creating fresh)
   - Add crate type and dependency using [Cargo snippet](./assets/Cargo.toml.snippet).
3. For target-specific builds, ensure host prerequisites:
   - Apple: Xcode command line tools.
   - Android: Android Studio + NDK.
   - Java: JDK 8+ and C toolchain.
   - CSharp: .NET SDK.
   - WASM: Node.js and `wasm-pack`.

Completion check:
- `cargo check` succeeds.
- `boltffi --help` runs.

## Step 2: Define Export-Safe API Surface

1. Start from [Rust template](./assets/lib.rs.template).
2. Mark value-like data with `#[data]`.
3. Mark public functions or impl blocks with `#[export]`.
4. Use `#[error]` for error structs returned in `Result<T, E>`.
5. Avoid unsupported or risky boundary types in public exports.

Decision points:
- New crate: begin with a minimal exported function and one data type.
- Existing crate: wrap existing internals with a clean FFI-facing API layer.

Reference:
- [Type mapping guide](./references/type-mapping.md)

Completion check:
- Exported API compiles with `cargo check`.
- No generics or trait objects in exported signatures.

## Step 3: Initialize And Tune BoltFFI Config

1. Generate baseline config:
   - `boltffi init`
2. Compare/update with strict [boltffi config template](./assets/boltffi.toml.template).
3. Choose target strategy:
   - Single-target validation first for faster iteration.
   - Full multi-target packaging for release readiness.
4. Optional: use overlays for CI/release variants.

Completion check:
- `boltffi.toml` exists and matches intended targets.

## Step 4: Pack Targets

Fast single-target loop:
- `boltffi pack wasm`
- `boltffi pack java`
- `boltffi pack apple`
- `boltffi pack android`
- `boltffi pack csharp`

Release-grade multi-target run:
- `boltffi pack all --release`

Decision points:
- Debugging API shape: pack one target first.
- Pre-release verification: pack all targets in release mode.

Reference:
- [Packaging outputs by target](./references/packaging-outputs.md)

Completion check:
- Expected `dist/<target>/` directories are generated.

## Step 5: Validate Outputs And Behavior

1. Inspect generated binding artifacts for selected targets.
2. Run a minimal smoke call in each language binding.
3. Validate naming/style and async/error behavior.
4. Confirm package metadata and runtime assets are present.

Completion check:
- Artifacts present for each target.
- One successful function call from each integrated language.

## Step 6: Troubleshoot And Iterate

Use [Troubleshooting guide](./references/troubleshooting.md) for common failures:
- missing crate type
- missing annotations
- type mapping mismatch
- packaging/runtime loading failures
- async runtime assumptions

Iteration loop:
1. identify failing stage
2. apply smallest API/config fix
3. repack target
4. rerun smoke check

## Quality Criteria (Done Definition)

- `cargo check` passes.
- `boltffi pack <target>` passes for intended targets.
- Generated artifact set matches target expectations.
- Exported API uses stable, mapping-safe types.
- Async and errors map as expected in at least one target-language smoke test.

## Example Prompts To Use This Skill

- "Set up BoltFFI for this existing Rust crate and generate WASM plus Java outputs."
- "Refactor these exports to be BoltFFI-safe, then pack Android and CSharp."
- "Troubleshoot why my `Result` error is not surfacing as a native exception in Kotlin."
- "Prepare this crate for `boltffi pack all --release` and run output validation checks."
