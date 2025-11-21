
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

##### Sub‑Issues (Specification Complete)

| Sub-Issue | Description |
|----------|-------------|
| 🎣 **[1. Register a User #52](https://github.com/input-output-hk/innovation-cavefish/issues/52)** | Covers signer setup, verification context retrieval, and account registration with the SP. |
| 🛠️ **[0. DevOps / Infrastructure Support #58](https://github.com/input-output-hk/innovation-cavefish/issues/58)** | Provides CI, Nix setup, cardano-node integration, Git LFS, and essential tooling. |
| 🎣 **[2. User Generates Intent → SP Produces Transaction #59](https://github.com/input-output-hk/innovation-cavefish/issues/59)** | Defines the `prepare` endpoint: intent ingestion, transaction construction, masking rules, auxiliary data, and ClientWitnessBundle. |
| 🎣 **[3. SP Produces Commitment → Sent to User #60](https://github.com/input-output-hk/innovation-cavefish/issues/60)** | Specifies SP handling of curve point `R`, commitment construction, and challenge preparation. |
| 🎣 **[4–5. User Requests the Proof that the SP’s Commitment Is Valid #61](https://github.com/input-output-hk/innovation-cavefish/issues/61)** | Defines how the user requests `(c, π)` and how the SP generates and returns the proof. |
| 🎣 **[6–7. User Verification and Authorization of the Prepared Transaction #63](https://github.com/input-output-hk/innovation-cavefish/issues/63)** | Covers proof verification and computation of scalar `s` to complete user-side authorization. |

Together, these form a consistent, end‑to‑end execution model ready for direct implementation.

![single SP flow - Tx-Level Strategy](https://github.com/user-attachments/assets/dea08c3a-2141-40d1-8d95-f8e64eb12d1f)

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
* [eb91990](https://github.com/input-output-hk/innovation-cavefish/commit/eb91990d27b8e6a158caa12aa51d62981a6ca650) — nixos git-lfs snarkJS support (#51) *(Kayvan  ≅ کیوان)*  
* [e4f44c1](https://github.com/input-output-hk/innovation-cavefish/commit/e4f44c1a4b2ad554e5ea4f5c79230f388e95c84d) — Merge pull request #56 from input-output-hk/49/fix-build *(Nicolas Henin)*  
* [a0e78a0](https://github.com/input-output-hk/innovation-cavefish/commit/a0e78a0620111e7c655c4b59ee0cb81a093bc5b8) — Fix build warnings *(kayvank)*  
* [0a1399b](https://github.com/input-output-hk/innovation-cavefish/commit/0a1399bb67af2ac5a02f083469cea9e3a5f7f881) — Merge pull request #50 from input-output-hk/wg/demo-ui *(Nicolas Henin)*  
* [cb48ff1](https://github.com/input-output-hk/innovation-cavefish/commit/cb48ff1d01ccc62ff371cdf4bc44dc4d444c6bd6) — Merge pull request #45 from input-output-hk/nhenin/wbps-register-accounts *(Nicolas Henin)*  
* [97d0527](https://github.com/input-output-hk/innovation-cavefish/commit/97d05278b8fe6533a07ced971dc687e9f3ea17f1) — Apply feedback from Kayvan’s PR review *(Nicolas Henin)*  
* [e96bc6c](https://github.com/input-output-hk/innovation-cavefish/commit/e96bc6c608b055f38a5e707145b867bdf9eff578) — format *(Will Gould)*  
* [3ab7574](https://github.com/input-output-hk/innovation-cavefish/commit/3ab7574460b37cc3b6d3fd073c01e60b66bd4062) — Merge remote-tracking branch 'origin/master' into wg/demo-ui *(Will Gould)*  
* [b1ed74e](https://github.com/input-output-hk/innovation-cavefish/commit/b1ed74ef624209deaadeaa65d3730de6710acce7) — 100 day demo initial dump *(Will Gould)*  
* [3e0d2f3](https://github.com/input-output-hk/innovation-cavefish/commit/3e0d2f3796501186791867294fccbf78dd13c8a8) — Update Logbook.md *(Nicolas Henin)*  
* [8ec36df](https://github.com/input-output-hk/innovation-cavefish/commit/8ec36df747b7d8d34f18020c190427) — Re-add inputs as LFS objects *(Nicolas Henin)*
