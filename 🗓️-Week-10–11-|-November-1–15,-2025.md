
## 🧭 Summary

During this period, the Cavefish initiative shifted decisively from architectural exploration to **fully specified and design-complete execution flows** for the Tx‑Level Strategy prototype.  

Both SRL 1 → 2 and SRL 2 → 3 workstreams advanced in parallel, with all major components of the Single Service Provider prototype now clearly structured, documented, and ready for full implementation.

We also refined the intent‑based transaction model, clarified the minimal anti‑malleability requirements for the light intent strategy, and ensured consistency across specifications, documentation, and design artefacts.

The milestone ends with a cohesive specification set, converged design language, and a validated development trajectory for the prototype.

## 🎯 Goals

### [SRL 1 → 2 | CPS(s), Specification & Scoped Requirements](https://github.com/input-output-hk/innovation-cavefish/milestone/3)

#### Intent Semantics & Ledger Alignment

- Continued refinement of intent semantics for Cardano.
- Integrated insights from the exploratory intent CIP:  
  - https://github.com/cardano-foundation/CIPs/pull/1110  
  - https://github.com/polinavino/CIPs/blob/polina/light/CIP-%3F%3F%3F%3F/README.md
- These insights strengthen the upcoming **Next Steps & Recommendations** deliverable.

#### Security & CBOR Structure Analysis

- Clarified the minimal checks needed to prevent CBOR-level malleability in the Tx‑Level Strategy.
- Key conclusions for the *light intent model*:  
  - Full CBOR decoding is *not required* inside the circuit.  
  - Off‑circuit verification is sufficient for public fields.  
  - In‑circuit checks remain limited to verifying equality outside zeroed fields.  
- Ongoing work continues for long-term intent semantics and extended intent models.

### [SRL 2 → 3 | Single Service Provider Prototype](https://github.com/input-output-hk/innovation-cavefish/milestone/1)

