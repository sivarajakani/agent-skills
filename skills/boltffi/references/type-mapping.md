# BoltFFI Type Mapping Notes

Use this file when designing API signatures intended for cross-language bindings.

## Design Rules

- Prefer small, explicit structs with primitive fields for boundary data.
- Use `&str` for string inputs and `String` for string outputs.
- Use `Result<T, E>` with `#[error]` on `E` for exception-like behavior in target languages.
- Use owned return values instead of borrowed outputs.
- Keep exported signatures concrete; avoid generics and trait objects.

## Common Mapping Examples

| Rust | Typical Target Shape |
|---|---|
| `i32`, `f64`, `bool` | native primitive scalar |
| `String`, `&str` | native string type |
| `Option<T>` | nullable/optional |
| `Vec<T>`, `&[T]` | array/list |
| `Vec<u8>` | byte array / data buffer |
| `Result<T, E>` | value or thrown exception/rejected promise |

## Async Mapping Expectations

- Rust async exports map to native async constructs in each target.
- Runtime execution is still your Rust crate responsibility.

Checklist:
- include one async smoke call per target using async bindings.

## Performance Notes

- Primitive-heavy APIs are generally fastest across boundaries.
- Strings and nested collections carry higher crossing overhead.
- Prefer batching data work in Rust and returning summarized results where possible.
