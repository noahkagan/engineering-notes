---
name: rust-cross-platform-crates
description: Conventions for Rust crates that target multiple platforms — module-per-platform structure and internal facades around platform crates (libc, windows-sys, core-foundation, nix, etc.). TRIGGER when editing or reviewing a Rust crate that has `[target.'cfg(...)'.dependencies]`, `#[cfg(target_os = ...)]`, `#[cfg(unix)]`, or imports from a platform crate; also when designing a new crate that will compile on more than one OS. SKIP for single-platform code, application binaries with no portability surface, and Rust code with no platform-conditional logic.
---

# Rust cross-platform crates

Operationalizes [engineering-notes ADR-0016](https://github.com/noahkagan/engineering-notes/blob/main/docs/adr/0016-cross-platform-rust-crates/decision.md). Consult the ADR for the *why*; this skill carries the rules and the review heuristics.

## Two rules

### 1. Module-per-platform — `#[cfg]` lives in exactly two places

- `mod` declarations in `lib.rs` (or a parent module) that select a platform module.
- `[target.'cfg(...)'.dependencies]` entries in `Cargo.toml`.

Nowhere else. Each platform implements an identical surface; the parent module is a thin facade with no `cfg`-gated public items.

```rust
#[cfg(target_os = "linux")]   #[path = "linux.rs"]   mod imp;
#[cfg(target_os = "macos")]   #[path = "macos.rs"]   mod imp;
#[cfg(target_os = "windows")] #[path = "windows.rs"] mod imp;

pub fn config_dir() -> Option<PathBuf> { imp::config_dir() }
```

Unsupported platforms get an `unsupported.rs` stub returning `Err`/`None`, or a `compile_error!` to fail loudly. Choose per-case.

### 2. Internal facades — platform crate types never escape their module

A platform module exposes std types and the crate's own types. Foreign types (`HANDLE`, `libc::c_int`, `CFStringRef`, etc.) stay inside the file that imports them.

- **Newtype wrappers** around foreign handles, with `Drop` for cleanup, marked `pub(crate)`.
- **Trait or type alias** in a shared module to express parity across platforms; re-export the platform impl as a single `PlatformWatcher`-style type.
- **Error translation at the boundary** — `io::Result` is the default; richer cases convert into a crate-level `Error` enum inside the platform module. Platform error types never escape.
- **No `pub use`** of a type from `libc`, `windows_sys`, `windows`, `core_foundation`, `nix`, or any other platform crate.

## Reject on sight

When writing or reviewing, treat each of these as a defect with a mechanical fix (move to platform module, wrap the type, or split the function):

- `#[cfg(...)]` on any `pub` item.
- `#[cfg(...)]` inside a function body.
- `if cfg!(...)` used for platform dispatch.
- `pub use` of a platform-crate type.
- A foreign platform-crate type in a signature outside its platform module — including `pub(crate)` items shared with the rest of the crate.
- `cfg_if!` blocks longer than a handful of lines, or any `cfg_if!` in a public module.

## When the pattern bends, extract a crate

A *single* `cfg` leak is a local defect. A *cluster* of leaks around the same concept (filesystem watching, registry, process spawning, IPC) signals that the subsystem has outgrown the facade and wants its own crate. Indicators:

- Multiple platform modules each contain non-trivial logic for the same concept, and the shared trait has accumulated platform-specific methods.
- The subsystem has its own error cases, lifecycle, and tests that are awkward to colocate.
- Adding a new platform requires touching the same handful of files in the same way each time.

The extracted crate applies this skill independently. The parent then sees a single platform-agnostic surface.

## Out of scope

- *Which* platform crate to use (`windows-sys` vs. `windows`, `nix` vs. raw `libc`, `rustix` for everything). The facade makes that choice reversible — pick per platform module.
- CI matrix design and tier-2/tier-3 coverage strategy. Operational, not structural.
- Single-platform crates and application binaries. This skill does not apply.
