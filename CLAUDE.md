# CryptoCalc — Guidelines for Claude Code

CryptoCalc is an educational, client-side-only web application that lets students of introductory cryptography explore cryptographic primitives and encryption schemes by running security experiments (for example, playing the distinguisher in a PRG or PRF experiment).

Before starting any task, read `docs/PROJECT_BRIEF.md` (product requirements and roadmap) and `docs/ARCHITECTURE.md` (architecture, folder layout and the v1 versus later scope split).

## Language

- Write everything in English: identifiers, code comments, docs, README, commit messages, PR titles and descriptions, issue text. This applies even when the maintainer writes to you in Hebrew.
- User-facing UI text is English by default and must live in dedicated string files (see "Configuration").

## Tech stack (confirmed in `docs/ARCHITECTURE.md`, section 1)

- React + TypeScript (strict), built with Vite, tested with Vitest and React Testing Library. Node.js tooling only (Node 24 LTS, 22 supported).
- React Router with a hash router (deployed to GitHub Pages). Styling with CSS Modules and CSS custom properties generated from the theme file.
- Client-side only: no backend, no server code, no database, no network calls at runtime.
- Cryptographic arithmetic uses `BigInt` where integers can exceed 2^53.
- Formal definitions are stored as LaTeX strings in data files and rendered with KaTeX.

## Architecture principles

- Follow SOLID. Code must be closed for modification and open for extension.
- Keep the core logic (primitives, schemes, experiments) fully separate from the UI. Core code must not import React.
- Every primitive, scheme, example and experiment is a module that implements a shared interface and is registered in a registry. Adding or removing one must not require editing existing logic. UI pages render from module metadata.
- Examples are generic. Each example (a PRG, a PRF, and so on) is a declarative definition: its LaTeX definition, its input and output lengths, its evaluation function, the component primitives it is built from, and whether it is secure. Pages and experiments take such a definition as input and never branch on a specific example. Adding a new example means adding data only, and every option (refute, prove, reveal the answer) works for it immediately.
- Experiments follow the same rules for every example: fresh randomness in every round, and bit-string lengths validated at every boundary (a fixed-length bit-string type is recommended) so that a string of the wrong length can never be passed to an oracle.
- The reduction framework is generic and shared by the PRG and the PRF (a PRG is the special case whose challenge oracle has the single input ε).
  - Interactive mode (PRG, v1): the site simulates the assumed distinguisher `D'` by exhaustive search over all `2^n` seeds, and the user plays the distinguishing experiment against the underlying primitive with `D'` as an extra oracle.
  - Strategy analysis (later): the engine drives predefined `D'` strategies. The reduction `Sim` is a declarative expression AST. In the first version the user picks `Sim` from candidates in the example's data. A builder UI that edits the same AST comes later. Do not build it before its roadmap task.
- Respect the v1 versus later split in `docs/ARCHITECTURE.md`, section 9: v1 code never imports later code.
- Exhaustive simulation requires small security parameters. Enforce the maximum `n` from the limits file, cache results per example and `n`, and run heavy computation off the UI thread (for example in a Web Worker).
- Depend on abstractions (interfaces), not concrete implementations. Inject dependencies (for example the randomness source) so that experiments are testable and deterministic under test.
- Prefer small, single-purpose modules and functions.

## Configuration

Nothing that may change during development is hard-coded in components. Keep it in dedicated, easy-to-edit files:

- UI strings, labels and explanations: `src/config/strings/<feature>.ts`.
- Theme tokens (colors, fonts, spacing): `src/config/theme.ts`.
- Limits and defaults: `src/config/limits.ts`. This covers the maximum security parameter (global 20, with lower caps per kind and per example), the maximum number of rounds (10) and the default security parameter (10).

The folder layout is defined in `docs/ARCHITECTURE.md`, section 2. Keep it consistent, and update that document in the same PR if it has to change.

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
