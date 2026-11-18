# CryptoCalc — Project Brief

## 1. Vision

CryptoCalc is a website that lets students of cryptography get to know cryptographic primitives and encryption schemes in depth. Students simulate security experiments for encryption schemes and run tests against cryptographic primitives.

The project doubles as a portfolio piece, so it is built the way an industry project would be: clean architecture, tests, pull requests and CI.

## 2. Constraints and non-functional requirements

- Built with Node.js tooling and React. Client side only, with no server.
- Beautiful, innovative design that fits a cyber and cryptography atmosphere.
- Formal definitions are presented in LaTeX.
- All strings, names and values that may change during development (colors, labels, limits, and so on) live in dedicated, easy-to-edit files, organized by context.
- Easy to extend: built-in primitives, schemes and examples can be added, edited or removed without touching existing logic. The maintainer writes the built-in content.
- Architecture follows SOLID and is open for extension and closed for modification.
- Deployed as a public site, easy to run locally, so that students can actually use it once there is enough content.
- Experiments run on small security parameters. This keeps the behavior easy to follow, and it lets the site simulate oracles by exhaustive search over all `2^n` seeds. The maximum `n` is enforced in the limits file so that the computation stays fast, and heavy computation runs off the UI thread.

## 3. Content model

Each primitive page offers two entry points:

1. **Existing examples:** a list of built-in examples, each with its own definition.
2. **Test a new construction:** the user enters their own definition through a parser. This option is deferred (see section 7). Until then it is visible but does nothing.

### Generic examples

The handling of every example, including the ones the parser will produce later, is generic. All examples offer the same options. Only the identity of the generator (or function) itself differs. Each example is therefore a declarative definition that carries everything the page and the experiments need:

- its definition, as a LaTeX string;
- its input length `n` and its output length (for a generator, `n' > n`, as a function of `n`);
- its evaluation function;
- the components it is built from (the generators or functions that appear in its definition), each with its own lengths and evaluation function;
- for proofs: the building block that is known to be secure;
- the answer: whether the example is secure.

Defining a new example means adding such a definition. Every option (refute, prove, reveal the answer) then works for it immediately, with no other code changes.

## 4. Navigation

The home page shows the site name, a short explanation, and a menu:

- Section **Cryptographic primitives:** PRG, PRF, cryptographic hash function.
- Section **Security experiments:** eavesdropper security, CPA security, CCA security.

## 5. PRG module

### 5.1 PRG page

The page opens with the formal definition of a pseudorandom generator, written in LaTeX. Below it are the list of examples and the (inactive) option to add a new PRG with the parser.

Before the list, an explanation tells the user that for each example they should try to work out whether the generator really is a PRG:

- To **refute**, show a distinguisher that tells the generator's output from a uniformly random string with non-negligible advantage. The user plays this distinguisher: they receive strings and must decide whether each is the generator's output or random. If they succeed consistently, the generator is probably not a PRG.
- To **prove**, argue by contradiction through a reduction. Assume a distinguisher exists for the generator. Show that this yields a distinguisher for another generator that appears in the example's definition and is known to be a PRG. The user plays that second distinguisher while having oracle access to the assumed one.

### 5.2 Example page

Opening an example shows its definition in LaTeX and these options:

1. Try to refute
2. Try to prove
3. Reveal the correct answer
4. Return to the PRG menu

### 5.3 Refutation: distinguishing experiment

