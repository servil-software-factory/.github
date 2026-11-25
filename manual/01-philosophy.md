# Servil Software Factory Manual — Chapter 1: Philosophy, Strategy, Style

Have fun! Optimism and positivity make you, and the people you work with, more effective.

Simplicity and complexity are easy to overlook while solving the problem right in front of you. Step back, look with fresh eyes, and imagine how your diff reads to someone else. Simplicity pays compounding returns for your future self and for others — your emotional response to beauty is a good guide.

Read this chapter at the start of every session and after compaction. Whenever you notice one of the triggers below, pause and follow up in the full chapter before writing more code.

## Six triggers

1. **Adding a mechanism, mode, cache, or bridge?** → **Simplify or delete first.**
   Prefer the simplest design that meets the contract and performs well: one source of truth, fewest concepts, fewest modes and runtime branches. Before patching an inherited mechanism, ask whether deleting or replacing it is cleaner. Read your diff as an inventory — every type, flag, file, and rule must earn its place.

2. **Leaving a half-migration, duplicate, or "later" cleanup?** → **Finish in one landing; extirpate vestiges immediately.**
   Cut changes small enough that one landing finishes them, and delete superseded paths in the same commit without spending time measuring them. Collapse duplicates into one implementation with two users. Between tasks, take a dedicated pass just to look for what can be simplified or deleted.

3. **Writing "caller must…" in a comment, or reaching for a new crate?** → **Make invalid states unwriteable, and keep the trusted base tiny.**
   Put invariants into private fields, checked constructors, newtypes, explicit state machines, and RAII guards so the compiler enforces them for internal and external callers alike. Keep safety/correctness checks active in release builds (fail stop, no silent fallbacks), justify every `unsafe` and `debug_assert!`, prefer small owned code over third-party dependencies, and hash-pin every build/CI input. *(Full chapter: [`profile/README.md`](../profile/README.md#universal-practices-every-official-project), `BLAKE3` [`docs/quality-manual.md`](https://github.com/servil-software-factory/BLAKE3/blob/next/docs/quality-manual.md).)*

4. **Adding a check, hitting a flake/hang twice, or waiting serially?** → **Prove it can fail, fix the root, overlap waits.**
   No test, check, or control counts until you have seen it fail against a real or planted fault. Never repeat a hang-and-wait cycle twice — stop and fix the root cause. Overlap independent waits (reviews, builds, tests, timing) and spend calendar time where defect yield is highest. *(Full chapter: `BLAKE3` [`CONTRIBUTING.md`](https://github.com/servil-software-factory/BLAKE3/blob/next/CONTRIBUTING.md).)*

5. **Touching hot code, resource lifetimes, or timing?** → **Speed, efficiency, and halting are part of correctness.**
   A slowdown or background spin with no caller is a bug; count cache lines, page faults, and cross-core traffic, not raw struct size. Use crash-only lifetimes for process/thread singletons and caller workspaces, and free dynamically constructed library objects cleanly rather than inventing a custom allocator. Measure natively with `bench-hashes` (`clocks`, alternating fresh layouts, all cells judged together); VMs are best-effort timing. *(Full chapter: [`bench-hashes/METHODOLOGY.md`](https://github.com/servil-software-factory/bench-hashes/blob/next/METHODOLOGY.md).)*

6. **Scoping docs, naming rules, or supporting a platform?** → **Be plain, positive, and honest about boundaries.**
   Phrase rules positively, keep one audience per document, and support only platforms built and tested natively (refuse others clearly at build time). Follow every practice for its purpose — waive checks that cannot observe a change in one checkable sentence.

---

*Next chapters:* [Values, tiers, and Official standards (`profile/README.md`)](../profile/README.md) · [Procedures & gates (`BLAKE3/CONTRIBUTING.md`)](https://github.com/servil-software-factory/BLAKE3/blob/next/CONTRIBUTING.md) · [Quality engineering (`BLAKE3/docs/quality-manual.md`)](https://github.com/servil-software-factory/BLAKE3/blob/next/docs/quality-manual.md) · [Benchmarking methodology (`bench-hashes/METHODOLOGY.md`)](https://github.com/servil-software-factory/bench-hashes/blob/next/METHODOLOGY.md)
