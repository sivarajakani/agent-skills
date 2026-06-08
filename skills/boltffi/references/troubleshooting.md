# BoltFFI Troubleshooting

## 1) No Bindings Generated For Expected Items

Symptoms:
- Generated package misses specific structs/functions.

Checks:
- Verify `#[data]`, `#[export]`, and `#[error]` are present where needed.
- Ensure items are public and part of the library API.

## 2) Native Library Not Produced

Symptoms:
- Pack command runs but target cannot load native binary.

Checks:
- Confirm crate type includes both `cdylib` and `staticlib`.
- Confirm target toolchain prerequisites are installed.

## 3) Type Mapping Errors

Symptoms:
- Codegen fails or generated API has unexpected shape.

Checks:
- Replace generic or trait-object exports with concrete types.
- Return owned values instead of borrowed references.
- Simplify nested collections in public signatures.

## 4) Errors Not Showing As Native Exceptions

Symptoms:
- Target side receives generic failure or unclear message.

Checks:
- Ensure error type is annotated with `#[error]`.
- Ensure function returns `Result<T, E>` in exported API.

## 5) Async API Misbehavior

Symptoms:
- Async methods appear but fail at runtime.

Checks:
- Confirm Rust async runtime setup in crate internals.
- Run a minimal async smoke call after packaging.

## 6) Platform Packaging Mismatch

Symptoms:
- Missing files under `dist/<target>/`.

Checks:
- Revisit `boltffi.toml` target config.
- Try single-target pack first, then multi-target run.
- Use release build for final packaging pass.

## Diagnostic Loop

1. Capture exact failing command and error text.
2. Isolate to one target and one exported function.
3. Apply smallest signature/config change.
4. Re-run pack and smoke test.
5. Scale fix to full target set.