#### Full Flow Specification & Design  
**[Epic #57 — Tx-Level Strategy & Single Service Provider Prototype](https://github.com/input-output-hk/innovation-cavefish/issues/57)**

The entire 7‑step WBPS ↔ Cavefish execution flow is now **fully specified and design‑complete**.  
All components were decomposed into implementable units, validated for correctness, and prepared for backend integration.

## 🛠 Implementation Progress

### Backend & Prototype Logic

- Registration flow now integrates real proving/verification contexts.
- Commitment-phase in progress:
  - SP receives user‑generated `R`.
  - Initial Haskell translation for ephemeral nonce logic (`r`, `R`).
  - Structures prepared for commitment → challenge → proof.
- Deterministic testing harness created to validate the cryptographic flow.

### UI Exploration (Paused)

- A UI scaffold was prototyped to validate browser‑side verification feasibility.
- UI development remains paused until backend flow stabilizes.

### CI, Tooling & Infrastructure

- Improved Nix tooling (WBPS dev shell).
- CI now installs Circom + snarkJS.
- Git LFS configuration updated.
- Test inputs restored under proper LFS tracking.
- Early submission infrastructure updated for the prototype.

## 📄 Deliverables

### Documents

- Updated Tx‑Level Strategy specification and design (Epic #57).  
- Updated WBPS flow diagrams:  
  https://miro.com/app/board/uXjVI1nt2yc=/  
  Password: `caevfish12345`

### Commits (November 1–15, 2025)

* [45e243a](https://github.com/input-output-hk/innovation-cavefish/commit/45e243a5f75a097646792d1abfaeadd9e5f5eab8) — Merge pull request #76 from input-output-hk/ci/cabal-fmt *(Nicolas Henin)*
* [85dce60](https://github.com/input-output-hk/innovation-cavefish/commit/85dce606e6985fcec21106ff66754577117bb0ff) — Fix cabal files formatting *(kayvank)*
* [eb91990](https://github.com/input-output-hk/innovation-cavefish/commit/eb91990d27b8e6a158caa12aa51d62981a6ca650) — nixos git-lfs snarkJS support (#51) *(kayvank)*
* [e4f44c1](https://github.com/input-output-hk/innovation-cavefish/commit/e4f44c1a4b2ad554e5ea4f5c79230f388e95c84d) — Merge pull request #56 from input-output-hk/49/fix-build *(Nicolas Henin)*
* [a0e78a0](https://github.com/input-output-hk/innovation-cavefish/commit/a0e78a0620111e7c655c4b59ee0cb81a093bc5b8) — Fix build warnings *(kayvank)*
* [0a1399b](https://github.com/input-output-hk/innovation-cavefish/commit/0a1399bb67af2ac5a02f083469cea9e3a5f7f881) — Merge pull request #50 from input-output-hk/wg/demo-ui *(Nicolas Henin)*
* [cb48ff1](https://github.com/input-output-hk/innovation-cavefish/commit/cb48ff1d01ccc62ff371cdf4bc44dc4d444c6bd6) — Merge pull request #45 from input-output-hk/nhenin/wbps-register-accounts *(Nicolas Henin)*
* [97d0527](https://github.com/input-output-hk/innovation-cavefish/commit/97d05278b8fe6533a07ced971dc687e9f3ea17f1) — Apply feedback from Kayvan’s PR review *(Nicolas Henin)*
* [e96bc6c](https://github.com/input-output-hk/innovation-cavefish/commit/e96bc6c608b055f38a5e707145b867bdf9eff578) — format *(Will Gould)*
* [3ab7574](https://github.com/input-output-hk/innovation-cavefish/commit/3ab7574460b37cc3b6d3fd073c01e60b66bd4062) — Merge remote-tracking branch 'origin/master' into wg/demo-ui *(Will Gould)*
* [b1ed74e](https://github.com/input-output-hk/innovation-cavefish/commit/b1ed74ef624209deaadeaa65d3730de6710acce7) — 100 day demo initial dump *(Will Gould)*
* [3e0d2f3](https://github.com/input-output-hk/innovation-cavefish/commit/3e0d2f3796501186791867294fccbf78dd13c8a8) — Update Logbook.md *(Nicolas Henin)*
* [8ec36df](https://github.com/input-output-hk/innovation-cavefish/commit/8ec36df747b7d8d34f18020c1904270d8e6a5fa2) — Re-add inputs as LFS objects *(Nicolas Henin)*
* [82d35fd](https://github.com/input-output-hk/innovation-cavefish/commit/82d35fd48a892c5b7a02dcf347a95717b89d2c3f) — instealled git lfs in CI to pull the input files for tests *(Nicolas Henin)*
* [0f10be2](https://github.com/input-output-hk/innovation-cavefish/commit/0f10be25489b3d934b6b67fe6ca442682caf537e) — [ci] extend CI to install circom/snarkjs *(Nicolas Henin)*
* [e4d05f7](https://github.com/input-output-hk/innovation-cavefish/commit/e4d05f7c1d9d381b667b375e0b6bb23764e7b71f) — [nix] Add WBPS dev shell to cavefish flake *(Nicolas Henin)*
* [01625dc](https://github.com/input-output-hk/innovation-cavefish/commit/01625dc6cacb5f3e19b61add3efd569d6e17a05b) — [nix] gate cardano tooling on non-aarch64 hosts *(Nicolas Henin)*
* [e853483](https://github.com/input-output-hk/innovation-cavefish/commit/e853483238d687e7dad4ae3c92844c6085994076) — Apply Fourmolu formatting *(Nicolas Henin)*
* [c853440](https://github.com/input-output-hk/innovation-cavefish/commit/c853440e909fb99348acfa509ec4eec57a56dc9d) — Fixed compiler warnings and relaxed strict build rules for selected packages.fixed some warnings and relaxed rules for some packages *(Nicolas Henin)*
* [8d7c3cb](https://github.com/input-output-hk/innovation-cavefish/commit/8d7c3cb16dd57726104b74ae0319528a003f8dad) — Track heavy artifacts with Git LFS *(Nicolas Henin)*
* [56f15f2](https://github.com/input-output-hk/innovation-cavefish/commit/56f15f2e4d102c55e4e2573f0ee6d6ccbfb7ded5) — [WBPS]- Setup Sub-package + Register Accounts *(Nicolas Henin)*
* [ccd0273](https://github.com/input-output-hk/innovation-cavefish/commit/ccd0273b70fc8d09a4ceec98d5eb24cf0e8e6f31) — Merge pull request #42 from input-output-hk/cabal/make-imports-implicit *(Nicolas Henin)*

### Issues (November 1–15, 2025)

#### Closed
* [#6](https://github.com/input-output-hk/cavefish/issues/6) — Introduce Haskell bindings for WBPS witness and proof flow
* [#29](https://github.com/input-output-hk/cavefish/issues/29) — UI Client
* [#36](https://github.com/input-output-hk/cavefish/issues/36) — Set-up WBPS Haskell lib
* [#44](https://github.com/input-output-hk/cavefish/issues/44) — [wbps]- Register an account
* [#46](https://github.com/input-output-hk/cavefish/issues/46) — 🧩 Fix compiler warnings and re-enable GHC -Werror in all sub-packages
* [#49](https://github.com/input-output-hk/cavefish/issues/49) — Add Git LFS support in the Nix development shell

#### Created
* [#44](https://github.com/input-output-hk/cavefish/issues/44) — [wbps]- Register an account
* [#46](https://github.com/input-output-hk/cavefish/issues/46) — 🧩 Fix compiler warnings and re-enable GHC -Werror in all sub-packages
* [#47](https://github.com/input-output-hk/cavefish/issues/47) — Restore Cardano tooling in the dev shell on aarch64
* [#48](https://github.com/input-output-hk/cavefish/issues/48) —  CI: Add Nix dev-shell job (`nix develop`) on x86_64 and aarch64
* [#49](https://github.com/input-output-hk/cavefish/issues/49) — Add Git LFS support in the Nix development shell
* [#52](https://github.com/input-output-hk/cavefish/issues/52) — 🎣 1. Register a User
* [#53](https://github.com/input-output-hk/cavefish/issues/53) — [Cavefish Server] Register an Account
* [#54](https://github.com/input-output-hk/cavefish/issues/54) — [wbps] Get Verification Key
* [#55](https://github.com/input-output-hk/cavefish/issues/55) — [Web Page] Register
* [#57](https://github.com/input-output-hk/cavefish/issues/57) — 🏝️  Tx-Level Strategy & Single Service Provider Prototype 
* [#58](https://github.com/input-output-hk/cavefish/issues/58) — 🛠️ 0. DevOps / Infrastructure Support (reopened)
* [#59](https://github.com/input-output-hk/cavefish/issues/59) — 🎣 2. User Generates Intent → SP Produces Transaction
* [#60](https://github.com/input-output-hk/cavefish/issues/60) — 🎣 3. SP Produces Commitment → Sent to User
* [#61](https://github.com/input-output-hk/cavefish/issues/61) — 🎣 4-5. User Requests the Proof that the SP’s Commitment Is Valid
* [#62](https://github.com/input-output-hk/cavefish/issues/62) — 🎣 5. SP Produces Challenge and Proof (duplicate)
* [#63](https://github.com/input-output-hk/cavefish/issues/63) — 🎣 6-7. User Verification and Authorization of the Prepared Transaction (reopened)
* [#64](https://github.com/input-output-hk/cavefish/issues/64) — 🎣 7. User Signs Commitment → SP Submits Signed Tx (duplicate)
* [#65](https://github.com/input-output-hk/cavefish/issues/65) — Tx-level Strategy / Security Analysis
* [#66](https://github.com/input-output-hk/cavefish/issues/66) — Secure "TxAbs" Strategy Implementation
* [#67](https://github.com/input-output-hk/cavefish/issues/67) — 2. [cavefish-server] Intent DSL interpreter (Cardano Transaction Builder)
* [#68](https://github.com/input-output-hk/cavefish/issues/68) — 1. [cavefish-core] Intent DSL
* [#69](https://github.com/input-output-hk/cavefish/issues/69) — 3. [cavefish-server] Endpoint: POST /intent/solve
* [#70](https://github.com/input-output-hk/cavefish/issues/70) — 4. [web-page] UI for creating intents + call to server
* [#71](https://github.com/input-output-hk/cavefish/issues/71) — a (not planned)
* [#72](https://github.com/input-output-hk/cavefish/issues/72) — b (not planned)
* [#73](https://github.com/input-output-hk/cavefish/issues/73) — c (not planned)
* [#74](https://github.com/input-output-hk/cavefish/issues/74) — Structured logging

#### Updated
* [#6](https://github.com/input-output-hk/cavefish/issues/6) — Introduce Haskell bindings for WBPS witness and proof flow
* [#29](https://github.com/input-output-hk/cavefish/issues/29) — UI Client
* [#36](https://github.com/input-output-hk/cavefish/issues/36) — Set-up WBPS Haskell lib
* [#40](https://github.com/input-output-hk/cavefish/issues/40) — Add cardano-node binary to the nix shell
* [#44](https://github.com/input-output-hk/cavefish/issues/44) — [wbps]- Register an account
* [#46](https://github.com/input-output-hk/cavefish/issues/46) — 🧩 Fix compiler warnings and re-enable GHC -Werror in all sub-packages
* [#47](https://github.com/input-output-hk/cavefish/issues/47) — Restore Cardano tooling in the dev shell on aarch64
* [#49](https://github.com/input-output-hk/cavefish/issues/49) — Add Git LFS support in the Nix development shell
* [#58](https://github.com/input-output-hk/cavefish/issues/58) — 🛠️ 0. DevOps / Infrastructure Support (reopened)
* [#66](https://github.com/input-output-hk/cavefish/issues/66) — Secure "TxAbs" Strategy Implementation

#### In Progress
* [#47](https://github.com/input-output-hk/cavefish/issues/47) — Restore Cardano tooling in the dev shell on aarch64
* [#58](https://github.com/input-output-hk/cavefish/issues/58) — 🛠️ 0. DevOps / Infrastructure Support (reopened)
* [#66](https://github.com/input-output-hk/cavefish/issues/66) — Secure "TxAbs" Strategy Implementation
