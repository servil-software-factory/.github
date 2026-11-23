# Servil Software Factory

**We aspire to create software that serves its users reliably and loyally.**

That is easy to say, and hard to do. We do it. Here is how.

---

## Our Values and Principles

Software called *servil* exists for one purpose: to serve the person running it — never an advertiser, never an unverified dependency chain, never its authors' convenience at the user's expense. Five principles govern every Official project we ship:

1. **Loyalty to the user.** Minimize what the user has to trust. Keep dependency trees tiny, prefer small owned code over sprawling third-party stacks, verify every build input by cryptographic hash rather than HTTPS alone, and ship reproducible artifacts anyone can rebuild and check byte-for-byte.
2. **Reliability by construction, not by hope.** Make invalid states unrepresentable in the type system, keep critical invariant checks active in release builds, fail closed rather than returning a wrong answer, prove critical kernels mechanically, and test the tests themselves with mutation testing and planted faults.
3. **Radical honesty.** Say plainly what our checks cover *and* what remains outside them. Disclose openly that our code is written primarily by AI agents under human direction and release approval, and never imply an independent human security audit where none has been done. Never hide a slowdown. When a serious bug gets past us, publish a frank after-action report explaining what broke and what structural check stops it from happening again.
4. **Evidence over claims.** Gate every commit mechanically on its exact SHA, never by memory. Measure performance on quiet, dedicated native hardware, with fresh link layouts and every cell judged at once; a virtual machine's timing is best effort and never decides a landing or a release. Calibrate our AI security auditors by measuring their actual recall on real bugs of ours before trusting them on a release.
5. **Clear boundaries.** Support only the platforms we actually build and test natively on our own hardware, and refuse unsupported targets clearly. Tell users plainly which software we are fully committed to supporting (**Official**, starting at `1.0`), which repositories exist to verify it (**Infrastructure**), and which are unfinished experiments (**Experimental**).

---

## Project Status

### Official (Supported) Projects

**None yet.**

We reserve Official status — our full commitment to long-term bug and security support — for projects that have reached **`1.0`** and satisfy every standard in this document. We are publishing this standard now so users can inspect our practices, follow our work toward `1.0`, and hold us to it.

### Pre-1.0 Candidate (Path to Official)

Projects actively running our Factory practices, gates, security reporting, and reproducible releases while APIs and supply-chain hardening finish ahead of `1.0`:

