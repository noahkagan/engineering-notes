# ADR-0016: Cross-Platform Structure for Rust Crates

- **Status:** Accepted
- **Date:** 2026-05-03
- **Supersedes:** _none_

## Context

Rust crates that target multiple platforms (e.g., Linux, macOS, Windows, BSDs, WASI) face two structural questions:

1. **How is platform-specific code organized at the module level?** Where do `#[cfg(...)]` attributes live, and what does the public API look like across platforms?
2. **How are platform crates (e.g., libc, windows-sys, core-foundation) contained?** Such crates churn on independent release cadences. If their types appear in a crate's signatures, that crate's semver becomes coupled to theirs.

## Decision

Adopt a two-layer approach: **module-per-platform** for code organization, and **internal facades** for platform crate isolation.

### Layer 1: Module-per-platform

`#[cfg(...)]` is permitted in exactly two places: on `mod` declarations that select a platform module, and on `[target.'cfg(...)'.dependencies]` entries in `Cargo.toml`. Nowhere else.

Specifically:

- **Public items must not carry `#[cfg]`.** No `#[cfg]` on `pub` functions, types, traits, fields, or impls. A `cfg`-gated public item leaks the platform set into every downstream caller and breaks rustdoc parity across platforms.
- **No `#[cfg]` inside function bodies.** A platform branch inside a function body is a signal that the function belongs in a platform module, not that it needs a `cfg` block.
- **No `if cfg!(...)` for platform dispatch.** Runtime dispatch on a compile-time constant compiles all branches and obscures intent. Use module selection instead.
- **`cfg_if!` is not an escape hatch.** It is permitted only inside platform modules or private helpers, and only when the divergent code is genuinely small (a handful of lines) and self-contained. Anything larger is a per-platform module.

The public API is platform-agnostic; callers never write `cfg`, and rustdoc on any supported platform shows the same surface.

The public API is platform-agnostic; callers never write `cfg`, and rustdoc on any supported platform shows the same surface.

```rust
// src/lib.rs — the only place #[cfg] appears in the source tree
#[cfg(target_os = "linux")]
#[path = "linux.rs"]
mod imp;

#[cfg(target_os = "macos")]
#[path = "macos.rs"]
mod imp;

#[cfg(target_os = "windows")]
#[path = "windows.rs"]
mod imp;

pub fn config_dir() -> Option<PathBuf> {
    imp::config_dir()
}
```

Each platform module implements an identical surface; `lib.rs` is a thin facade with no `cfg`-gated public items.

Cargo dependencies are scoped to the targets that need them:

```toml
[target.'cfg(unix)'.dependencies]
libc = "0.2"

[target.'cfg(target_os = "windows")'.dependencies]
windows-sys = { version = "0.52", features = ["Win32_Foundation"] }

[target.'cfg(target_os = "macos")'.dependencies]
core-foundation = "0.9"
```

Unsupported platforms get an `unsupported.rs` stub returning `Err`/`None`, or a `compile_error!` to fail loudly.

### Layer 2: Internal facades around platform crates

A platform module's exposed items are expressed in std types and the crate's own types — never in terms of the platform crate. Returning `PathBuf` keeps the underlying implementation swappable; returning `windows_sys::Win32::Foundation::HANDLE` makes the platform crate part of the public API.

**Newtype wrappers around foreign handles.** Wrap platform resources, do not expose them:

```rust
// windows.rs — internal to the crate
pub(crate) struct RegistryKey(HANDLE);

impl RegistryKey {
    pub(crate) fn open(path: &str) -> io::Result<Self> { /* ... */ }
    pub(crate) fn read_string(&self, name: &str) -> io::Result<String> { /* ... */ }
}

impl Drop for RegistryKey {
    fn drop(&mut self) {
        unsafe { RegCloseKey(self.0); }
    }
}
```

`pub(crate)` keeps the wrapper out of the public API. The underlying platform crate can be swapped or rewritten without touching anything outside this file.

**Trait-based facade for parity.** When the same conceptual resource exists on multiple platforms, define a trait in a shared module and re-export the platform impl as a single type:

```rust
// src/watcher/mod.rs
pub(crate) trait Watcher {
    fn watch(&mut self, path: &Path) -> io::Result<()>;
    fn poll(&mut self) -> io::Result<Vec<Event>>;
}

#[cfg(target_os = "linux")]
pub(crate) use self::inotify::InotifyWatcher as PlatformWatcher;
// ...
```

Callers use `PlatformWatcher` as if it were one type. Prefer the trait when the compiler should enforce parity between platform implementations; a type alias with matching inherent methods suffices when no trait-object use is anticipated.