1. The user chooses the security parameter `n` (default 10, hard maximum set in the limits file, see section 9) and the number of rounds `t` (hard maximum 10).
2. In each round, a hidden coin is flipped. With probability 1/2 the user receives the generator's output: a fresh uniformly random seed of length `n` is drawn, given to the generator, and the output is shown. Otherwise the user receives a fresh uniformly random string of length `n'`. Every round draws new strings, and every string must have the correct length.
3. **Oracle section.** By Kerckhoffs's principle the algorithms are not secret. The user therefore gets oracle access to every generator that appears in the example's definition. The number of oracles and their contents vary between examples, so this is generic: for each one, the user enters an input of the correct length and receives that generator's output.
4. The user guesses whether the challenge is the generator's output or random, and then sees the correct answer.
5. After `t` rounds, a summary shows successes versus failures, so the user can judge whether they distinguish with non-negligible advantage.

### 5.4 Proof: reduction

Some examples define a generator `G'` in terms of a generator `G` that is known to be a PRG. Proving that `G'` is a PRG always works through a reduction. In the interactive mode the user plays the reduction directly, so nothing has to be built. Choosing "Try to prove" starts the following flow, accompanied by a plain-language explanation of the argument:

1. **Assumption for contradiction.** Assume `G'` is not a PRG. Then a distinguisher `D'` for `G'` exists. The site simulates it.
2. **Simulated `D'`.** Because `n` is small, the site simulates `D'` by exhaustive search: it runs `G'` on all `2^n` seeds and answers whether a given string is a possible output of `G'`. `D'` is added to the oracle section described in 5.3. This `D'` is an unbounded distinguisher on purpose, since the proof only assumes that some distinguisher exists.
3. **The user plays against `G`.** The user runs the distinguishing experiment from 5.3 against the underlying generator `G`, now with `D'` available as an extra oracle. The task is to use `D'` to win the experiment.
4. **Contradiction.** If the user wins with non-negligible advantage, then `D'` lets them distinguish `G`, which contradicts `G` being a PRG. Hence the assumption was false and `G'` is a PRG.

The same simulation machinery serves every example, including future ones produced by the parser.