| Project | Repository | Ships | Current release | Notes |
|---|---|---|---|---|
| **BLAKE3-Servil** | [`servil-software-factory/BLAKE3`](https://github.com/servil-software-factory/BLAKE3) | `blake3-servil-core`, `blake3-servil`, `blake3-servil-version`, `b3sums` (`b3sum` CLI), C API (`capi/`) | `0.10.0` (pre-1.0) | First candidate for Official status at `1.0`. It is where the practices below come from, and it runs most of them today: `SECURITY.md`, `QUALITY.md`, the landing gates, mutation testing, the proofs, reproducible releases, public after-action reports, hash-pinned CI inputs, minimized dependencies, `cargo-deny`/`cargo-vet` gating (#479), and cross-family landing review on shipped code (a trial). Currently rolling out ahead of `1.0` (#499): rewriting stateful types so types carry their rules (private fields, checked constructors), crate-wide lint tables (no narrowing `as`, a checked reason on every `unsafe` block and `debug_assert!`), an enforced stack budget, guard-page test runs, full target×feature `-D warnings` coverage, lifecycle work counts, and removing the native C build (#487). Pre-1.0 APIs may still change between minor versions. |

### Infrastructure

Public repositories that exist to test, benchmark, and verify our projects. They are public so skeptical users can audit our methodology and raw results — not supported end-user products (best-effort fixes, no security support commitment):

- **[`servil-software-factory/bench-hashes`](https://github.com/servil-software-factory/bench-hashes)** — Cross-machine hash benchmarking suite and published performance maps used to verify BLAKE3-Servil's speed claims and run A/B regression checks.
- **[`servil-software-factory/mac-runner`](https://github.com/servil-software-factory/mac-runner)** — Public job queue and verdict archive for our dedicated Apple Silicon test and timing machines.

### Experimental Projects (Not Supported)

Research prototypes, libraries, and tools under exploration. **We do not commit to bug fixes, security response, API stability, or continued maintenance for experimental projects.** They may change incompatibly or be abandoned without notice. Do not rely on them to protect anything of value until they are promoted to Official.

| Project | Repository | What it is | Notes |
|---|---|---|---|
| **Sigil** | [`johnservil/sigil`](https://github.com/johnservil/sigil) | Compact multi-script glyph encoding (`sigilcode`) for hashes and bytes | `0.2.0` (hex-prefixed) is released on trial to fix 0.1's Unicode 15.1 Cham line-break flaw. *(Treat `b3sums --sigilcode` as experimental until the 0.2 format is finalized.)* |
| **Ode** | [`samshackleton-tech/ode`](https://github.com/samshackleton-tech/ode) | Research prototype for proof-carrying native code (binary + short contract + machine-checkable proof) | Early research prototype. |
| **Tare** | [`devonjonte/tare`](https://github.com/devonjonte/tare) | A/B benchmarking tool using fresh link layouts, alternating runs, and simultaneous multi-cell judging | Extracted from our benchmarking methodology. |
| **cargo-mutamancer** | [`johnservil/cargo-mutamancer`](https://github.com/johnservil/cargo-mutamancer) | Fast commit-scoped Rust mutation testing experiment | Research/tooling prototype. |
| **content-derived-chunk-slicing** | [`johnservil/content-derived-chunk-slicing`](https://github.com/johnservil/content-derived-chunk-slicing) | Content-defined chunking store + slice manifest experiment | Research prototype. |
| **blake3-servil-c-b3sum** | [`samshackleton-tech/blake3-servil-c-b3sum`](https://github.com/samshackleton-tech/blake3-servil-c-b3sum) | Example C consumer using the BLAKE3-Servil C API | Reference example. |

*(Internal workflow forks such as `johnservil/pi` and writings/talks are not Servil Software Factory products.)*

---

## How We Build Official Projects

These policies, practices, and tools were earned through real bugs, regressions, and release mistakes in `BLAKE3-Servil` (see its [`QUALITY.md`](https://github.com/servil-software-factory/BLAKE3/blob/main/QUALITY.md) and public after-action reports). Any project promoted to Official (`1.0+`) must satisfy all universal practices below, plus the conditional practices matching its risk surface.

### Universal Practices (Every Official Project)

1. **Fail closed, enforce invariants in types, and write contracts for stateful types.**
   - Internal callers go through the same checked APIs as external callers (for example, `BLAKE3-Servil` removed its `hazmat` module, made `Platform` `#[non_exhaustive]` so safe code cannot forge CPU capability tokens, and enforced buffer alignment and padding through `Block` and `Batch`).
   - Types holding invariants keep their fields private and document their invariants and every operation, including trait implementations (`Clone`, `Drop`, `Eq`, `Debug`).
   - Cheap invariant checks stay active in release builds; `debug_assert!` requires a site comment explaining why it cannot run in release builds. Every `unsafe` block states why it is sound, and crate lint tables deny narrowing `as` casts and silent error drops. Crash-only binaries (`panic = "abort"`) stop immediately rather than continuing after a broken invariant or failed I/O safety check.

2. **Maintenance and design guidelines: finish changes, prune vestiges immediately, collapse copies.**
   - **Cut changes small enough that one landing finishes them.** Never leave two versions of one idea alive in the tree; "later" is not a state. Code left past its sell-by date — a new version started while callers stay on the old one — is where stale copies and bugs gather.
   - **Extirpate vestigial code immediately** in the commit that makes it vestigial (dead paths, unused options, superseded APIs), without spending time measuring them.
   - **Collapse duplicate implementations into one with two users**, even at a small cost in speed (weighed on real measurements where speed is promised). Keep a replaced component as a test oracle while its replacement lands, and watch non-rising counts of shipped lines, entry points, and `cfg`s so complexity does not creep back.

3. **Verify against an independent oracle, and test the tests.**
   - Check outputs against an independent implementation or specification (e.g., official BLAKE3 test vectors and the upstream `blake3` crate, `bao-tree` for outboards). Where work is countable by arithmetic (such as compressions or kernel calls), check work counts in lifecycle tests so excess work fails immediately on any machine without a clock.
   - Run **diff-scoped mutation testing** as a landing gate (`tools/mutants/check.py` narrowed by line reach via `tools/tests/reach.py`, typically ~1 minute per landing) plus periodic whole-crate mutation runs. Every surviving mutant needs either a killing test or a written reason in a checked-in list; a timeout is a bug to turn into a fast failure.
   - **Plant deliberate faults** to verify safety checks and verification tools reject them (removing CLI safety checks one by one, planting wrong rotations or bounds in kernels, planting concurrency bugs in model checks).

4. **Mechanical landing gates on the exact commit SHA (`tools/land.py`).**
   - Nothing lands on `next` by memory or by "CI was green earlier." A mechanical landing script requires every gate status — CI across every supported target, compiler, and feature combination with warnings denied, native platform suites, mutation testing, regression checks (where compiled code changed), and landing review (where shipped code changed) — on the **exact commit** being fast-forwarded.
   - Support only platforms we actually build and test natively; fail clearly at build time on unsupported platforms and point to an upstream alternative where one exists.
   - Every open issue and pull request has a single assignee whose move is next.

5. **Cross-family AI landing review + recall-calibrated multi-family release audits.**
   - **At landing (Rule 2, trialing on `BLAKE3-Servil`):** Every change to *shipped code* (libraries, CLI, C API, build scripts, manifests) receives one round of independent review by an AI model of a **different family** than the author, running tests and adversarial probes in a sandbox on our build host (`review-landing`). Findings hold the landing only for a wrong result, crash/panic, hang, or slowdown; non-blocking findings land and become tracked issues. Tools, CI workflows, docs, and mutation lists need no landing review.
   - **Before every stable release:** At least two independent AI agents from different leading-edge model families audit the release candidate and final version diff. A model family counts toward the release bar only after we have **measured its recall** on a real defect tree of ours with known bugs. Audit reports and finding dispositions are published with the release evidence.
   - **Candid limit:** AI review catches real bugs, *and* models share blind spots. **AI review is not an independent human security audit.** Each project's `QUALITY.md` states plainly whether an independent human audit has occurred.

6. **Minimal dependencies and a hash-verified supply chain.**
   - Prefer small, readable, owned code over large dependency trees, proc-macros, and unnecessary external build toolchains (e.g., a purpose-built thread pool instead of Rayon, replacing heavy CLI/helper crates with small special-purpose code, and eliminating native C build dependencies — tracked in `BLAKE3` #479).
   - Pin every GitHub Action to a full 40-character commit SHA, every container image to a `@sha256:` digest, and every downloaded bootstrap tool to a versioned URL verified by SHA-256 (rolled out across `BLAKE3` in #479).
   - Commit lockfiles and enforce `--locked` builds; gate dependencies with `cargo-deny` and `cargo-vet` across all crates and features; dependency changes happen only in explicit, reviewable commits, never silently at release time.

7. **Clean, reproducible, independently re-verified releases.**
   - Stamp releases mechanically (`tools/gen-ver.py`) as `vX.Y.Z+<40-char-code-commit-sha>`, binding the tag, manifests, lockfiles, headers, and `--version` to the exact source commit. Lock release tags and GitHub releases against modification or deletion.
   - Verify every release step mechanically (`tools/release-check.py`). A human maintainer (Zooko) explicitly approves every release tag, crate publication, yank, and advisory.
   - Build binaries and crate archives byte-for-byte reproducibly inside a pinned toolchain/container (`tools/b3sum-release-container.sh`, `tools/crates-release.sh`), require two independent builders to match byte-for-byte before release, and re-verify what crates.io and GitHub actually serve after publication (`tools/release_artifacts.py TAG --published --rebuilt`).
   - Where we do not use GPG or SLSA signatures, say so plainly and rely on immutable commit-bound tags + independent reproducible rebuilds instead.

8. **Honest documentation, clear security response, and public after-action reports.**
   - Every Official project ships a plain-English `QUALITY.md` stating every protection layer beside what remains outside it, with commands to reproduce the checks yourself.
   - Every Official project ships a `SECURITY.md` with private vulnerability reporting via GitHub Security Advisories, fixed releases, public advisories, and crate yanks when continued use of a release is unsafe.
   - Serious defects get a public after-action report explaining what failed, how it got past our checks, and what structural check was added so that class of bug cannot pass again.

### Conditional Practices (Required Whenever a Project Has That Risk Surface)

| If a project contains… | It must also apply… | Proven pattern from `BLAKE3-Servil` |
|---|---|---|
| **Hand-written assembly or crypto inner kernels** | Machine-checked formal proofs (outputs, memory bounds, ABI, secret-independent branches/addresses); prover fault controls; startup known-answer self-test. | `tools/verify/` (Z3 + Sail/C2SP symbolic proofs, input-hashed long proofs, `test_mutants.py`), `initialize()` self-test. |
| **Custom concurrency / lock-free synchronization** | Exhaustive schedule model checking with planted concurrency faults; weak-memory fence verification on Arm64 (`RCpc` loads). | Shuttle tests (`shuttle_finish`, `shuttle_park`, `shuttle_pipeline` + 10 planted bugs), `tools/check-waker-order.py`. |
| **`unsafe` memory handling or FFI** | Unreadable guard-page tests flush against input/output buffers; script-checked `SAFETY:` comments on every `unsafe` block. | `src/test_support.rs::guard`, `SAFETY` comment checker. |
| **Performance as a core user promise** | All-cells A/B regression gate on quiet, dedicated **native** hardware (alternating runs, fresh link layouts, FDR-controlled judging); a virtual machine's timing is best effort and never decides a landing or a release; own every real slowdown (fix it, or state it plainly in `CHANGELOG.md` with machine and size). | `bench-hashes`, `clocks` / `tare`, dedicated timing Macs & Linux hosts under `bench-turn`, `tools/size.py`. |

---

## What We Promise Users — and What We Do Not

### For Official (`1.0+`) projects, we promise:
1. **Active bug & security response for the latest release:** private vulnerability reporting via GitHub Security Advisories (`SECURITY.md`), fixes for confirmed vulnerabilities, public advisories, crate yanks where unsafe, and frank public after-action reports for serious released defects.
2. **Verifiable releases:** every official artifact corresponds byte-for-byte to an immutable `vX.Y.Z+<commit>` tag that anyone can rebuild and verify.
3. **Honest boundaries:** `QUALITY.md` states what our checks cover *and* what they do not, including supported platforms, whether any human security audit has occurred, and any known slowdowns in `CHANGELOG.md`.
4. **Semver discipline:** no breaking API changes within a `1.x` major version; every user-visible change documented in `CHANGELOG.md`.
5. **No silent abandonment:** if an Official project is ever retired or demoted, we will mark it plainly in its README, release notes, and package registry.

### We explicitly do NOT promise:
- **No bug-free guarantee and no implied independent human security audit** (unless a project's `QUALITY.md` explicitly names one).
- **No backports to older releases** (support covers the latest release only).
- **No support commitment for Experimental projects, Infrastructure repos, or pre-1.0 candidates** (pre-1.0 candidates like `BLAKE3-Servil 0.10.0` still receive active security fixes and advisories as we work toward `1.0`, while APIs may still change).
- **No support for unlisted platforms** (we refuse unsupported builds rather than shipping untested targets).
- **No contractual clock-time SLA:** we respond promptly on a best-effort basis.

---

## Promotion Checklist: Experimental / Pre-1.0 → Official (`1.0`)

A project is promoted to Official only by explicit decision of the organization owner (Zooko) at `1.0`, once all of the following are in place:

1. Stable `1.0` API surface (no experimental dependencies or unstable flags exposed without clear separation).
2. `README.md` states scope, supported platforms, and AI authorship under human direction, and links to this standard.
3. `CONTRIBUTING.md` leads with these values and practices and documents the project's gates.
4. `QUALITY.md` documents every verification layer, what remains outside it, and reproduction commands.
5. `SECURITY.md` enables private vulnerability reporting and states scope and supported versions.
6. Mechanical landing gate (`land.py` or equivalent) enforces CI, native tests, mutation testing / fault controls, and Rule 2 cross-family landing review on shipped code.
7. Dependencies are minimized and locked; CI actions, containers, and downloads are pinned by commit SHA / cryptographic hash.
8. Reproducible release tooling + `--published` verification work end-to-end, with immutable release tags enabled.
9. All conditional practices matching the project's risk surface are in place.
10. Pre-release multi-family AI security audits (using recall-calibrated models) are complete with credible findings resolved and dispositions archived.