**Error translation at the boundary.** Platform-specific errors are translated inside the platform module. `io::Result` is the default for OS-ish operations; richer cases convert into a crate-level `Error` enum at the boundary. Platform error types never escape the file.

**No `pub use` of platform crate types.** Re-exporting a foreign type at the crate root couples the crate's semver to the platform crate's. Wrap instead.

## Consequences

### Positive

- The public API is decoupled from platform crate semver. Upgrading a platform crate across breaking changes — or swapping one wrapper for another — does not force a major version bump.
- Adding a new platform is a localized change: one new module file, one new `[target.'cfg(...)'.dependencies]` block, no edits to existing platform code.
- Review has a clear rule: a foreign type outside its platform module is a defect.
- One file per platform is easier to navigate than `#[cfg]` interleaved through every module.

### Negative

- More boilerplate up front. A trivial wrapper around a `HANDLE` is ceremony when the underlying call is one line.
- Trait-based parity requires keeping multiple implementations in sync. Trait changes touch every platform module in the same PR.
- Some platform APIs lack natural cross-platform analogues. Platform-only methods on the facade are sometimes unavoidable; mark them clearly and use sparingly.
- `compile_error!` on unsupported targets can surprise users on tier-3 platforms. The choice between hard failure and a soft `unsupported.rs` fallback is per-case.

### Neutral

- The decision does not constrain *which* platform crate to use (e.g., `windows-sys` vs. `windows`, `nix` vs. raw `libc`). The facade makes that choice reversible.
- CI matrix coverage across the supported platforms, plus `cargo check` on claimed tier-2 targets, is a follow-on operational requirement — not part of this decision.

## Alternatives considered

**Inline `#[cfg]` everywhere.** Lower up-front cost, but `cfg` propagates through call sites and tests, and every refactor must consider all platforms simultaneously. Rejected because the cost compounds with codebase size.

**Single platform crate (e.g., `rustix` for everything).** Attractive — `rustix` already does much of this work. Rejected as the *primary* strategy because coverage is incomplete on some targets and the dependency on one upstream's roadmap is total. Acceptable *inside* a platform module.

**Re-export platform crate types at the crate root.** Maximally convenient for callers, maximally coupling for the crate. Rejected; the semver risk is precisely what this ADR exists to prevent.

**Trait objects (`Box<dyn Watcher>`) as the primary cross-platform abstraction.** Adds runtime indirection where static dispatch suffices. The type alias re-export pattern provides the same ergonomics with zero overhead. Rejected as default; available where dynamic dispatch is genuinely needed.

## Review heuristics

Reject on sight:

- `#[cfg(...)]` on any `pub` item.
- `#[cfg(...)]` inside a function body.
- `if cfg!(...)` used for platform dispatch.
- `pub use` of a type from `libc`, `windows_sys`, `core_foundation`, `nix`, `windows`, or any other platform crate.
- A foreign platform-crate type appearing in a signature outside its platform module — including `pub(crate)` items intended for the rest of the crate.
- `cfg_if!` blocks longer than a handful of lines, or any `cfg_if!` in a public module.

Each of these has a mechanical fix: move the code into a platform module, wrap the foreign type, or split the function.

### When leaks signal crate-ification

A single `cfg` leak is a local defect. A *cluster* of leaks around the same concept — filesystem watching, registry access, process spawning, IPC — is a different signal: a subsystem has outgrown the facade pattern and wants to be its own crate.

Indicators that crate-ification is the right response, not another wrapper:

- Multiple platform modules each contain non-trivial logic for the same concept, and the shared trait or type alias has accumulated platform-specific methods.
- The subsystem has its own error cases, its own lifecycle, and its own tests that are awkward to colocate with the parent crate.
- Adding a new platform requires touching the same handful of files in the same way each time.

The fix is to extract the subsystem into its own crate (internal workspace member first; published only if it stands on its own). The new crate applies this ADR independently: its own platform modules, its own facade, its own `pub(crate)` wrappers. The parent crate then depends on it and sees a single platform-agnostic surface.

This is a judgment call, not a mechanical rule — but the trigger is mechanical: when the same subsystem keeps appearing in review heuristics violations, stop fixing the violations and extract the crate.

## References

- `dirs` / `directories` — module-per-OS layout for filesystem paths
- `notify` — abstracts inotify/kqueue/`ReadDirectoryChangesW` behind one API
- `socket2` — wraps platform socket APIs in a unified type
- `rustix` — modern syscall wrapper with strong trait-based abstractions
