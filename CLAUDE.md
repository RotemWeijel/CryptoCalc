# CryptoCalc — Guidelines for Claude Code

CryptoCalc is an educational, client-side-only web application that lets students of introductory cryptography explore cryptographic primitives and encryption schemes by running security experiments (for example, playing the distinguisher in a PRG or PRF experiment).

Before starting any task, read `docs/PROJECT_BRIEF.md` (product requirements and roadmap).

## Language

- Write everything in English: identifiers, code comments, docs, README, commit messages, PR titles and descriptions, issue text. This applies even when the maintainer writes to you in Hebrew.
- User-facing UI text is English by default and must live in dedicated string files (see "Configuration").

## Tech stack (defaults, to be confirmed in the architecture task)

- React + TypeScript, built with Vite, tested with Vitest. Node.js tooling only.
- Client-side only: no backend, no server code, no database, no network calls at runtime.
- Cryptographic arithmetic uses `BigInt` where integers can exceed 2^53.
- Formal definitions are stored as LaTeX strings in data files and rendered with KaTeX (default choice, to be confirmed).

## Architecture principles

- Follow SOLID. Code must be closed for modification and open for extension.
- Keep the core logic (primitives, schemes, experiments) fully separate from the UI. Core code must not import React.
- Every primitive, scheme, example and experiment is a module that implements a shared interface and is registered in a registry. Adding or removing one must not require editing existing logic. UI pages render from module metadata.
- Examples are generic. Each example (a PRG, a PRF, and so on) is a declarative definition: its LaTeX definition, its input and output lengths, its evaluation function, the component primitives it is built from, and whether it is secure. Pages and experiments take such a definition as input and never branch on a specific example. Adding a new example means adding data only, and every option (refute, prove, reveal the answer) works for it immediately.
- Experiments follow the same rules for every example: fresh randomness in every round, and bit-string lengths validated at every boundary (a fixed-length bit-string type is recommended) so that a string of the wrong length can never be passed to an oracle.
- Proof mode is automatic. The reduction framework is generic: the site simulates the assumed distinguisher `D'` by exhaustive search over all `2^n` seeds, and the user plays the distinguishing experiment against the underlying primitive with `D'` as an extra oracle. There is no reduction builder.
- Exhaustive simulation requires small security parameters. Enforce the maximum `n` from the limits file, cache results per example and `n`, and run heavy computation off the UI thread (for example in a Web Worker).
- Depend on abstractions (interfaces), not concrete implementations. Inject dependencies (for example the randomness source) so that experiments are testable and deterministic under test.
- Prefer small, single-purpose modules and functions.

## Configuration

Nothing that may change during development is hard-coded in components. Keep it in dedicated, easy-to-edit files:

- UI strings, labels and explanations: string files, grouped by feature.
- Theme tokens (colors, fonts, spacing): a theme file.
- Limits and defaults, for example the maximum security parameter (kept small because of exhaustive simulation, value decided in the architecture task), the maximum number of rounds (10) and the default security parameter (10): a limits file.

The exact folder layout is decided in the architecture task. Keep it consistent afterwards.

## Terminology

Use standard cryptographic terminology in code, UI and docs:

- pseudorandom generator (PRG), pseudorandom function (PRF), cryptographic hash function
- security parameter `n`
- encryption scheme (never "consensus")
- distinguisher, oracle access, reduction, proof by contradiction, negligible advantage, challenge bit
- Kerckhoffs's principle: algorithms are public, so users get oracle access to every component in an example's definition
- never "manufacturer" or "producer" (use "generator"), and never "safety factor" (use "security parameter")
- eavesdropper (EAV) security, CPA security, CCA security

## Scope

- The parser that lets users enter their own primitive or scheme definition is deferred to a later stage. Do not implement it now. Where the UI shows that option, it is visible but does nothing.
- This is a teaching tool. The cryptographic code is for demonstration only and must never be presented as production-grade.

## Working agreement

- Before starting each task, ask clarifying questions if anything is ambiguous. For any non-trivial task, propose a plan first and wait for approval.
- One task per branch and per pull request. Keep pull requests small and reviewable.
- Never push directly to `main`. Branch names: `feat/<topic>`, `fix/<topic>`, `docs/<topic>`, `chore/<topic>`.
- Commit messages follow Conventional Commits (`feat:`, `fix:`, `docs:`, `test:`, `refactor:`, `chore:`).
- Before opening a pull request, run lint, tests and the build (once they exist) and make sure they pass.
- Write unit tests for core logic. Update the README and docs together with the code.
- Do not commit secrets, `node_modules` or build output.

## Commands

To be filled in after the project scaffold is merged (install, dev server, build, lint, test).
