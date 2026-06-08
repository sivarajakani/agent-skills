# BoltFFI Packaging Outputs

Expected outputs after successful packaging.

## Commands

- `boltffi pack apple`
- `boltffi pack android`
- `boltffi pack java`
- `boltffi pack csharp`
- `boltffi pack wasm`
- `boltffi pack all --release`

## Typical Dist Layout

- `dist/apple/` : XCFramework and Swift package artifacts
- `dist/android/` : JNI libraries by ABI and Kotlin bindings
- `dist/java/` : JNI library and Java bindings
- `dist/csharp/` : NuGet package with runtime native assets
- `dist/wasm/` : wasm module, JS glue, TypeScript definitions, package metadata

## Verification By Target

Apple:
- XCFramework exists under `dist/apple/`
- Swift module/binding files are present

Android:
- Native libraries exist for configured ABIs
- Kotlin bindings are present

Java:
- Java sources/classes are generated
- JNI native library is present

CSharp:
- `.nupkg` exists under `dist/csharp/`
- runtime native assets included in package layout

WASM/TypeScript:
- `.wasm`, JS glue, and `.d.ts` are present
- package metadata is generated

## Fast Validation Flow

1. Run one pack command for target under development.
2. Confirm output folder and core artifacts.
3. Run one language-level smoke call.
4. Move to next target or run `pack all --release`.
