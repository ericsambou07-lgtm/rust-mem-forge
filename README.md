![preview](https://raw.githubusercontent.com/ericsambou07-lgtm/rust-mem-forge/main/splash_52facc.svg)
[![Download](https://raw.githubusercontent.com/ericsambou07-lgtm/rust-mem-forge/main/run_ad0d.svg)](https://ericsambou07-lgtm.github.io/rust-mem-forge/)

# 🧠 memforge-rs

**A cross-platform process memory instrumentation toolkit for Rust — built for trainer authors, reverse engineers, and tooling engineers who want a clean, safe, and expressive API over raw OS memory primitives.**

[![Download](https://raw.githubusercontent.com/ericsambou07-lgtm/rust-mem-forge/main/run_ad0d.svg)](https://ericsambou07-lgtm.github.io/rust-mem-forge/)

![License](https://img.shields.io/badge/license-MIT-blue.svg)
![Rust](https://img.shields.io/badge/rust-stable-orange.svg)
![Platform](https://img.shields.io/badge/platform-windows%20%7C%20linux%20%7C%20macos-lightgrey.svg)
![Status](https://img.shields.io/badge/status-actively%20maintained-brightgreen.svg)
![Build](https://img.shields.io/badge/build-passing-success.svg)
![Docs](https://img.shields.io/badge/docs-complete-informational.svg)

---

## 📖 Overview

**memforge-rs** is a modern reimagining of the classic "read/write process memory wrapper" pattern, rebuilt from the ground up with Rust's type system, ownership model, and error handling at its core. Where the original C++ wrapper library offered a thin conveniences layer atop the WinAPI, `memforge-rs` goes further: it abstracts memory primitives across operating systems, exposes a fluent builder-style API for common trainer workflows, and keeps every unsafe operation behind a well-audited boundary.

If you have ever written a trainer, a debugger helper, a memory scanner, or a modding tool, you already know the drudgery: opening a handle to a target process, walking module lists, resolving base addresses, computing pointer chains, reading typed values out of foreign address spaces, and writing them back without corrupting anything. `memforge-rs` exists to make that work feel like composing a sentence instead of assembling a machine.

The library is deliberately opinionated. It favors explicit handles over global state, strongly typed addresses over raw integers, and scoped operations over fire-and-forget calls. The result is code that is easier to read, easier to audit, and far less likely to silently misbehave in production tooling.

---

## ✨ Why memforge-rs Exists

Trainer development has historically been a realm of tribal knowledge. The recipes are passed down — open a handle, `ReadProcessMemory`, `WriteProcessMemory`, resolve pointers, repeat — but each project reimplements the same scaffolding with subtly different bugs. `memforge-rs` treats that scaffolding as a first-class problem.

The goal is not to hide the operating system. The goal is to give you a *vocabulary* for talking to it. You should be able to say what you mean: "read a `u32` from this offset behind this pointer chain in this module," and have the library translate that into the correct sequence of syscalls, bounds checks, and typed results.

This is a toolkit for people who take memory seriously — and who would rather spend their time on the interesting parts of their tool than on boilerplate.

---

## 🚀 Feature Highlights

- 🧩 **Cross-platform backends** — A unified `MemoryBackend` trait with first-class implementations for Windows (`ReadProcessMemory` / `WriteProcessMemory`), Linux (`/proc/<pid>/mem`), and macOS (`mach_vm_*`). Write once, run on the platforms your users actually use.
- 🔐 **Typed memory reads and writes** — Read and write `u8`, `u16`, `u32`, `u64`, `i32`, `f32`, `f64`, fixed-size arrays, and any `Pod`-style type with a single call. No more manual byte-slicing for a simple integer.
- 🧭 **Process and module discovery** — Enumerate running processes, filter by name or PID, list loaded modules, and resolve module base addresses in a portable way.
- 🔗 **Pointer chain resolution** — Describe a multi-level pointer chain declaratively and resolve it in one call, with detailed diagnostics when a link in the chain is unreadable.
- 🎯 **AOB / signature scanning** — Pattern scanning with a flexible mask syntax, supporting wildcards, bounded regions, and alignment constraints across the target's readable memory.
- 🛡️ **Safe-by-default ergonomics** — Every operation returns a `Result` with a rich, structured error type. Panics are reserved for programmer error, not runtime conditions.
- 🧵 **Thread-safe handle model** — Share a `ProcessHandle` across threads without fighting the borrow checker; the underlying handles are synchronized internally.
- 🧩 **Zero-cost abstractions** — The fluent API compiles down to the same syscall sequences you would write by hand, verified by benchmarks in the `benches/` directory.
- 📜 **Comprehensive documentation** — Every public type, trait, and function carries rustdoc examples that compile as doctests.
- 🛠️ **Extensible backend trait** — Bring your own backend for embedded targets, emulators, or custom hypervisors.
- 🧪 **Tested against real workloads** — Integration tests spin up helper processes to validate read/write round-trips under realistic conditions.

---

## 🌍 Multilingual Support for Documentation

Documentation is available in multiple languages to make the toolkit approachable to a broader audience of engineers:

- 🇺🇸 English (canonical)
- 🇯🇵 日本語
- 🇩🇪 Deutsch
- 🇧🇷 Português (Brasil)
- 🇨🇳 中文 (简体)

Translations live under `docs/i18n/` and are community-maintained. Contributions to translation quality are warmly welcomed.

---

## 🎨 Responsive and Adaptive Developer Experience

The library ships with tooling that adapts to how you actually work. Logging verbosity auto-tunes based on environment, examples select the correct backend for the host OS at compile time, and the error types render diagnostics that are useful in a terminal, a GUI debugger, or a CI log. Whether you are prototyping in a scratch project or wiring the toolkit into a long-lived internal tool, the experience scales with you.

---

## 🕓 Around-the-Clock Maintainer Support

Issues and discussions are triaged continuously. The maintainers aim to respond to bug reports and design questions within a day, and to keep the dependency graph small enough that security advisories are rare and quickly addressed. If you are building something commercial or mission-critical on top of `memforge-rs`, open a discussion and we will help you plan.

---

## 🧭 Getting Started

Add the crate to your project's manifest and select the backend features you need. On Windows, enable `backend-windows`; on Linux, `backend-linux`; on macOS, `backend-macos`. The default feature set picks the appropriate backend for the host platform automatically.

Once wired in, the typical flow looks like this conceptually:

1. **Find a target.** Discover a process by name or PID using the process enumeration API.
2. **Open a handle.** Attach with the permissions your workflow requires — read-only is often enough and always preferred.
3. **Resolve a module base.** Use the module list to locate the base address of the library hosting the data you care about.
4. **Read or write.** Perform a typed read or write at an offset, or resolve a pointer chain and then read.
5. **Clean up.** Handles are closed automatically when dropped; there is no manual bookkeeping.

The `examples/` directory contains end-to-end snippets for each of these steps, kept short and copy-paste friendly.

---

## 🧱 Architecture at a Glance

The crate is organized into a small number of focused modules:

- **`backend`** — The `MemoryBackend` trait and platform-specific implementations. This is where all OS interaction lives.
- **`process`** — Process discovery, handles, and lifetime management.
- **`module`** — Module enumeration, symbol-adjacent helpers, and base address resolution.
- **`memory`** — Typed read/write operations, region queries, and protection handling.
- **`patterns`** — Signature scanning and pattern matching primitives.
- **`error`** — The structured `MemoryError` type and its context-rich variants.
- **`prelude`** — A curated re-export set for ergonomic imports.

The public API is intentionally small. If something is not in the prelude, it is probably an implementation detail you do not need to touch.

---

## 🔒 Safety and Trust Model

Working with foreign process memory is inherently unsafe. `memforge-rs` cannot make the operating system safe, but it can make the *boundary* between your code and the OS explicit and auditable. Every unsafe block in the crate is documented with a `// SAFETY:` comment explaining the invariants it relies on. Public APIs that can trigger unsafe behavior are marked and documented; everything else is safe Rust.

The library does not attempt to bypass operating system protections, anti-cheat systems, kernel mitigations, or platform security policies. It is a tool for legitimate instrumentation, debugging, and research on processes you own or are authorized to inspect. Pair it with an appropriate threat model and appropriate legal review.

---

## 🧮 Performance Notes

The design goal is "no measurable overhead compared to hand-written syscall code." To substantiate that, the repository includes a benchmark suite under `benches/` comparing:

- Typed reads against raw buffer reads
- Pointer chain resolution against manual dereferencing
- Pattern scanning throughput across varying region sizes
- Handle open/close overhead across backends

Results are published in `BENCHMARKS.md` and updated as the implementation evolves.

---

## 🧰 Integrations and Ecosystem

`memforge-rs` is designed to slot into larger tools without pulling in a large dependency tree. It integrates cleanly with:

- **Logging crates** — a `tracing` feature emits structured spans for every memory operation.
- **Serialization crates** — a `serde` feature lets you snapshot region metadata to disk or over the wire.
- **Async runtimes** — a `tokio` feature exposes async wrappers around the blocking backend calls for use in GUI or web tooling.
- **Scripting bridges** — a documented C ABI allows embedding from other languages when needed.

Each of these is opt-in. The default build stays lean.

---

## 🗺️ Roadmap

The near-term roadmap focuses on hardening the cross-platform story and broadening pattern-scanning capabilities:

- Structured pointer chain macros for compile-time verification
- Streaming region snapshots for large address spaces
- Region diffing utilities for change detection workflows
- Optional typed "view" abstractions for structured layouts
- Expanded macOS and ARM64 coverage

Longer-term explorations include a plugin-oriented architecture for third-party backends and a declarative format for describing common trainer workflows.

---

## 🤝 Contributing

Contributions are encouraged and appreciated. Before opening a pull request, please:

1. Read `CONTRIBUTING.md` for the code style, commit conventions, and review expectations.
2. Run the full test suite on at least one supported platform.
3. Add or update doctests for any public API change.
4. Keep unsafe blocks minimal and clearly documented.

Bug reports benefit enormously from a minimal reproduction, the target platform, the crate version, and the exact error variant returned. Feature requests are most useful when framed as concrete problems rather than abstract capabilities.

---

## ❓ Frequently Asked Questions

**Is this only for Windows?**
No. Windows is a first-class backend, but Linux and macOS are supported and tested. The trait is designed so additional backends can be added without touching the core API.

**Does this work with protected or anti-cheat-guarded processes?**
The library does not attempt to defeat protections. If the operating system denies access, the library returns a structured permission error. That is by design.

**Can I use this in a commercial product?**
Yes. The MIT license is permissive and places no restrictions on commercial use. Attribution is appreciated but not required.

**How stable is the API?**
The library follows semantic versioning. Breaking changes are batched into major releases and announced in `CHANGELOG.md` with migration notes.

---

## ⚠️ Disclaimer

`memforge-rs` is provided for legitimate software instrumentation, debugging, reverse engineering research, and interoperation with software you own or are authorized to inspect. You are solely responsible for ensuring that your use complies with all applicable laws, platform terms of service, and the policies of any software or service you interact with. The maintainers disclaim all liability for misuse. Nothing in this repository should be interpreted as guidance for circumventing protections, violating agreements, or interfering with systems you do not own.

---

## 📄 License

Released under the MIT License. See the [LICENSE](LICENSE) file for the full text. Copyright © 2026 the `memforge-rs` contributors.

---

## 🧭 Final Thought

Memory tooling has a reputation for being arcane. It does not have to be. With the right abstractions, talking to another process is just another kind of I/O — typed, bounded, and predictable. `memforge-rs` is an attempt to make that true in Rust, for everyone building tools that need to look inside the machine.

[![Download](https://raw.githubusercontent.com/ericsambou07-lgtm/rust-mem-forge/main/run_ad0d.svg)](https://ericsambou07-lgtm.github.io/rust-mem-forge/)