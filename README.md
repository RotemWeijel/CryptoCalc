# CryptoCalc

An interactive, client-side cryptography calculator for students. CryptoCalc lets you explore cryptographic primitives and encryption schemes in depth by running security experiments in the browser: play the distinguisher, try to break a construction, or try to prove it secure through a reduction.

> **Status:** work in progress. Not yet deployed.

> **Disclaimer:** CryptoCalc is an educational tool. The implementations are for demonstration only and must not be used to protect real data.

## Planned features

- **Cryptographic primitives:** pseudorandom generators (PRG), pseudorandom functions (PRF), cryptographic hash functions.
- **Security experiments:** eavesdropper (EAV) security, CPA security, CCA security.
- For each primitive: the formal definition, a list of built-in examples, and two ways to study an example:
  - **Try to refute:** play the distinguisher in a simulated experiment and see your success statistics.
  - **Try to prove:** build a reduction to a primitive that is assumed to be secure.
- Reveal the answer for any example (secure or not).
- Formal definitions rendered in LaTeX.
- Every example is a generic, declarative definition, so adding one means adding data, not code.
- Easy to extend: examples, strings and theme values are plain data files.
- Later: a small parser for entering your own primitive or scheme definitions.

## Tech stack

React, TypeScript, Vite, Vitest and KaTeX. Client-side only, with no backend.

## Getting started

Setup instructions will be added once the project scaffold is merged.

## Documentation

- [Project brief and roadmap](docs/PROJECT_BRIEF.md)

## Contributing

Work happens on feature branches and lands through pull requests. Commit messages follow [Conventional Commits](https://www.conventionalcommits.org/).

## License

Released under the [MIT License](LICENSE).