**Strategy analysis (later, after the PRF reduction in task 10).** The PRG is the oracle-free special case of the PRF reduction framework (section 6.2): the challenge is an oracle on the single input ε. The PRG proof page therefore gains a second mode next to the interactive one. In that mode the user picks a reduction `Sim` from candidates stored in the example's definition, and the site runs a library of predefined `D'` strategies, with exhaustive search as one of them, and shows the exact advantages side by side. See `docs/ARCHITECTURE.md`, sections 3.11 and 4.4.

### 5.5 Reveal the answer

At any point the user can reveal whether the example is a PRG, using the answer stored in the example's definition.

## 6. PRF module

The PRF page mirrors the PRG page: definition, examples, and the (inactive) option to add a new PRF. It reuses the generic example mechanism.

### 6.1 Refutation

1. The user chooses the security parameter `n` and the number of attempts `m`.
2. In each attempt, a hidden coin is flipped. With probability 1/2 the user receives oracle access to `F_k`, where the key `k` is a fresh uniformly random `n`-bit string. Otherwise the user receives oracle access to a truly random function.
3. The user may query the oracle on different inputs, then guesses which case it is.
4. Every attempt flips a new coin and samples a new key.

### 6.2 Proof

The proof is a reduction to the PRF `F` given in the example's definition, which is assumed to be a PRF. In the formal reduction, `D'` chooses its own queries and the reduction only translates them, so the user does not answer `D'`'s queries by hand:

1. **`D'` is a black box.** The site ships a library of predefined `D'` strategies: random guess (baseline), fixed non-adaptive queries, collision test, structure tests (equal halves, swapped halves on `x` and `x̄`), exhaustive key consistency, and an adaptive strategy. The engine drives each strategy as a coroutine.
2. **The user supplies the reduction `Sim`** as a declarative mapping, for example "for query `x`, ask `O` at `0‖x` and `1‖x` and return the concatenation". It is stored as a small expression AST. In the first version the user picks `Sim` from candidates listed in the example's definition (correct ones and instructive flawed ones). A builder UI that edits the same AST comes later. The engine applies `Sim` to every `D'` query, so the simulated oracle is `O' = Sim(O)`.
3. **Oracles are memoized.** The PRF oracle is `F_k` with a fixed `k`. The random oracle uses lazy sampling with a table, so it behaves as a true function.
4. **Exact advantages.** For small `n` the site computes `Pr[D'^{F'_k}=1]` over all `k`, `Pr[D'^g=1]` over random `g` (by enumeration, or by sampling when the space is too big), and the same two quantities for `D = D'∘Sim` over `F_k` and a random `f`. They are shown side by side.
5. **Optional step-by-step trace** of `D'` queries, `O` queries and answers, for small `q`.

**Key teaching moment.** For the broken `F'_k(x) = F_k(x) ‖ F_k(x̄)`, the natural reduction ("query `O(x)` and `O(x̄)`, concatenate") fails. With `O = F_k`, `D'` sees the swap symmetry and says "PRF". With a random `O = f`, the simulated answers `f(x)‖f(x̄)` are symmetric too, so `D'` again says "PRF", and `D` has zero advantage. The simulated oracle in the random case is not a random function, and the UI shows this as `Pr[D^f=1] ≠ Pr[D'^g=1]`. For the secure `F'_k(x) = F_k(0‖x) ‖ F_k(1‖x)` the two numbers are equal.

**Caveat shown in the UI.** With a small key space, `F` itself is not a PRF, because brute-force key search distinguishes it. The tool demonstrates that the reduction preserves the distributions (`Adv_D = Adv_{D'}`), not that `F` is secure. `F` is treated as a black box.

**Later extension.** Constructions such as `F_k(F_k(x))` need a hybrid argument (an oracle that behaves like `F` up to query `i` and is random afterwards). The architecture leaves an extension point for this.

### 6.3 Reveal the answer

Same as for the PRG.

## 7. Later stages

- The remaining pages: cryptographic hash functions and the EAV, CPA and CCA security experiments.
- A parser that lets users enter their own primitive or scheme definition and run all available tools on it, except for deciding whether it is secure. The parser is simple and supports only a closed list of basic operations. A user definition always builds on some other primitive or scheme, which is what makes reductions possible. Parser output uses the same generic example definition as the built-in examples.

## 8. Roadmap

Tasks are done one by one, in this order:

- [ ] 1. Design a proposed architecture (SOLID, open for extension), including the generic example definition.
- [ ] 2. Home page with title, explanation and the menu from section 4.
- [ ] 3. PRG page: LaTeX definition, explanation, and the two entry points.
- [ ] 4. PRG example list and the example page (refute, prove, reveal, navigation).
- [ ] 5. PRG distinguishing experiment, oracle section, and the reduction experiment.
- [ ] 6. Make sure the "add a new PRG" option exists but does nothing.
- [ ] 7. PRF page: definition, examples, inactive "add a new PRF" option.
- [ ] 8. PRF example page: reveal, refute, prove.
- [ ] 9. PRF distinguishing experiment.
- [ ] 10. PRF reduction experiment.
- [ ] 11. Strategy-analysis mode for the PRG proof (shared reduction framework).
- [ ] 12. Reduction builder UI that edits the `Sim` expression AST.
- [ ] 13. Hybrid arguments (for constructions such as `F_k(F_k(x))`).
- [ ] 14. Next tasks, to be defined.

## 9. Open questions

Resolved in the architecture task (see `docs/ARCHITECTURE.md`):

- ~~The hard maximum for `n`.~~ A global cap in the limits file (proposed 20), a per-kind cap (PRF: 16), and an optional lower cap per example. The effective maximum is the smallest of the three.
- ~~How the PRF oracle `D'` is simulated.~~ A library of predefined `D'` strategies driven by the engine, plus a declarative reduction `Sim` (section 6.2).
- ~~Whether the example page links directly to the home page.~~ A global header and breadcrumbs that always include Home.
- ~~Deployment target.~~ GitHub Pages, deployed by GitHub Actions, using hash-based routing.

Still open:

- Maximum number of attempts `m` and of queries per attempt in the PRF experiment (proposed: 10 and 16).
- The initial list of built-in PRG and PRF examples, and the toy base generator `G` and function `F`.
- The principle written as "SVM" in the original prompt needs to be identified, since SOLID is the confirmed principle.
